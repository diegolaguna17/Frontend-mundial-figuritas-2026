<template>
  <div class="country-card">
    <div class="card-header">
      <h3>{{ country.pais }}</h3>
      <span class="code-badge">{{ country.codigo }}</span>
    </div>
    
    <div class="card-content">
      <div class="progress-info">
        <div class="progress-stats">
          <span class="label">Completado</span>
          <span class="count"><span class="highlight">{{ obtainedCount }}</span> / {{ country.figuritas.length }}</span>
        </div>
        <div class="progress-bar">
          <div class="fill" :style="{ width: progressPercentage + '%' }"></div>
        </div>
      </div>
    </div>
    
    <div class="card-footer">
      <span>Abrir Equipo</span>
      <span class="arrow">→</span>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  country: {
    type: Object,
    required: true
  },
  filterType: {
    type: String,
    default: 'all'
  }
});

const obtainedCount = computed(() => {
  return props.country.figuritas.filter(f => f.obtenida).length;
});

const progressPercentage = computed(() => {
  if (props.country.figuritas.length === 0) return 0;
  return (obtainedCount.value / props.country.figuritas.length) * 100;
});
</script>

<style scoped>
.country-card {
  background: var(--card-bg);
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  display: flex;
  flex-direction: column;
  border: 1px solid #f1f2f6;
  position: relative;
}

.country-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background: linear-gradient(90deg, var(--pitch-green), var(--gold));
  opacity: 0;
  transition: opacity 0.3s;
}

.country-card:hover {
  transform: translateY(-8px) scale(1.02);
  box-shadow: var(--shadow-md);
}

.country-card:hover::before {
  opacity: 1;
}

.card-header {
  padding: 1.5rem 1.5rem 0.5rem;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

h3 {
  margin: 0;
  color: var(--text-main);
  font-size: 1.3rem;
  font-weight: 800;
}

.code-badge {
  background: #f1f2f6;
  color: var(--text-muted);
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 1px;
}

.card-content {
  padding: 1rem 1.5rem;
  flex: 1;
}

.progress-info {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.progress-stats {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

.progress-stats .label {
  font-size: 0.85rem;
  color: var(--text-muted);
  font-weight: 600;
  text-transform: uppercase;
}

.count {
  font-size: 0.95rem;
  color: var(--text-muted);
  font-weight: 600;
}

.count .highlight {
  color: var(--pitch-green);
  font-weight: 800;
  font-size: 1.1rem;
}

.progress-bar {
  height: 8px;
  background: #f1f2f6;
  border-radius: 4px;
  overflow: hidden;
}

.fill {
  height: 100%;
  background: var(--pitch-green);
  border-radius: 4px;
  transition: width 0.5s cubic-bezier(0.25, 0.8, 0.25, 1);
}

.card-footer {
  padding: 1rem 1.5rem;
  background: rgba(10, 92, 54, 0.03);
  border-top: 1px solid #f1f2f6;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: var(--pitch-green);
  font-weight: 700;
  font-size: 0.9rem;
  text-transform: uppercase;
}

.arrow {
  transition: transform 0.3s;
  font-size: 1.2rem;
}

.country-card:hover .arrow {
  transform: translateX(5px);
  color: var(--gold);
}
</style>
