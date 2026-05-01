<template>
  <div class="auth-page">
    <div class="auth-decoration">
      <div class="deco-circle deco-1"></div>
      <div class="deco-circle deco-2"></div>
      <div class="deco-circle deco-3"></div>
    </div>

    <div class="auth-card">
      <div class="auth-header">
        <div class="auth-logo">⚽</div>
        <h2>Bienvenido</h2>
        <p class="auth-subtitle">Inicia sesión en tu álbum</p>
      </div>

      <form @submit.prevent="handleLogin">
        <div class="form-group">
          <label for="login-email">Email</label>
          <div class="input-wrapper">
            <span class="input-icon">✉</span>
            <input
              id="login-email"
              type="email"
              v-model="email"
              required
              placeholder="tu@email.com"
              autocomplete="email"
            />
          </div>
        </div>

        <div class="form-group">
          <label for="login-password">Contraseña</label>
          <div class="input-wrapper">
            <span class="input-icon">🔒</span>
            <input
              id="login-password"
              :type="showPassword ? 'text' : 'password'"
              v-model="password"
              required
              placeholder="••••••••"
              autocomplete="current-password"
            />
            <button
              type="button"
              class="toggle-password"
              @click="showPassword = !showPassword"
              aria-label="Alternar visibilidad de contraseña"
            >
              {{ showPassword ? '🙈' : '👁️' }}
            </button>
          </div>
        </div>

        <button type="submit" class="btn-primary" :disabled="loading">
          <span v-if="loading" class="btn-loading">
            <span class="mini-spinner"></span>
            Entrando...
          </span>
          <span v-else class="btn-content">
            Iniciar Sesión
            <span class="btn-arrow">→</span>
          </span>
        </button>

        <Transition name="fade">
          <p class="error-msg" v-if="error">{{ error }}</p>
        </Transition>
      </form>

      <p class="switch-auth">
        ¿No tienes cuenta?
        <router-link to="/register">Crea una aquí</router-link>
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

const emit = defineEmits(['auth-change']);

const handleLogin = async () => {
  loading.value = true;
  error.value = '';
  try {
    const apiUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000';
    const res = await axios.post(`${apiUrl}/api/auth/login`, {
      email: email.value,
      password: password.value
    });
    localStorage.setItem('token', res.data.data.token);
    emit('auth-change');
    router.push('/');
  } catch (err) {
    error.value = err.response?.data?.error || 'Error al iniciar sesión';
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.auth-page {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  position: relative;
  overflow: hidden;
}

/* Decorative circles */
.auth-decoration {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.deco-circle {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.2;
}

.deco-1 {
  width: 400px;
  height: 400px;
  background: var(--wc-purple);
  top: -100px;
  left: -100px;
  animation: float 8s ease-in-out infinite;
}

.deco-2 {
  width: 350px;
  height: 350px;
  background: var(--wc-red);
  bottom: -50px;
  right: -50px;
  animation: float 6s ease-in-out infinite reverse;
}

.deco-3 {
  width: 300px;
  height: 300px;
  background: var(--wc-mint);
  top: 40%;
  right: 10%;
  animation: float 10s ease-in-out infinite;
}

/* Card */
.auth-card {
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  padding: 2.5rem;
  border-radius: var(--radius-xl);
  width: 100%;
  max-width: 420px;
  position: relative;
  z-index: 1;
  backdrop-filter: blur(20px);
}

.auth-header {
  text-align: center;
  margin-bottom: 2rem;
}

.auth-logo {
  font-size: 3rem;
  margin-bottom: 0.5rem;
  animation: float 3s ease-in-out infinite;
}

h2 {
  font-size: 1.8rem;
  font-weight: 800;
  color: var(--text-primary);
  margin-bottom: 0.3rem;
}

.auth-subtitle {
  color: var(--text-secondary);
  font-size: 0.95rem;
}

/* Form */
.form-group {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon {
  position: absolute;
  left: 1rem;
  font-size: 1rem;
  z-index: 1;
  opacity: 0.5;
}

input {
  width: 100%;
  padding: 0.85rem 1rem 0.85rem 2.8rem;
  background: var(--bg-card);
  border: 2px solid var(--border-color);
  border-radius: var(--radius-md);
  font-size: 1rem;
  color: var(--text-primary);
  transition: all 0.3s var(--ease-smooth);
  font-family: 'Outfit', sans-serif;
}

input::placeholder {
  color: var(--text-muted);
}

input:focus {
  outline: none;
  border-color: var(--wc-mint);
  box-shadow: 0 0 0 4px rgba(86, 224, 192, 0.15);
}

.toggle-password {
  position: absolute;
  right: 0.8rem;
  background: none;
  border: none;
  font-size: 1.1rem;
  cursor: pointer;
  padding: 0.2rem;
  transition: transform 0.15s;
  z-index: 1;
}

.toggle-password:active {
  transform: scale(0.85);
}

/* Button */
.btn-primary {
  width: 100%;
  padding: 0.9rem;
  background: var(--gradient-primary);
  color: var(--bg-color);
  border: none;
  border-radius: var(--radius-md);
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s var(--ease-smooth);
  font-family: 'Outfit', sans-serif;
  position: relative;
  overflow: hidden;
}

.btn-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.btn-arrow {
  transition: transform 0.3s;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: var(--shadow-glow-mint);
}

.btn-primary:hover:not(:disabled) .btn-arrow {
  transform: translateX(4px);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(0) scale(0.98);
}

.btn-primary:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.btn-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.mini-spinner {
  width: 18px;
  height: 18px;
  border: 2px solid rgba(0,0,0,0.2);
  border-top-color: var(--bg-color);
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}

/* Error */
.error-msg {
  color: var(--wc-red);
  margin-top: 1rem;
  text-align: center;
  font-size: 0.9rem;
  font-weight: 500;
  padding: 0.5rem;
  background: rgba(227, 24, 55, 0.1);
  border-radius: var(--radius-sm);
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* Switch */
.switch-auth {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 0.9rem;
  color: var(--text-secondary);
}

.switch-auth a {
  color: var(--wc-mint);
  text-decoration: none;
  font-weight: 700;
  transition: color 0.2s;
}

.switch-auth a:hover {
  color: var(--wc-green-neon);
  text-decoration: underline;
}

/* Responsive */
@media (max-width: 480px) {
  .auth-card {
    margin: 1rem;
    padding: 2rem 1.5rem;
  }
}
</style>
