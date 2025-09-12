<template>
  <v-container fluid class="login-page">
    <v-row justify="center" align="center" class="fill-height">
      <v-col cols="12" sm="8" md="5" lg="4" xl="3">
        
        <!-- Componente Login -->
        <LoginComponent
          :logo-url="companyLogo"
          :logo-placeholder="companyInitials"
          :app-title="pageTitle"
          :app-subtitle="pageSubtitle"
          @login-success="onLoginSuccess"
          @forgot-password="onForgotPassword"
          @register-request="onRegisterRequest"
          @show-message="showNotification"
          @logo-error="onLogoError"
        />
        
      </v-col>
    </v-row>

    <!-- Snackbar para notificaciones -->
    <v-snackbar 
      v-model="snackbar.show" 
      :color="snackbar.color"
      :timeout="snackbar.timeout"
      top
    >
      <v-icon left>{{ snackbar.icon }}</v-icon>
      {{ snackbar.message }}
      <template v-slot:action="{ attrs }">
        <v-btn
          text
          v-bind="attrs"
          @click="snackbar.show = false"
        >
          Cerrar
        </v-btn>
      </template>
    </v-snackbar>

    <!-- Loading overlay global -->
    <v-overlay :value="globalLoading">
      <v-progress-circular
        indeterminate
        size="64"
        color="primary"
      ></v-progress-circular>
    </v-overlay>
  </v-container>
</template>

<script>
// Importar el componente
import LoginComponent from '@/components/forms/LoginForm.vue'
import logoImage from '@/assets/logo-dark-transparent.png'

export default {
  name: 'LoginView',
  
  components: {
    LoginComponent
  },
  
  data() {
    return {
      // 🎨 CONFIGURACIÓN DE LA EMPRESA/APLICACIÓN
      companyLogo: logoImage, // 👈 AQUÍ PON TU LOGO: './assets/company-logo.png'
      companyInitials: 'AJ', // O iniciales como 'MC' para Mi Compañía
      pageTitle: 'ArJess',
      pageSubtitle: 'Punto de venta',
      
      // Estados globales
      globalLoading: false,
      
      // Snackbar para notificaciones
      snackbar: {
        show: false,
        message: '',
        color: 'info',
        timeout: 4000,
        icon: 'mdi-information'
      }
    }
  },
  
  created() {
    console.log('🚀 LoginView iniciada');
    
    // Verificar si ya está autenticado
    this.checkExistingAuth();
    
    // Configurar empresa/aplicación
    this.setupCompanyBranding();
  },
  
  methods: {
    // 🎨 Configurar branding de la empresa
    setupCompanyBranding() {
      // 💡 PERSONALIZA AQUÍ TU EMPRESA
      // this.companyLogo = './assets/mi-empresa-logo.png';
      // this.companyInitials = 'ME'; // Mis Empresas
      // this.pageTitle = 'Mi Empresa';
      // this.pageSubtitle = 'Sistema de Ventas';
      
      // También puedes cargar desde API o localStorage
      const savedBranding = localStorage.getItem('companyBranding');
      if (savedBranding) {
        const branding = JSON.parse(savedBranding);
        this.companyLogo = branding.logo || this.companyLogo;
        this.pageTitle = branding.title || this.pageTitle;
        this.pageSubtitle = branding.subtitle || this.pageSubtitle;
      }
    },
    
    // 🔐 Verificar autenticación existente
    checkExistingAuth() {
      const token = localStorage.getItem('authToken');
      if (token) {
        // Si ya tiene token válido, redirigir al dashboard
        this.showNotification({
          message: 'Ya tienes una sesión activa',
          type: 'info'
        });
        
        // Verificar si el token es válido antes de redirigir
        this.validateExistingToken(token);
      }
    },
    
    // 🔍 Validar token existente
    async validateExistingToken(token) {
      try {
        // 📡 AQUÍ VALIDARÍAS EL TOKEN CON TU BACKEND
        /*
        const response = await fetch('/api/auth/validate', {
          headers: {
            'Authorization': `Bearer ${token}`
          }
        });
        
        if (response.ok) {
          // Token válido, redirigir al dashboard
          setTimeout(() => {
            this.$router.push('/dashboard');
          }, 1500);
        } else {
          // Token inválido, limpiar localStorage
          localStorage.removeItem('authToken');
        }
        */
        
        // 🎭 SIMULACIÓN (eliminar al conectar backend)
        console.log('🔍 Validando token existente...');
      } catch (error) {
        console.error('Error validando token:', error);
        localStorage.removeItem('authToken');
      }
    },
    
    // ✅ Manejar login exitoso
    async onLoginSuccess(userData) {
      console.log('✅ Login exitoso:', userData);
      
      this.globalLoading = true;
      
      try {
        // Guardar datos del usuario
        localStorage.setItem('authToken', userData.token);
        localStorage.setItem('userData', JSON.stringify(userData.user));
        
        // Mostrar mensaje de éxito
        this.showNotification({
          message: `¡Bienvenido, ${userData.user.name || userData.user.email}!`,
          type: 'success'
        });
        
        // 📡 AQUÍ PUEDES HACER LLAMADAS ADICIONALES
        // await this.loadUserPermissions();
        // await this.loadCompanySettings();
        
        // Redirigir al dashboard después de un delay
        setTimeout(() => {
          this.$router.push({
            name: 'Dashboard',
            // Puedes pasar datos como query params si necesitas
            query: { welcome: 'true' }
          });
        }, 2000);
        
      } catch (error) {
        console.error('Error post-login:', error);
        this.showNotification({
          message: 'Error al procesar el login',
          type: 'error'
        });
      } finally {
        this.globalLoading = false;
      }
    },
    
    // 🔑 Manejar recuperación de contraseña
    onForgotPassword(email) {
      console.log('🔑 Solicitud de recuperación para:', email);
      
      // 📡 AQUÍ HARÍAS LA LLAMADA A TU ENDPOINT
      /*
      fetch('/api/auth/forgot-password', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email })
      });
      */
      
      this.showNotification({
        message: `Enlace de recuperación enviado a ${email}`,
        type: 'info'
      });
    },
    
    // 📝 Manejar solicitud de registro
    onRegisterRequest() {
      console.log('📝 Redirigiendo a registro');
      
      // Redirigir a la vista de registro
      this.$router.push({ name: 'Register' });
      
      // O mostrar modal de registro si prefieres
      // this.showRegisterModal = true;
    },
    
    // 🖼️ Manejar error de logo
    onLogoError() {
      console.warn('⚠️ Error cargando logo de empresa');
      // Podrías intentar cargar un logo de respaldo
      // this.companyLogo = './assets/default-logo.png';
    },
    
    // 📢 Mostrar notificaciones
    showNotification({ message, type = 'info' }) {
      const config = {
        info: { color: 'info', icon: 'mdi-information' },
        success: { color: 'success', icon: 'mdi-check-circle' },
        warning: { color: 'warning', icon: 'mdi-alert' },
        error: { color: 'error', icon: 'mdi-alert-circle' }
      };
      
      this.snackbar = {
        show: true,
        message,
        color: config[type].color,
        icon: config[type].icon,
        timeout: type === 'error' ? 6000 : 4000
      };
    }
  },
  
  // 🛡️ Guards de navegación
  beforeRouteEnter(to, from, next) {
    // Se ejecuta antes de entrar a la ruta
    console.log('🛡️ Entrando a LoginView desde:', from.name);
    next();
  },
  
  beforeRouteLeave(to, from, next) {
    // Se ejecuta antes de salir de la ruta
    console.log('🛡️ Saliendo de LoginView hacia:', to.name);
    next();
  }
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  background: linear-gradient(135deg, #F5F7FA, #E4EBF1);
}

/* Responsive adjustments */
@media (max-width: 600px) {
  .login-page {
    padding: 0;
  }
}
</style>