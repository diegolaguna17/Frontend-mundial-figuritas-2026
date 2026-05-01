<template>
  <div class="auth-container">
    <div class="auth-card">
      <h2>Registro</h2>
      <form @submit.prevent="handleRegister">
        <div class="form-group">
          <label>Email</label>
          <input type="email" v-model="email" required placeholder="tu@email.com" />
        </div>
        <div class="form-group">
          <label>Contraseña</label>
          <div class="password-wrapper">
            <input :type="showPassword ? 'text' : 'password'" v-model="password" required placeholder="••••••••" />
            <button type="button" class="toggle-password" @click="showPassword = !showPassword" aria-label="Alternar visibilidad de contraseña">
              <span v-if="showPassword">👁️‍🗨️</span>
              <span v-else>👁️</span>
            </button>
          </div>
        </div>
        <button type="submit" class="btn-primary" :disabled="loading">
          {{ loading ? 'Creando cuenta...' : 'Registrarse' }}
        </button>
        <p class="error" v-if="error">{{ error }}</p>
      </form>
      <p class="switch-auth">
        ¿Ya tienes cuenta? <router-link to="/login">Inicia sesión</router-link>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
const email = ref('');
const password = ref('');
const showPassword = ref(false);
const error = ref('');
const loading = ref(false);

const handleRegister = async () => {
  loading.value = true;
  error.value = '';
  try {
    const apiUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000';
    await axios.post(`${apiUrl}/api/auth/register`, {
      email: email.value,
      password: password.value
    });
    // Redirect to login after successful registration
    router.push('/login');
  } catch (err) {
    error.value = err.response?.data?.error || 'Error al registrar usuario';
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.auth-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(100vh - 150px);
}

.auth-card {
  background: white;
  padding: 3rem;
  border-radius: 16px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.05);
  width: 100%;
  max-width: 400px;
}

h2 {
  text-align: center;
  margin-bottom: 2rem;
  color: #1e3c72;
}

.form-group {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: #555;
}

input {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

input:focus {
  outline: none;
  border-color: #2a5298;
}

.password-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.password-wrapper input {
  padding-right: 3rem;
}

.toggle-password {
  position: absolute;
  right: 0.5rem;
  background: none;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 0.2rem;
  color: #777;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.1s;
}

.toggle-password:active {
  transform: scale(0.9);
}

.btn-primary {
  width: 100%;
  padding: 0.75rem;
  background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(30,60,114,0.3);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(0) scale(0.98);
}

.btn-primary:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.error {
  color: #e74c3c;
  margin-top: 1rem;
  text-align: center;
  font-size: 0.9rem;
}

.switch-auth {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 0.9rem;
}

.switch-auth a {
  color: #2a5298;
  text-decoration: none;
  font-weight: 600;
}

.switch-auth a:hover {
  text-decoration: underline;
}
</style>
