<template>
  <div class="sticker-grid-container">
    <div v-if="filteredStickers.length === 0" class="no-stickers">
      <p>No hay figuritas que coincidan con los filtros.</p>
    </div>
    <TransitionGroup name="list" tag="div" class="grid">
      <div 
        v-for="sticker in filteredStickers" 
        :key="sticker.figura"
        :class="['sticker-wrapper', { obtenida: sticker.obtenida, 'special-00': sticker.figura === '00' }]"
        @click="toggleSticker(sticker)"
      >
        <div class="sticker">
          <div class="sticker-inner">
            <div class="sticker-content">
              <span class="code">{{ sticker.figura }}</span>
            </div>
            <div class="glow-effect"></div>
            <div class="status-badge" v-if="sticker.obtenida">✓</div>
          </div>
        </div>
      </div>
    </TransitionGroup>
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
  },
  searchQuery: {
    type: String,
    default: ''
  }
});

const emit = defineEmits(['update-sticker']);

const filteredStickers = computed(() => {
  let result = props.country.figuritas;

  if (props.filterType === 'obtained') {
    result = result.filter(f => f.obtenida);
  } else if (props.filterType === 'missing') {
    result = result.filter(f => !f.obtenida);
  }

  const query = props.searchQuery.toLowerCase().trim();
  if (query) {
    // If the country name or code matches the query, we don't necessarily filter out stickers.
    // Let's check if we should filter stickers based on query:
    const countryMatch = props.country.pais.toLowerCase().includes(query) || 
                         props.country.codigo.toLowerCase().includes(query);
    
    // If the country itself matches, show all its stickers (that pass the filterType).
    // Otherwise, filter stickers by their code.
    if (!countryMatch) {
      result = result.filter(f => f.figura.toLowerCase().includes(query));
    }
  }

  return result;
});

const toggleSticker = (sticker) => {
  emit('update-sticker', props.country.codigo, sticker.figura, !sticker.obtenida);
};
</script>

<style scoped>
.sticker-grid-container {
  padding: 2rem;
  overflow-y: auto;
  background: var(--bg-color);
  border-bottom-left-radius: var(--radius-lg);
  border-bottom-right-radius: var(--radius-lg);
  flex: 1;
}

.no-stickers {
  text-align: center;
  padding: 3rem;
  color: var(--text-muted);
  font-weight: 600;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(90px, 1fr));
  gap: 1.2rem;
}

/* Animations for list */
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: scale(0.8);
}

.sticker-wrapper {
  perspective: 1000px;
  cursor: pointer;
}

.sticker {
  position: relative;
  width: 100%;
  aspect-ratio: 0.75; /* Classic sticker aspect ratio */
  transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  transform-style: preserve-3d;
}

.sticker-wrapper:hover .sticker {
  transform: scale(1.08) translateY(-5px);
}

.sticker-inner {
  position: absolute;
  width: 100%;
  height: 100%;
  background: white;
  border: 3px solid #e0e0e0;
  border-radius: 8px;
  box-shadow: inset 0 0 10px rgba(0,0,0,0.05), 0 4px 6px rgba(0,0,0,0.1);
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  transition: all 0.3s ease;
}

/* Base style (Missing) */
.sticker-content {
  position: relative;
  z-index: 2;
}

.code {
  font-weight: 800;
  font-size: 1.1rem;
  color: #bdc3c7;
  text-align: center;
  transition: color 0.3s;
}

.glow-effect {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.8) 0%, rgba(255,255,255,0) 70%);
  opacity: 0;
  transform: rotate(45deg);
  transition: opacity 0.5s;
  pointer-events: none;
  z-index: 1;
}

.status-badge {
  position: absolute;
  top: -5px;
  right: -5px;
  background: var(--gold);
  color: white;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  font-size: 0.9rem;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  z-index: 3;
  animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

@keyframes popIn {
  0% { transform: scale(0); }
  100% { transform: scale(1); }
}

/* Obtained State */
.sticker-wrapper.obtenida .sticker-inner {
  border-color: var(--gold);
  background: linear-gradient(135deg, var(--pitch-green) 0%, var(--pitch-green-light) 100%);
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3), inset 0 0 20px rgba(0,0,0,0.2);
}

.sticker-wrapper.obtenida .code {
  color: white;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
}

.sticker-wrapper.obtenida .glow-effect {
  opacity: 0.15;
}

/* Click Animation Effect */
.sticker-wrapper:active .sticker {
  transform: scale(0.95);
  transition: transform 0.1s;
}

/* Special Sticker "00" */
.sticker-wrapper.special-00 .sticker-inner {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border: 3px solid var(--gold);
  box-shadow: 0 4px 15px rgba(212, 175, 55, 0.4);
}

.sticker-wrapper.special-00 .code {
  color: var(--pitch-green);
  font-size: 1.4rem;
  font-weight: 900;
  text-shadow: 1px 1px 0px rgba(255,255,255,0.8);
}

.sticker-wrapper.special-00 .glow-effect {
  opacity: 0.3;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.5) 0%, rgba(212, 175, 55, 0) 70%);
}

.sticker-wrapper.special-00.obtenida .sticker-inner {
  background: linear-gradient(135deg, var(--gold) 0%, #f1c40f 100%);
  border-color: var(--pitch-green);
  box-shadow: 0 8px 25px rgba(212, 175, 55, 0.6), inset 0 0 20px rgba(255,255,255,0.5);
}

.sticker-wrapper.special-00.obtenida .code {
  color: var(--pitch-green);
  text-shadow: none;
}
</style>
