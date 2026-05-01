<template>
  <div class="home">
    <div class="dashboard-header">
      <div class="header-info">
        <h2>Tu Colección 🏆</h2>
        <div class="progress-badge">
          <span class="label">Progreso Global:</span>
          <div class="progress-text">{{ progresoTotal }} / 994 <span class="percentage">({{ globalPercentage }}%)</span></div>
          <div class="progress-bar-global">
            <div class="fill-global" :style="{ width: globalPercentage + '%' }"></div>
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
          placeholder="Buscar país, código o figurita (ej. ARG 1)..."
          class="search-input"
        />
      </div>
      
      <div class="controls">
        <div class="filter-group">
          <button 
            :class="['filter-btn', { active: filterType === 'all' }]" 
            @click="filterType = 'all'"
          >Todas</button>
          <button 
            :class="['filter-btn', { active: filterType === 'obtained' }]" 
            @click="filterType = 'obtained'"
          >Obtenidas</button>
          <button 
            :class="['filter-btn', { active: filterType === 'missing' }]" 
            @click="filterType = 'missing'"
          >Faltantes</button>
        </div>

        <div class="sort-group">
          <label>Ordenar:</label>
          <select v-model="sortBy" class="sort-select">
            <option value="country">Por País</option>
            <option value="progress-desc">Mayor Progreso</option>
            <option value="progress-asc">Menor Progreso</option>
          </select>
        </div>
      </div>
    </div>

    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>Cargando tu álbum...</p>
    </div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <div v-if="filteredAndSortedColeccion.length === 0" class="no-results">
        <p>No se encontraron resultados para tu búsqueda.</p>
      </div>
      
      <div class="countries-grid">
        <CountryCard
          v-for="country in filteredAndSortedColeccion"
          :key="country.codigo"
          :country="country"
          :filter-type="filterType"
          @click="selectCountry(country)"
        />
      </div>
    </div>

    <!-- Modal para ver figuritas de un país -->
    <div v-if="selectedCountry" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <div class="modal-header">
          <div class="modal-title-group">
            <h3>{{ selectedCountry.pais }}</h3>
            <span class="modal-code">{{ selectedCountry.codigo }}</span>
          </div>
          <button class="close-btn" @click="closeModal">&times;</button>
        </div>
        <StickerGrid
          :country="selectedCountry"
          :filter-type="filterType"
          :search-query="searchQuery"
          @update-sticker="handleUpdateSticker"
        />
      </div>
    </div>
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

// Toolbar state
const searchQuery = ref('');
const filterType = ref('all'); // 'all', 'obtained', 'missing'
const sortBy = ref('country'); // 'country', 'progress-desc', 'progress-asc'

const globalPercentage = computed(() => {
  return ((progresoTotal.value / 994) * 100).toFixed(0); // Precise percentage with 0 decimals
});

// Computed property to filter and sort the collection
const filteredAndSortedColeccion = computed(() => {
  let result = coleccion.value;

  // 1. Apply Search Query
  const query = searchQuery.value.toLowerCase().trim();
  if (query) {
    result = result.filter(c => {
      const matchCountry = c.pais.toLowerCase().includes(query);
      const matchCode = c.codigo.toLowerCase().includes(query);
      const matchSticker = c.figuritas.some(f => f.figura.toLowerCase().includes(query));
      return matchCountry || matchCode || matchSticker;
    });
  }

  // 2. Apply Filters (if 'obtained', only show countries that have obtained stickers that match query, etc. Actually easier: just filter countries if they have >0 matching stickers according to the current filter)
  if (filterType.value !== 'all') {
    result = result.filter(c => {
      // How many stickers in this country match the current filter?
      const matchingStickers = c.figuritas.filter(f => {
        if (filterType.value === 'obtained') return f.obtenida;
        if (filterType.value === 'missing') return !f.obtenida;
        return true;
      });
      return matchingStickers.length > 0;
    });
  }

  // 3. Apply Sorting
  result = [...result].sort((a, b) => {
    if (sortBy.value === 'country') {
      // assuming original array is ordered by album order (we don't want to mess it up if they sort by country, maybe we leave it as is if country is selected, because original order is FWC, MEX, etc).
      // Let's rely on the original index if 'country' is selected. We can do that by just not sorting or using a stored original index.
      // But if we must sort:
      return 0; // Keep original album order
    } else {
      const progA = a.figuritas.filter(f => f.obtenida).length;
      const progB = b.figuritas.filter(f => f.obtenida).length;
      
      if (sortBy.value === 'progress-desc') return progB - progA;
      if (sortBy.value === 'progress-asc') return progA - progB;
    }
  });

  return result;
});

const fetchColeccion = async () => {
  try {
    const token = localStorage.getItem('token');
    const apiUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000';
    const res = await axios.get(`${apiUrl}/api/coleccion`, {
      headers: { Authorization: `Bearer ${token}` }
    });
    coleccion.value = res.data.data.coleccion;
    progresoTotal.value = res.data.data.progreso_total;
    
    if (route.query.country) {
      selectedCountry.value = coleccion.value.find(c => c.codigo === route.query.country) || null;
    }
  } catch (err) {
    error.value = 'Error al cargar la colección. Por favor, intenta de nuevo.';
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

const handleUpdateSticker = async (codigo_pais, figura, nuevaObtenida) => {
  try {
    const token = localStorage.getItem('token');
    // Optimistic update locally
    const country = coleccion.value.find(c => c.codigo === codigo_pais);
    let previousState = false;
    if (country) {
      const sticker = country.figuritas.find(f => f.figura === figura);
      if (sticker) {
        previousState = sticker.obtenida;
        sticker.obtenida = nuevaObtenida;
      }
    }
    
    // Recalculate global progress locally
    let total = 0;
    coleccion.value.forEach(c => {
      c.figuritas.forEach(f => {
        if (f.obtenida) total++;
      });
    });
    progresoTotal.value = total;

    // Send request
    const apiUrl = import.meta.env.VITE_API_URL || 'http://localhost:5000';
    await axios.put(`${apiUrl}/api/coleccion/figurita`, {
      codigo_pais,
      figura,
      obtenida: nuevaObtenida
    }, {
      headers: { Authorization: `Bearer ${token}` }
    });

  } catch (err) {
    console.error('Error al actualizar figurita', err);
    // Revert on error
    alert('No se pudo guardar la figurita en el servidor.');
  }
};
</script>

<style scoped>
.dashboard-header {
  background: var(--card-bg);
  border-radius: var(--radius-lg);
  padding: 2rem;
  box-shadow: var(--shadow-sm);
  margin-bottom: 2rem;
  border-left: 6px solid var(--gold);
}

.header-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.header-info h2 {
  color: var(--pitch-green);
  font-size: 2rem;
  font-weight: 800;
}

.progress-badge {
  background: var(--bg-color);
  padding: 1rem 1.5rem;
  border-radius: var(--radius-md);
  min-width: 300px;
}

.progress-badge .label {
  font-size: 0.9rem;
  color: var(--text-muted);
  font-weight: 600;
  text-transform: uppercase;
  margin-bottom: 0.5rem;
  display: block;
}

.progress-text {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--pitch-green);
  margin-bottom: 0.5rem;
}

.progress-text .percentage {
  font-size: 1rem;
  color: var(--gold);
}

.progress-bar-global {
  height: 10px;
  background: #e0e0e0;
  border-radius: 5px;
  overflow: hidden;
}

.fill-global {
  height: 100%;
  background: linear-gradient(90deg, var(--pitch-green) 0%, var(--gold) 100%);
  border-radius: 5px;
  transition: width 0.5s ease-out;
}

/* Toolbar */
.toolbar {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  margin-bottom: 2rem;
  background: var(--card-bg);
  padding: 1rem;
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
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
  color: var(--text-muted);
}

.search-input {
  width: 100%;
  padding: 0.8rem 1rem 0.8rem 2.5rem;
  border: 2px solid #eee;
  border-radius: var(--radius-md);
  font-size: 1rem;
  transition: var(--smooth-transition);
}

.search-input:focus {
  outline: none;
  border-color: var(--pitch-green);
  box-shadow: 0 0 0 3px rgba(10, 92, 54, 0.1);
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  align-items: center;
}

.filter-group {
  display: flex;
  background: var(--bg-color);
  border-radius: 8px;
  padding: 0.25rem;
}

.filter-btn {
  background: none;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  color: var(--text-muted);
  transition: all 0.2s;
}

.filter-btn:hover {
  color: var(--text-main);
}

.filter-btn.active {
  background: white;
  color: var(--pitch-green);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.sort-group {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.sort-group label {
  font-weight: 600;
  color: var(--text-muted);
}

.sort-select {
  padding: 0.5rem 1rem;
  border: 2px solid #eee;
  border-radius: 8px;
  background: white;
  font-weight: 600;
  color: var(--text-main);
  cursor: pointer;
}

.sort-select:focus {
  outline: none;
  border-color: var(--pitch-green);
}

/* Grid */
.countries-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
}

.loading, .error, .no-results {
  text-align: center;
  padding: 4rem;
  font-size: 1.2rem;
  color: var(--text-muted);
  background: var(--card-bg);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
}

.spinner {
  border: 4px solid rgba(0,0,0,0.1);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border-left-color: var(--pitch-green);
  animation: spin 1s linear infinite;
  margin: 0 auto 1rem auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error {
  color: #e74c3c;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(10, 92, 54, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(8px);
}

.modal-content {
  background: var(--bg-color);
  width: 95%;
  max-width: 900px;
  max-height: 90vh;
  border-radius: var(--radius-lg);
  display: flex;
  flex-direction: column;
  box-shadow: var(--shadow-lg);
  animation: modalIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  overflow: hidden;
}

@keyframes modalIn {
  from { opacity: 0; transform: translateY(30px) scale(0.95); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}

.modal-header {
  padding: 1.5rem 2rem;
  background: var(--card-bg);
  border-bottom: 2px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-title-group {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.modal-title-group h3 {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 800;
  color: var(--pitch-green);
}

.modal-code {
  background: var(--gold);
  color: white;
  padding: 0.3rem 0.6rem;
  border-radius: 4px;
  font-weight: 700;
  font-size: 0.9rem;
}

.close-btn {
  background: #f1f2f6;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  font-size: 1.5rem;
  color: var(--text-muted);
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.2s;
}

.close-btn:hover {
  background: #ff6b81;
  color: white;
  transform: rotate(90deg);
}

/* Responsive */
@media (max-width: 768px) {
  .header-info h2 {
    font-size: 1.5rem;
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
}
</style>
