<template>
  <div class="country-card" :class="{ 'card-complete': progressPercentage === 100 }">
    <div class="card-accent"></div>
    
    <div class="card-body">
      <div class="card-header">
        <h3>{{ country.pais }}</h3>
        <span class="code-badge">{{ country.codigo }}</span>
      </div>
      
      <div class="card-progress">
        <div class="progress-stats">
          <span class="count">
            <span class="obtained">{{ obtainedCount }}</span>
            <span class="separator">/</span>
            <span class="total">{{ country.figuritas.length }}</span>
          </span>
          <span class="percent">{{ Math.round(progressPercentage) }}%</span>
        </div>
        <div class="progress-bar">
          <div class="fill" :style="{ width: progressPercentage + '%' }"></div>
        </div>
      </div>
    </div>
    
    <div class="card-footer">
      <span class="footer-text">Abrir</span>
      <span class="footer-arrow">→</span>
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
  return props.country.figuritas.filter(f => f.tiene).length;
});

const progressPercentage = computed(() => {
  if (props.country.figuritas.length === 0) return 0;
  return (obtainedCount.value / props.country.figuritas.length) * 100;
});
</script>

<style scoped>
.country-card {
  background: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-lg);
  overflow: hidden;
  cursor: pointer;
  transition: all 0.35s var(--ease-smooth);
  display: flex;
  flex-direction: column;
  position: relative;
}

.card-accent {
  height: 3px;
  background: var(--gradient-primary);
  opacity: 0;
  transition: opacity 0.3s;
}

.country-card:hover {
  transform: translateY(-6px);
  box-shadow: var(--shadow-md);
  border-color: rgba(86, 224, 192, 0.2);
}

.country-card:hover .card-accent {
  opacity: 1;
}

.country-card:active {
  transform: translateY(-2px) scale(0.99);
}

/* Completed state */
.card-complete {
  border-color: rgba(86, 224, 192, 0.3);
}

.card-complete .card-accent {
  opacity: 1;
  background: var(--gradient-warm);
}

.card-body {
  padding: 1.25rem 1.5rem;
  flex: 1;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
}

h3 {
  margin: 0;
  color: var(--text-primary);
  font-size: 1.15rem;
  font-weight: 700;
}

.code-badge {
  background: var(--bg-card);
  color: var(--text-muted);
  padding: 0.15rem 0.5rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 1px;
  border: 1px solid var(--border-color);
}

/* Progress */
.card-progress {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.progress-stats {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.count {
  font-size: 0.9rem;
  color: var(--text-secondary);
  font-weight: 600;
}

.obtained {
  color: var(--wc-mint);
  font-weight: 800;
  font-size: 1rem;
}

.separator {
  margin: 0 0.15rem;
  color: var(--text-muted);
}

.total {
  color: var(--text-muted);
}

.percent {
  font-size: 0.8rem;
  font-weight: 700;
  color: var(--text-muted);
}

.progress-bar {
  height: 6px;
  background: var(--bg-card);
  border-radius: 3px;
  overflow: hidden;
}

.fill {
  height: 100%;
  background: var(--wc-mint);
  border-radius: 3px;
  transition: width 0.6s var(--ease-smooth);
}

/* Footer */
.card-footer {
  padding: 0.75rem 1.5rem;
  border-top: 1px solid var(--border-color);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.footer-text {
  font-size: 0.8rem;
  font-weight: 700;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 1px;
}

.footer-arrow {
  color: var(--text-muted);
  transition: all 0.3s;
  font-size: 1rem;
}

.country-card:hover .footer-text {
  color: var(--wc-mint);
}

.country-card:hover .footer-arrow {
  color: var(--wc-mint);
  transform: translateX(4px);
}
</style>
