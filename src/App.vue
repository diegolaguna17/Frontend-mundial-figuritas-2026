<template>
  <div class="app-container">
    <header v-if="isAuthenticated">
      <div class="logo-container">
        <h1>Álbum <span class="gold-accent">2026</span></h1>
      </div>
      <button @click="logout" class="btn-logout">Cerrar Sesión</button>
    </header>
    <main>
      <router-view @auth-change="checkAuth" />
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
}

header {
  background: linear-gradient(135deg, var(--pitch-green) 0%, var(--pitch-green-light) 100%);
  color: white;
  padding: 1.5rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: var(--shadow-md);
  border-bottom: 4px solid var(--gold);
}

.logo-container {
  display: flex;
  align-items: center;
  gap: 1rem;
}

h1 {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 800;
  letter-spacing: 1px;
  text-transform: uppercase;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.gold-accent {
  color: var(--gold);
}

.btn-logout {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
  padding: 0.6rem 1.2rem;
  border-radius: var(--radius-md);
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(5px);
}

.btn-logout:hover {
  background: var(--gold);
  border-color: var(--gold);
  color: var(--pitch-green);
  transform: translateY(-2px);
  box-shadow: var(--shadow-sm);
}

main {
  flex: 1;
  padding: 2rem;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
}
</style>
