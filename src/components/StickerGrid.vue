<template>
  <div class="sticker-grid-container">
    <div v-if="filteredStickers.length === 0" class="no-stickers">
      <span class="empty-icon">📭</span>
      <p>No hay figuritas que coincidan con los filtros.</p>
    </div>
    <TransitionGroup name="sticker-list" tag="div" class="grid">
      <div 
        v-for="sticker in filteredStickers" 
        :key="sticker.figura"
        :class="[
          'sticker-wrapper',
          { obtenida: sticker.tiene },
          { 'special-00': sticker.figura === '00' }
        ]"
        @click="handleClick(sticker)"
        @contextmenu.prevent="resetSticker(sticker)"
        @touchstart="startPress(sticker)"
        @touchend="endPress"
        @touchmove="endPress"
      >
        <div class="sticker">
          <div class="sticker-face">
            <span class="sticker-code">{{ sticker.figura }}</span>
            <div v-if="sticker.tiene" class="check-badge">✓</div>
            <div v-if="sticker.repetidas > 0" :key="'badge-' + sticker.repetidas" class="repeat-badge">{{ sticker.repetidas }}</div>
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
  },
  readOnly: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['update-sticker']);

const filteredStickers = computed(() => {
  let result = props.country.figuritas;

  if (props.filterType === 'obtained') {
    result = result.filter(f => f.tiene);
  } else if (props.filterType === 'missing') {
    result = result.filter(f => !f.tiene);
  } else if (props.filterType === 'repeated') {
    result = result.filter(f => f.repetidas > 0);
  }

  const query = props.searchQuery.toLowerCase().trim();
  if (query) {
    const countryMatch = props.country.pais.toLowerCase().includes(query) || 
                         props.country.codigo.toLowerCase().includes(query);
    
    if (!countryMatch) {
      result = result.filter(f => f.figura.toLowerCase().includes(query));
    }
  }

  return result;
});

const handleClick = (sticker) => {
  if (props.readOnly) return;
  if (!sticker.tiene) {
    emit('update-sticker', props.country.codigo, sticker.figura, true, 0);
  } else {
    emit('update-sticker', props.country.codigo, sticker.figura, true, sticker.repetidas + 1);
  }
};

const resetSticker = (sticker) => {
  if (props.readOnly) return;
  if (sticker.tiene) {
    if (confirm('¿Deseas marcar esta figurita como no obtenida?')) {
      emit('update-sticker', props.country.codigo, sticker.figura, false, 0);
    }
  }
};

let pressTimer = null;
const startPress = (sticker) => {
  if (props.readOnly) return;
  pressTimer = setTimeout(() => {
    resetSticker(sticker);
  }, 600); // 600ms para long press
};

const endPress = () => {
  if (pressTimer) clearTimeout(pressTimer);
};
</script>

<style scoped>
.sticker-grid-container {
  padding: 1.5rem;
  overflow-y: auto;
  flex: 1;
}

.no-stickers {
  text-align: center;
  padding: 3rem;
  color: var(--text-muted);
}

.empty-icon {
  font-size: 2.5rem;
  display: block;
  margin-bottom: 0.75rem;
}

.no-stickers p {
  font-weight: 600;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
  gap: 0.75rem;
}

/* Sticker list transitions */
.sticker-list-enter-active {
  transition: all 0.3s var(--ease-smooth);
}
.sticker-list-leave-active {
  transition: all 0.2s var(--ease-smooth);
}
.sticker-list-enter-from,
.sticker-list-leave-to {
  opacity: 0;
  transform: scale(0.8);
}

/* Sticker Wrapper */
.sticker-wrapper {
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
}

.sticker {
  position: relative;
  width: 100%;
  aspect-ratio: 0.78;
  transition: transform 0.3s var(--ease-spring);
}

.sticker-wrapper:hover .sticker {
  transform: scale(1.08) translateY(-4px);
}

.sticker-wrapper:active .sticker {
  transform: scale(0.95);
  transition: transform 0.1s;
}

/* Sticker Face */
.sticker-face {
  position: absolute;
  inset: 0;
  background: var(--bg-card);
  border: 2px solid var(--border-color);
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: all 0.3s var(--ease-smooth);
  overflow: hidden;
}

.sticker-code {
  font-weight: 800;
  font-size: 0.95rem;
  color: var(--text-muted);
  text-align: center;
  transition: all 0.3s;
  line-height: 1.2;
  padding: 0.25rem;
}

.check-badge {
  position: absolute;
  top: -2px;
  right: -2px;
  background: var(--wc-mint);
  color: var(--bg-color);
  width: 20px;
  height: 20px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: 900;
  font-size: 0.65rem;
  animation: popIn 0.3s var(--ease-spring);
}

.repeat-badge {
  position: absolute;
  top: -6px;
  left: -6px;
  background: var(--wc-red);
  color: white;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: 900;
  font-size: 0.8rem;
  box-shadow: 0 2px 5px rgba(0,0,0,0.3);
  animation: popBounce 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

@keyframes popIn {
  0% { transform: scale(0); }
  100% { transform: scale(1); }
}

@keyframes popBounce {
  0% { transform: scale(0.5); opacity: 0; }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); opacity: 1; }
}

/* Obtained State */
.sticker-wrapper.obtenida .sticker-face {
  background: linear-gradient(145deg, var(--wc-green-deep) 0%, var(--wc-green-forest) 100%);
  border-color: var(--wc-mint);
  box-shadow: 0 0 12px rgba(86, 224, 192, 0.2);
}

.sticker-wrapper.obtenida .sticker-code {
  color: var(--wc-mint);
  text-shadow: 0 0 8px rgba(86, 224, 192, 0.3);
}

/* Special Sticker "00" — Panini Logo */
.sticker-wrapper.special-00 .sticker-face {
  background: linear-gradient(145deg, #1a1a2e, #16213e);
  border-color: var(--wc-gold);
  box-shadow: 0 0 12px rgba(212, 175, 55, 0.25);
}

.sticker-wrapper.special-00 .sticker-code {
  color: var(--wc-gold);
  font-size: 1.2rem;
  font-weight: 900;
}

.sticker-wrapper.special-00.obtenida .sticker-face {
  background: linear-gradient(145deg, var(--wc-gold), #f1c40f);
  border-color: var(--wc-yellow-warm);
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.4);
}

.sticker-wrapper.special-00.obtenida .sticker-code {
  color: var(--bg-color);
  text-shadow: none;
}

/* Responsive */
@media (max-width: 480px) {
  .grid {
    grid-template-columns: repeat(auto-fill, minmax(65px, 1fr));
    gap: 0.5rem;
  }
  .sticker-code {
    font-size: 0.8rem;
  }
}
</style>
