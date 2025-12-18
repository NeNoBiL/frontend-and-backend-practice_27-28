<template>
  <div class="color-card" :style="{ backgroundColor: color }">
    <div class="color-info">
      <div class="color-value" :class="{ 'light-text': isLightColor }">
        {{ formattedColor }}
      </div>
      
      <div class="color-actions">
        <button 
          @click="$emit('copy', color)" 
          class="action-btn copy-btn"
          :class="{ 'light-text': isLightColor }"
          title="Копировать"
        >
          📋
        </button>
        
        <button 
          @click="$emit('pin', index)" 
          class="action-btn pin-btn"
          :class="{ 'light-text': isLightColor, pinned: pinned }"
          :title="pinned ? 'Открепить' : 'Закрепить'"
        >
          {{ pinned ? '📍' : '📌' }}
        </button>
        
        <button 
          @click="$emit('lock', index)" 
          class="action-btn lock-btn"
          :class="{ 'light-text': isLightColor, locked: locked }"
          :title="locked ? 'Разблокировать' : 'Заблокировать'"
        >
          {{ locked ? '🔒' : '🔓' }}
        </button>
      </div>
    </div>
    
    <div class="color-overlay" :style="{ backgroundColor: color }"></div>
  </div>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'ColorCard',
  
  props: {
    color: {
      type: String,
      required: true,
      default: '#000000'
    },
    format: {
      type: String,
      default: 'hex'
    },
    pinned: {
      type: Boolean,
      default: false
    },
    locked: {
      type: Boolean,
      default: false
    },
    index: {
      type: Number,
      default: 0
    }
  },
  
  emits: ['copy', 'pin', 'lock'],
  
  setup(props) {
    // Вычисляемые свойства
    const formattedColor = computed(() => {
      if (props.format === 'rgb') {
        const hex = props.color.replace('#', '')
        const r = parseInt(hex.substring(0, 2), 16)
        const g = parseInt(hex.substring(2, 4), 16)
        const b = parseInt(hex.substring(4, 6), 16)
        return `rgb(${r}, ${g}, ${b})`
      }
      return props.color.toUpperCase()
    })
    
    const isLightColor = computed(() => {
      const hex = props.color.replace('#', '')
      const r = parseInt(hex.substring(0, 2), 16)
      const g = parseInt(hex.substring(2, 4), 16)
      const b = parseInt(hex.substring(4, 6), 16)
      
      // Формула восприятия яркости
      const brightness = (r * 299 + g * 587 + b * 114) / 1000
      return brightness > 155
    })
    
    return {
      formattedColor,
      isLightColor
    }
  }
}
</script>

<style scoped>
.color-card {
  position: relative;
  height: 180px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
  cursor: pointer;
}

.color-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.color-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  opacity: 0.9;
}

.color-info {
  position: relative;
  z-index: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 1rem;
}

.color-value {
  font-family: 'Courier New', monospace;
  font-weight: 600;
  font-size: 1rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
  user-select: all;
}

.light-text {
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.7);
}

.color-actions {
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
}

.action-btn {
  background: rgba(255, 255, 255, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 6px;
  width: 36px;
  height: 36px;
  font-size: 1.2rem;
  cursor: pointer;
  transition: all 0.2s ease;
  backdrop-filter: blur(4px);
}

.action-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  border-color: rgba(255, 255, 255, 0.5);
  transform: scale(1.1);
}

.action-btn.pinned,
.action-btn.locked {
  background: rgba(255, 255, 255, 0.4);
  border-color: rgba(255, 255, 255, 0.7);
}

.action-btn.copy-btn:hover {
  background: rgba(76, 175, 80, 0.3);
}

.action-btn.pin-btn:hover {
  background: rgba(255, 193, 7, 0.3);
}

.action-btn.lock-btn:hover {
  background: rgba(33, 150, 243, 0.3);
}
</style>