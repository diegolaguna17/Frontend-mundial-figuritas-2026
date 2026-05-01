<template>
  <div class="app-container">
    <!-- Decorative background blobs -->
    <div class="blob-decoration blob-1"></div>
    <div class="blob-decoration blob-2"></div>
    <div class="blob-decoration blob-3"></div>

    <header v-if="isAuthenticated">
      <nav class="navbar">
        <div class="nav-brand">
          <div class="brand-icon">⚽</div>
          <h1>ÁLBUM <span class="accent-text">2026</span></h1>
        </div>
        <button @click="logout" class="btn-logout">
          <span class="btn-text">Cerrar Sesión</span>
          <span class="btn-icon">→</span>
        </button>
      </nav>
    </header>

    <main>
      <router-view v-slot="{ Component }">
        <transition name="route" mode="out-in">
          <component :is="Component" @auth-change="checkAuth" />
        </transition>
      </router-view>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const isAuthenticated = ref(false);

const checkAuth = () => {
  isAuthenticated.value = !!localStorage.getItem('token');
};

onMounted(() => {
  checkAuth();
});

const logout = () => {
  localStorage.removeItem('token');
  checkAuth();
  router.push('/login');
};
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
}

/* Decorative Blobs */
.blob-1 {
  width: 500px;
  height: 500px;
  background: var(--wc-purple);
  top: -150px;
  right: -100px;
}
.blob-2 {
  width: 400px;
  height: 400px;
  background: var(--wc-red);
  bottom: 100px;
  left: -150px;
}
.blob-3 {
  width: 350px;
  height: 350px;
  background: var(--wc-mint);
  bottom: -100px;
  right: 20%;
}

/* Navbar */
header {
  position: sticky;
  top: 0;
  z-index: 100;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  background: rgba(13, 17, 23, 0.8);
  border-bottom: 1px solid var(--border-color);
}

.navbar {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.nav-brand {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.brand-icon {
  font-size: 1.6rem;
  animation: float 3s ease-in-out infinite;
}

h1 {
  margin: 0;
  font-size: 1.6rem;
  font-weight: 900;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--text-primary);
}

.accent-text {
  background: var(--gradient-primary);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.btn-logout {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: transparent;
  border: 1px solid var(--border-color);
  color: var(--text-secondary);
  padding: 0.5rem 1.2rem;
  border-radius: var(--radius-md);
  font-weight: 600;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.3s var(--ease-smooth);
  font-family: 'Outfit', sans-serif;
}

.btn-logout:hover {
  color: var(--wc-red);
  border-color: var(--wc-red);
  background: rgba(227, 24, 55, 0.1);
}

.btn-logout:active {
  transform: scale(0.97);
}

.btn-icon {
  transition: transform 0.3s;
}

.btn-logout:hover .btn-icon {
  transform: translateX(3px);
}

/* Main */
main {
  flex: 1;
  padding: 2rem;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  position: relative;
  z-index: 1;
}

/* Route Transitions */
.route-enter-active {
  animation: routeIn 0.35s var(--ease-smooth);
}
.route-leave-active {
  animation: routeOut 0.25s var(--ease-smooth);
}

@keyframes routeIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes routeOut {
  from { opacity: 1; transform: translateY(0); }
  to { opacity: 0; transform: translateY(-8px); }
}

/* Responsive */
@media (max-width: 768px) {
  .navbar {
    padding: 0.8rem 1rem;
  }
  h1 {
    font-size: 1.2rem;
  }
  main {
    padding: 1rem;
  }
  .btn-text {
    display: none;
  }
  .btn-icon {
    font-size: 1.1rem;
  }
}
</style>
