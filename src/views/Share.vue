<template>
  <div class="home">
    <div v-if="loading" class="feedback-card">
      <h2>Cargando colección...</h2>
    </div>

    <div v-else-if="error" class="feedback-card error">
      <span class="feedback-icon">⚠️</span>
      <p>{{ error }}</p>
    </div>

    <div v-else>
      <!-- Hero Progress Section -->
      <div class="hero-section">
        <div class="hero-content">
          <div class="hero-left">
            <h2 class="hero-title">
              Colección <span class="gradient-text">Compartida</span>
            </h2>
            <p class="hero-subtitle">Modo de solo lectura</p>
          </div>
          <div class="hero-right">
            <div class="progress-ring-container">
              <svg class="progress-ring" viewBox="0 0 120 120">
                <circle class="ring-bg" cx="60" cy="60" r="52" />
                <circle
                  class="ring-fill"
                  cx="60" cy="60" r="52"
                  :style="{ strokeDashoffset: ringOffset }"
                />
              </svg>
              <div class="ring-label">
                <span class="ring-percentage">{{ globalPercentage }}%</span>
              </div>
            </div>
            <div class="progress-detail">
              <span class="progress-count">{{ progresoTotal }}</span>
              <span class="progress-divider">/</span>
              <span class="progress-total">994</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Toolbar: Search, Filters, Sort -->
      <div class="toolbar">
        <div class="search-box">
          <span class="search-icon">🔍</span>
          <input 
            type="text" 
            v-model="searchQuery" 
            placeholder="Buscar país, código o figurita..."
            class="search-input"
          />
          <button v-if="searchQuery" class="search-clear" @click="searchQuery = ''">✕</button>
        </div>
        
        <div class="controls">
          <div class="filter-group">
            <button 
              v-for="f in filters" :key="f.value"
              :class="['filter-btn', { active: filterType === f.value }]" 
              @click="filterType = f.value"
            >{{ f.label }}</button>
          </div>

          <div class="sort-group">
            <select v-model="sortBy" class="sort-select">
              <option value="country">Por País</option>
              <option value="progress-desc">Mayor Progreso</option>
              <option value="progress-asc">Menor Progreso</option>
            </select>
          </div>
        </div>
      </div>

      <div v-if="filteredAndSortedColeccion.length === 0" class="feedback-card empty">
        <span class="feedback-icon">🔎</span>
        <p>No se encontraron resultados para "<strong>{{ searchQuery }}</strong>"</p>
      </div>
      
      <TransitionGroup name="grid-item" tag="div" class="countries-grid">
        <CountryCard
          v-for="country in filteredAndSortedColeccion"
          :key="country.codigo"
          :country="country"
          :filter-type="filterType"
          @click="selectCountry(country)"
        />
      </TransitionGroup>
    </div>

    <!-- Modal -->
    <Transition name="modal">
      <div v-if="selectedCountry" class="modal-overlay" @click.self="closeModal">
        <div class="modal-content">
          <div class="modal-header">
            <div class="modal-title-group">
              <h3>{{ selectedCountry.pais }}</h3>
              <span class="modal-code">{{ selectedCountry.codigo }}</span>
            </div>
            <button class="close-btn" @click="closeModal">
              <span>✕</span>
            </button>
          </div>
          <StickerGrid
            :country="selectedCountry"
            :filter-type="filterType"
            :search-query="searchQuery"
            :read-only="true"
          />
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import axios from 'axios';
import CountryCard from '../components/CountryCard.vue';
import StickerGrid from '../components/StickerGrid.vue';

const route = useRoute();
const router = useRouter();

const coleccion = ref([]);
const progresoTotal = ref(0);
const loading = ref(true);
const error = ref('');
const selectedCountry = ref(null);

const searchQuery = ref('');
const filterType = ref('all');
const sortBy = ref('country');

const filters = [
  { value: 'all', label: 'Todas' },
  { value: 'obtained', label: 'Obtenidas' },
  { value: 'missing', label: 'Faltantes' },
  { value: 'repeated', label: 'Repetidas' }
];

const globalPercentage = computed(() => {
  return ((progresoTotal.value / 994) * 100).toFixed(0);
});

const ringOffset = computed(() => {
  const circumference = 2 * Math.PI * 52;
  const percent = progresoTotal.value / 994;
  return circumference - (percent * circumference);
});

const filteredAndSortedColeccion = computed(() => {
  let result = coleccion.value;

  const query = searchQuery.value.toLowerCase().trim();
  if (query) {
    result = result.filter(c => {
      const matchCountry = c.pais.toLowerCase().includes(query);
      const matchCode = c.codigo.toLowerCase().includes(query);
      const matchSticker = c.figuritas.some(f => f.figura.toLowerCase().includes(query));
      return matchCountry || matchCode || matchSticker;
    });
  }

  if (filterType.value !== 'all') {
    result = result.filter(c => {
      const matchingStickers = c.figuritas.filter(f => {
        if (filterType.value === 'obtained') return f.tiene;
        if (filterType.value === 'missing') return !f.tiene;
        if (filterType.value === 'repeated') return f.repetidas > 0;
        return true;
      });
      return matchingStickers.length > 0;
    });
  }

  result = [...result].sort((a, b) => {
    if (sortBy.value === 'country') {
      return 0;
    } else {
      const progA = a.figuritas.filter(f => f.tiene).length;
      const progB = b.figuritas.filter(f => f.tiene).length;
      if (sortBy.value === 'progress-desc') return progB - progA;
      if (sortBy.value === 'progress-asc') return progA - progB;
    }
  });

  return result;
});

const fetchColeccion = async () => {
  try {
    const idUsuario = route.params.id;
    const apiUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000';
    const res = await axios.get(`${apiUrl}/api/coleccion/share/${idUsuario}`);
    
    coleccion.value = res.data.data.coleccion;
    progresoTotal.value = res.data.data.progreso_total;
    
    if (route.query.country) {
      selectedCountry.value = coleccion.value.find(c => c.codigo === route.query.country) || null;
    }
  } catch (err) {
    error.value = 'Error al cargar la colección. Es posible que el link sea inválido o el usuario no exista.';
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchColeccion();
});

watch(
  () => route.query.country,
  (newCountryCode) => {
    if (newCountryCode && coleccion.value.length > 0) {
      selectedCountry.value = coleccion.value.find(c => c.codigo === newCountryCode) || null;
    } else {
      selectedCountry.value = null;
    }
  }
);

const selectCountry = (country) => {
  router.push({ query: { ...route.query, country: country.codigo } });
};

const closeModal = () => {
  const query = { ...route.query };
  delete query.country;
  router.push({ query });
};
</script>

<style scoped>
/* Resumen de estilos necesarios (los mismos que Home.vue) */
.hero-section {
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-xl);
  padding: 2rem;
  margin-bottom: 1.5rem;
  position: relative;
  overflow: hidden;
}

.hero-section::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: var(--gradient-hero);
}

.hero-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1.5rem;
}

.hero-title {
  font-size: 2.2rem;
  font-weight: 900;
  color: var(--text-primary);
  line-height: 1.1;
}

.gradient-text {
  background: var(--gradient-primary);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero-subtitle {
  color: var(--text-muted);
  font-size: 0.95rem;
  margin-top: 0.5rem;
  font-weight: 500;
  letter-spacing: 1px;
  text-transform: uppercase;
}

.hero-right {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.progress-ring-container {
  position: relative;
  width: 90px;
  height: 90px;
}

.progress-ring {
  width: 100%;
  height: 100%;
  transform: rotate(-90deg);
}

.ring-bg {
  fill: none;
  stroke: var(--bg-card);
  stroke-width: 8;
}

.ring-fill {
  fill: none;
  stroke: url(#ring-gradient);
  stroke: var(--wc-mint);
  stroke-width: 8;
  stroke-linecap: round;
  stroke-dasharray: 326.7;
  transition: stroke-dashoffset 0.8s var(--ease-smooth);
}

.ring-label {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.ring-percentage {
  font-size: 1.3rem;
  font-weight: 800;
  color: var(--wc-mint);
}

.progress-detail {
  display: flex;
  align-items: baseline;
  gap: 0.2rem;
}

.progress-count {
  font-size: 2rem;
  font-weight: 900;
  color: var(--text-primary);
}

.progress-divider {
  font-size: 1.2rem;
  color: var(--text-muted);
  margin: 0 0.15rem;
}

.progress-total {
  font-size: 1.2rem;
  color: var(--text-muted);
  font-weight: 600;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-bottom: 1.5rem;
  align-items: center;
}

.search-box {
  flex: 1;
  min-width: 250px;
  position: relative;
}

.search-icon {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  font-size: 0.9rem;
  opacity: 0.5;
}

.search-input {
  width: 100%;
  padding: 0.75rem 2.5rem 0.75rem 2.5rem;
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  color: var(--text-primary);
  font-size: 0.95rem;
  font-family: 'Outfit', sans-serif;
  transition: all 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: var(--wc-mint);
  box-shadow: 0 0 0 3px rgba(86, 224, 192, 0.12);
}

.search-clear {
  position: absolute;
  right: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  background: var(--bg-elevated);
  border: none;
  color: var(--text-muted);
  width: 22px;
  height: 22px;
  border-radius: 50%;
  font-size: 0.7rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  align-items: center;
}

.filter-group {
  display: flex;
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 3px;
}

.filter-btn {
  background: none;
  border: none;
  padding: 0.45rem 1rem;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 600;
  font-size: 0.85rem;
  color: var(--text-muted);
  transition: all 0.25s;
  font-family: 'Outfit', sans-serif;
}

.filter-btn.active {
  background: var(--wc-mint);
  color: var(--bg-color);
}

.sort-select {
  padding: 0.5rem 1rem;
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  color: var(--text-primary);
  font-weight: 600;
  font-size: 0.85rem;
  cursor: pointer;
  font-family: 'Outfit', sans-serif;
}

.countries-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.25rem;
}

.grid-item-enter-active {
  transition: all 0.4s var(--ease-smooth);
}
.grid-item-leave-active {
  transition: all 0.3s var(--ease-smooth);
}
.grid-item-enter-from,
.grid-item-leave-to {
  opacity: 0;
  transform: scale(0.92);
}

.feedback-card {
  text-align: center;
  padding: 4rem 2rem;
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-xl);
}

.feedback-icon {
  font-size: 3rem;
  display: block;
  margin-bottom: 1rem;
}

.feedback-card p {
  color: var(--text-secondary);
  font-size: 1.1rem;
}

.feedback-card.error {
  border-color: rgba(227, 24, 55, 0.3);
}

.feedback-card.error p {
  color: var(--wc-red);
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(12px);
}

.modal-content {
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  width: 95%;
  max-width: 900px;
  max-height: 90vh;
  border-radius: var(--radius-xl);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-header {
  padding: 1.5rem 2rem;
  background: var(--bg-card);
  border-bottom: 1px solid var(--border-color);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-title-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.modal-title-group h3 {
  margin: 0;
  font-size: 1.6rem;
  font-weight: 800;
  color: var(--text-primary);
}

.modal-code {
  background: var(--wc-mint);
  color: var(--bg-color);
  padding: 0.2rem 0.6rem;
  border-radius: 6px;
  font-weight: 800;
  font-size: 0.8rem;
  letter-spacing: 1px;
}

.close-btn {
  background: var(--bg-elevated);
  border: 1px solid var(--border-color);
  width: 36px;
  height: 36px;
  border-radius: 50%;
  color: var(--text-muted);
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 0.9rem;
  transition: all 0.2s;
}

.close-btn:hover {
  background: var(--wc-red);
  border-color: var(--wc-red);
  color: white;
  transform: rotate(90deg);
}

.modal-enter-active {
  transition: opacity 0.3s;
}
.modal-enter-active .modal-content {
  animation: modalIn 0.35s var(--ease-spring);
}
.modal-leave-active {
  transition: opacity 0.25s;
}
.modal-leave-active .modal-content {
  animation: modalOut 0.25s var(--ease-smooth);
}
.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

@keyframes modalIn {
  from { opacity: 0; transform: translateY(30px) scale(0.95); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}
@keyframes modalOut {
  from { opacity: 1; transform: translateY(0) scale(1); }
  to { opacity: 0; transform: translateY(15px) scale(0.97); }
}

@media (max-width: 768px) {
  .hero-title {
    font-size: 1.6rem;
  }
  .hero-content {
    flex-direction: column;
    text-align: center;
  }
  .hero-right {
    justify-content: center;
  }
  .toolbar {
    flex-direction: column;
    align-items: stretch;
  }
  .controls {
    flex-direction: column;
    align-items: stretch;
  }
  .filter-group {
    justify-content: space-between;
  }
  .countries-grid {
    grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  }
}

@media (max-width: 480px) {
  .hero-section {
    padding: 1.5rem;
  }
  .progress-ring-container {
    width: 70px;
    height: 70px;
  }
  .progress-count {
    font-size: 1.5rem;
  }
}
</style>
