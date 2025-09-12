<template>
  <v-card 
    class="login-card elevation-0" 
    :loading="isLoading"
    max-width="400"
  >
    <!-- Header con Logo -->
    <v-card-title class="login-header justify-center pa-8 pb-4">
      <div  class="full-width">
        <!-- Logo de la empresa -->
        <div class="logo-container mb-6 text-center">
          <div
            v-if="logoUrl && !logoError"
            class="logo-image-container"
          >
            <v-img
              :src="logoUrl"
              :alt="appTitle + ' Logo'"
              @error="handleLogoError"
              contain
              max-height="80"
              class="logo-image"
            />
          </div>
          
          <!-- Fallback con texto -->
          <div
            v-else
            class="logo-fallback"
          >
            <span class="logo-text">
              {{ logoPlaceholder }}
            </span>
          </div>
        </div>
        
        <!-- Línea divisoria elegante -->
        <div class="divider-container mb-6">
          <v-divider class="custom-divider"></v-divider>
        </div>
        
      </div>
    </v-card-title>

    <!-- Formulario de Login -->
    <v-card-text class="pa-8 pt-0">
      <v-form
        ref="loginForm"
        v-model="formValid"
        @submit.prevent="handleLogin"
        lazy-validation
      >
        <!-- Campo Email -->
        <div class="input-group mb-4">
          <label class="input-label">Email</label>
          <v-text-field
            v-model="formData.email"
            :rules="emailRules"
            :error-messages="fieldErrors.email"
            placeholder="login@gmail.com"
            type="email"
            flat
            solo
            dense
            class="custom-input"
            autocomplete="email"
            :disabled="isLoading"
            @input="clearFieldError('email')"
            @keyup.enter="focusPassword"
            hide-details="auto"
          />
        </div>

        <!-- Campo Contraseña -->
        <div class="input-group mb-6">
          <div class="d-flex justify-space-between align-center mb-2">
            <label class="input-label">Password</label>
            <v-btn
              text
              x-small
              color="grey"
              class="forgot-link"
              :disabled="isLoading"
              @click="handleForgotPassword"
            >
              Olvidaste tu contraseña?
            </v-btn>
          </div>
          <v-text-field
            ref="passwordField"
            v-model="formData.password"
            :rules="passwordRules"
            :error-messages="fieldErrors.password"
            :type="showPassword ? 'text' : 'password'"
            :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
            placeholder="••••••••••••"
            flat
            solo
            dense
            class="custom-input"
            autocomplete="current-password"
            :disabled="isLoading"
            @click:append="showPassword = !showPassword"
            @input="clearFieldError('password')"
            @keyup.enter="handleLogin"
            hide-details="auto"
          />
        </div>

        <!-- Mensaje de error general -->
        <v-alert
          v-if="generalError"
          type="error"
          dense
          text
          class="mb-4"
          dismissible
          @input="generalError = ''"
        >
          {{ generalError }}
        </v-alert>

        <!-- Botón de Login -->
        <v-btn
          :loading="isLoading"
          :disabled="!formValid || isLoading"
          color="#E8A5C4"
          large
          block
          depressed
          class="login-btn mb-6"
          @click="handleLogin"
        >
          LOGIN
          <v-icon right small class="ml-2">mdi-arrow-right</v-icon>
        </v-btn>

         <!-- Footer con información adicional -->
    <v-divider></v-divider>
    <v-card-actions class="pa-4 justify-center">
      <div class="text-center">
        <p class="text-caption grey--text text--darken-1 mb-1">
          ¿Necesitas ayuda?
        </p>
        <v-btn
          text
          x-small
          color="primary"
          :disabled="isLoading"
          @click="showHelpDialog = true"
        >
          Contactar Soporte
        </v-btn>
      </div>
    </v-card-actions>

    <!-- Dialog de Ayuda -->
    <v-dialog v-model="showHelpDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">
          Soporte Técnico
        </v-card-title>
        <v-card-text>
          <p>Si tienes problemas para acceder:</p>
          <ul class="ml-4">
            <li>Verifica tu correo y contraseña</li>
            <li>Usa "Olvidé mi contraseña"</li>
            <li>Contacta al administrador</li>
          </ul>
          <v-divider class="my-3"></v-divider>
          <p class="text-caption grey--text">
            📧 soporte@empresa.com<br>
            📞 +52 222 123 4567
          </p>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="showHelpDialog = false">
            Cerrar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
      </v-form>
    </v-card-text>
  </v-card>
</template>

<script>
export default {
  name: 'LoginComponent',
  
  props: {
    logoUrl: {
      type: String,
      default: ''
    },
    logoPlaceholder: {
      type: String,
      default: 'ArJes'
    },
    appTitle: {
      type: String,
      default: 'ArJess'
    },
    appSubtitle: {
      type: String,
      default: 'Punto de venta'
    }
  },
  
  data() {
    return {
      // Estados del formulario
      formValid: false,
      isLoading: false,
      showPassword: false,
      logoError: false,
      showHelpDialog: false,
      
      // Datos del formulario
      formData: {
        email: '',
        password: '',
        rememberMe: false
      },
      
      // Errores
      generalError: '',
      fieldErrors: {
        email: [],
        password: []
      },
      
      // Reglas de validación
      emailRules: [
        v => /.+@.+\..+/.test(v) || 'Debe ser un correo válido'
      ],
      
      passwordRules: [
        v => (v && v.length >= 6) || 'Mínimo 6 caracteres'
      ]
    }
  },
  
  mounted() {
    // Auto-focus en el campo email
    this.$nextTick(() => {
      if (this.$refs.loginForm) {
        const emailField = this.$refs.loginForm.$el.querySelector('input[type="email"]');
        if (emailField) {
          emailField.focus();
        }
      }
    });
    
    // Cargar datos guardados si "recordar sesión" estaba activo
    this.loadSavedCredentials();
  },
  
  methods: {
    // 🔐 Manejar el login
    async handleLogin() {
      if (!this.$refs.loginForm.validate()) {
        this.emitMessage('Por favor, completa todos los campos correctamente', 'warning');
        return;
      }
      
      this.isLoading = true;
      this.generalError = '';
      this.clearAllFieldErrors();
      
      try {
        console.log('🔐 Intentando login...', {
          email: this.formData.email,
          rememberMe: this.formData.rememberMe
        });
        
        // 📡 AQUÍ HARÍAS LA LLAMADA A TU API
        const loginData = await this.performLogin({
          email: this.formData.email,
          password: this.formData.password,
          rememberMe: this.formData.rememberMe
        });
        
        // Guardar credenciales si se seleccionó "recordar"
        if (this.formData.rememberMe) {
          this.saveCredentials();
        } else {
          this.clearSavedCredentials();
        }
        
        // Emitir evento de éxito
        this.$emit('login-success', loginData);
        
      } catch (error) {
        console.error('❌ Error en login:', error);
        this.handleLoginError(error);
      } finally {
        this.isLoading = false;
      }
    },
    
    // 📡 Simular llamada de login (reemplazar con tu API)
    async performLogin(credentials) {
      // 🎭 SIMULACIÓN - Reemplazar con llamada real
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          // Simular diferentes casos
          if (credentials.email === 'admin@empresa.com' && credentials.password === '123456') {
            resolve({
              token: 'fake-jwt-token-' + Date.now(),
              user: {
                id: 1,
                name: 'Administrador',
                email: credentials.email,
                role: 'admin',
                avatar: null
              }
            });
          } else if (credentials.email === 'usuario@empresa.com' && credentials.password === '123456') {
            resolve({
              token: 'fake-jwt-token-' + Date.now(),
              user: {
                id: 2,
                name: 'Usuario Demo',
                email: credentials.email,
                role: 'user',
                avatar: null
              }
            });
          } else if (credentials.email === 'error@test.com') {
            reject({
              message: 'Error del servidor',
              errors: {
                email: ['Este correo tiene problemas técnicos']
              }
            });
          } else {
            reject({
              message: 'Credenciales incorrectas',
              status: 401
            });
          }
        }, 1500); // Simular delay de red
      });
    },
    
    // ❌ Manejar errores de login
    handleLoginError(error) {
      if (error.status === 401) {
        this.generalError = 'Correo o contraseña incorrectos';
      } else if (error.errors) {
        // Errores específicos de campos
        this.fieldErrors = { ...this.fieldErrors, ...error.errors };
      } else {
        this.generalError = error.message || 'Error al iniciar sesión';
      }
      
      this.emitMessage(this.generalError || 'Error al iniciar sesión', 'error');
    },
    
    // 🔑 Manejar "Olvidé mi contraseña"
    handleForgotPassword() {
      const email = this.formData.email;
      
      if (!email) {
        this.emitMessage('Ingresa tu correo para recuperar la contraseña', 'warning');
        return;
      }
      
      this.$emit('forgot-password', email);
    },
    
    // 📝 Manejar solicitud de registro
    handleRegisterRequest() {
      this.$emit('register-request');
    },
    
    // 🖼️ Manejar error de logo
    handleLogoError() {
      this.logoError = true;
      this.$emit('logo-error');
    },
    
    // 🔧 Enfocar campo de contraseña
    focusPassword() {
      if (this.$refs.passwordField) {
        this.$refs.passwordField.focus();
      }
    },
    
    // 🧹 Limpiar errores
    clearFieldError(field) {
      if (this.fieldErrors[field]) {
        this.fieldErrors[field] = [];
      }
    },
    
    clearAllFieldErrors() {
      this.fieldErrors = {
        email: [],
        password: []
      };
    },
    
    // 💾 Guardar/cargar credenciales
    saveCredentials() {
      const credentials = {
        email: this.formData.email,
        rememberMe: true
      };
      localStorage.setItem('savedLoginCredentials', JSON.stringify(credentials));
    },
    
    loadSavedCredentials() {
      const saved = localStorage.getItem('savedLoginCredentials');
      if (saved) {
        const credentials = JSON.parse(saved);
        this.formData.email = credentials.email || '';
        this.formData.rememberMe = credentials.rememberMe || false;
      }
    },
    
    clearSavedCredentials() {
      localStorage.removeItem('savedLoginCredentials');
    },
    
    // 📢 Emitir mensajes
    emitMessage(message, type) {
      this.$emit('show-message', { message, type });
    }
  }
}
</script>

<style scoped>
.login-card {
  border-radius: 24px !important;
  background: white !important;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1) !important;
}

.full-width {
  width: 100%;
}

.login-header {
  background: transparent;
}

.logo-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.logo-image-container {
  max-width: 300px;
  width: 100%;
}

.logo-image {
  width: 100%;
}

.logo-fallback {
  background: #60a498;
  border-radius: 8px;
  padding: 12px 24px;
  display: inline-block;
}

.logo-text {
  color: white;
  font-size: 18px;
  font-weight: 500;
}

/* Estilos para la línea divisoria */
.divider-container {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.custom-divider {
  background-color: #E0E0E0 !important;
  height: 1px !important;
  width: 60% !important;
  opacity: 0.6;
}

.welcome-text {
  color: #666;
  font-size: 14px;
  margin: 0;
}

.main-title {
  font-family: 'Montserrat', sans-serif;
  color: #333;
  font-size: 35px;
  font-weight: 100;
  margin: 10;          /* quitar margen extra */
  padding: 0;         /* quitar padding interno */
  text-align: left;   /* asegurar alineación izquierda */
}

.input-group {
  margin-bottom: 20px;
}

.input-label {
  color: #333;
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 8px;
  display: block;
}

.custom-input {
  background-color: #E3F2FD !important;
  border-radius: 8px !important;
}

.custom-input >>> .v-input__control {
  min-height: 48px !important;
}

.custom-input >>> .v-input__slot {
  background-color: #E3F2FD !important;
  border-radius: 8px !important;
  box-shadow: none !important;
  margin-bottom: 0 !important;
}

.custom-input >>> .v-text-field__slot input {
  color: #333 !important;
  font-size: 14px !important;
}

.custom-input >>> .v-text-field__slot input::placeholder {
  color: #999 !important;
}

.forgot-link {
  color: #999 !important;
  text-decoration: none !important;
  font-size: 12px !important;
  text-transform: none !important;
  height: auto !important;
  min-width: auto !important;
  padding: 0 !important;
}

.forgot-link:hover {
  color: #666 !important;
}

.login-btn {
  background-color: #E091B8 !important;
  color: white !important;
  font-weight: 600 !important;
  font-size: 14px !important;
  letter-spacing: 1px !important;
  text-transform: uppercase !important;
  height: 48px !important;
  border-radius: 24px !important;
  box-shadow: none !important;
  transition: all 0.3s ease !important;
}

.login-btn:hover {
  background-color: #559282 !important;
  transform: translateY(-1px);
  box-shadow: 0 4px 15px rgba(232, 165, 196, 0.3) !important;
}

.login-btn:disabled {
  transform: none !important;
  opacity: 0.6 !important;
}

.signup-text {
  color: #999;
  font-size: 14px;
}

.signup-link {
  color: #E8A5C4 !important;
  text-decoration: none !important;
  font-size: 14px !important;
  text-transform: none !important;
  font-weight: 500 !important;
  height: auto !important;
  min-width: auto !important;
  padding: 0 4px !important;
}

.signup-link:hover {
  color: #E091B8 !important;
  text-decoration: underline !important;
}

/* Responsive para móviles */
@media (max-width: 600px) {
  .login-card {
    margin: 16px;
    border-radius: 16px !important;
  }
  
  .main-title {
    font-size: 28px;
  }
  
  .login-header {
    padding: 32px 24px 16px !important;
  }
  
  .v-card-text {
    padding: 24px !important;
    padding-top: 0 !important;
  }
  
  .custom-divider {
    width: 80% !important;
  }
}

/* Animación de carga */
.v-card--loading {
  position: relative;
  overflow: hidden;
}

.v-card--loading::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 3px;
  background: linear-gradient(90deg, transparent, rgba(232, 165, 196, 0.4), transparent);
  animation: loading 1.5s infinite;
  z-index: 1;
}

@keyframes loading {
  0% { left: -100%; }
  100% { left: 100%; }
}

/* Estilos para el estado de carga */
.v-input--is-disabled {
  opacity: 0.7;
}
</style>