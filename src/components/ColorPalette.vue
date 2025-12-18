<template>
  <div class="color-palette">
    <div class="controls">
      <button @click="generatePalette" class="btn generate-btn">
        🎲 Сгенерировать новую палитру
      </button>
      
      <div class="settings">
        <div class="setting-group">
          <label>Количество цветов:</label>
          <select v-model="colorCount" class="select-input">
            <option value="3">3</option>
            <option value="5" selected>5</option>
            <option value="7">7</option>
          </select>
        </div>
        
        <div class="setting-group">
          <label>Формат:</label>
          <select v-model="colorFormat" class="select-input">
            <option value="hex">HEX</option>
            <option value="rgb">RGB</option>
          </select>
        </div>
        
        <div class="setting-group">
          <label>
            <input type="checkbox" v-model="autoSave">
            Автосохранение
          </label>
        </div>
      </div>
    </div>

    <div v-if="colors.length === 0" class="empty-state">
      <p>Нажмите кнопку "Сгенерировать новую палитру" чтобы создать первую палитру!</p>
    </div>

    <div v-else class="palette-display">
      <div class="colors-grid">
        <ColorCard
          v-for="(color, index) in colors"
          :key="index"
          :color="color"
          :format="colorFormat"
          :pinned="pinnedColors[index]"
          @copy="copyToClipboard"
          @pin="togglePin(index)"
          @lock="toggleLock(index)"
        />
      </div>

      <div class="palette-actions">
        <button @click="savePalette" class="btn save-btn">
          💾 Сохранить палитру
        </button>
        <button @click="clearPalette" class="btn clear-btn">
          🗑️ Очистить
        </button>
        <button @click="copyAllColors" class="btn copy-all-btn">
          📋 Копировать все цвета
        </button>
      </div>

      <div class="palette-info">
        <div class="info-card">
          <h3>📊 Информация о палитре</h3>
          <p>Количество цветов: {{ colors.length }}</p>
          <p>Закрепленных: {{ pinnedCount }}</p>
          <p>Формат отображения: {{ colorFormat.toUpperCase() }}</p>
        </div>
        
        <div class="preview-card">
          <h3>👁️ Превью</h3>
          <div class="preview-ui">
            <div class="preview-header" :style="{ backgroundColor: colors[0] || '#667eea' }">
              <h4>Заголовок</h4>
            </div>
            <div class="preview-body">
              <button class="preview-button" :style="{ backgroundColor: colors[1] || '#764ba2', color: '#fff' }">
                Кнопка
              </button>
              <div class="preview-card" :style="{ borderColor: colors[2] || '#4ecdc4' }">
                <p>Карточка с текстом</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <div v-if="notification.show" class="notification" :class="notification.type">
        {{ notification.message }}
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import ColorCard from './ColorCard.vue'

export default {
  name: 'ColorPalette',
  components: {
    ColorCard
  },
  
  setup() {
    // Реактивные данные
    const colors = ref([])
    const pinnedColors = ref([])
    const colorCount = ref(5)
    const colorFormat = ref('hex')
    const autoSave = ref(true)
    
    const notification = ref({
      show: false,
      message: '',
      type: 'success'
    })
    
    // Вычисляемые свойства
    const pinnedCount = computed(() => {
      return pinnedColors.value.filter(Boolean).length
    })
    
    // Методы
    const generateRandomColor = () => {
      const letters = '0123456789ABCDEF'
      let color = '#'
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)]
      }
      return color
    }
    
    const generatePalette = () => {
      const newColors = []
      const newPinned = []
      
      for (let i = 0; i < colorCount.value; i++) {
        // Если цвет закреплен, оставляем его
        if (pinnedColors.value[i] && colors.value[i]) {
          newColors.push(colors.value[i])
          newPinned.push(true)
        } else {
          newColors.push(generateRandomColor())
          newPinned.push(false)
        }
      }
      
      colors.value = newColors
      pinnedColors.value = newPinned
      
      if (autoSave.value) {
        saveToLocalStorage()
      }
      
      showNotification('🎨 Палитра сгенерирована!')
    }
    
    const copyToClipboard = (color) => {
      let textToCopy = color
      
      if (colorFormat.value === 'rgb') {
        // Конвертируем HEX в RGB
        const hex = color.replace('#', '')
        const r = parseInt(hex.substring(0, 2), 16)
        const g = parseInt(hex.substring(2, 4), 16)
        const b = parseInt(hex.substring(4, 6), 16)
        textToCopy = `rgb(${r}, ${g}, ${b})`
      }
      
      navigator.clipboard.writeText(textToCopy)
        .then(() => {
          showNotification(`Скопировано: ${textToCopy}`, 'success')
        })
        .catch(err => {
          console.error('Ошибка копирования:', err)
          showNotification('Ошибка копирования', 'error')
        })
    }
    
    const copyAllColors = () => {
      const colorsText = colors.value
        .map(color => {
          if (colorFormat.value === 'rgb') {
            const hex = color.replace('#', '')
            const r = parseInt(hex.substring(0, 2), 16)
            const g = parseInt(hex.substring(2, 4), 16)
            const b = parseInt(hex.substring(4, 6), 16)
            return `rgb(${r}, ${g}, ${b})`
          }
          return color
        })
        .join('\n')
      
      navigator.clipboard.writeText(colorsText)
        .then(() => {
          showNotification('Все цвета скопированы!', 'success')
        })
        .catch(err => {
          console.error('Ошибка копирования:', err)
          showNotification('Ошибка копирования', 'error')
        })
    }
    
    const togglePin = (index) => {
      pinnedColors.value[index] = !pinnedColors.value[index]
      if (autoSave.value) {
        saveToLocalStorage()
      }
      showNotification(
        pinnedColors.value[index] 
          ? '📍 Цвет закреплен' 
          : '📌 Цвет откреплен',
        'info'
      )
    }
    
    const toggleLock = (index) => {
      // В упрощенной версии lock = pin
      togglePin(index)
    }
    
    const savePalette = () => {
      saveToLocalStorage()
      showNotification('💾 Палитра сохранена в localStorage', 'success')
    }
    
    const clearPalette = () => {
      if (confirm('Очистить текущую палитру?')) {
        colors.value = []
        pinnedColors.value = []
        localStorage.removeItem('colorPalette')
        showNotification('Палитра очищена', 'info')
      }
    }
    
    const saveToLocalStorage = () => {
      const paletteData = {
        colors: colors.value,
        pinned: pinnedColors.value,
        count: colorCount.value,
        format: colorFormat.value
      }
      localStorage.setItem('colorPalette', JSON.stringify(paletteData))
    }
    
    const loadFromLocalStorage = () => {
      const savedData = localStorage.getItem('colorPalette')
      if (savedData) {
        try {
          const data = JSON.parse(savedData)
          colors.value = data.colors || []
          pinnedColors.value = data.pinned || []
          colorCount.value = data.count || 5
          colorFormat.value = data.format || 'hex'
        } catch (e) {
          console.error('Ошибка загрузки данных:', e)
        }
      }
    }
    
    const showNotification = (message, type = 'success') => {
      notification.value = {
        show: true,
        message,
        type
      }
      
      setTimeout(() => {
        notification.value.show = false
      }, 3000)
    }
    
    // Watchers
    watch([colorCount, colorFormat], () => {
      if (autoSave.value) {
        saveToLocalStorage()
      }
    })
    
    // Lifecycle hooks
    onMounted(() => {
      loadFromLocalStorage()
      
      // Если нет сохраненной палитры, генерируем первую
      if (colors.value.length === 0) {
        generatePalette()
      }
    })
    
    return {
      colors,
      pinnedColors,
      colorCount,
      colorFormat,
      autoSave,
      notification,
      pinnedCount,
      generatePalette,
      copyToClipboard,
      copyAllColors,
      togglePin,
      toggleLock,
      savePalette,
      clearPalette
    }
  }
}
</script>

<style scoped>
.color-palette {
  max-width: 1200px;
  margin: 0 auto;
}

.controls {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.settings {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.setting-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.setting-group label {
  font-weight: 600;
  color: #555;
}

.select-input {
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 1rem;
}

.btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
}

.generate-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  font-size: 1.1rem;
  padding: 1rem 2rem;
  align-self: flex-start;
}

.generate-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(102, 126, 234, 0.4);
}

.save-btn {
  background-color: #28a745;
  color: white;
}

.clear-btn {
  background-color: #dc3545;
  color: white;
}

.copy-all-btn {
  background-color: #17a2b8;
  color: white;
}

.btn:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

.empty-state {
  text-align: center;
  padding: 4rem 2rem;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  color: #666;
}

.empty-state p {
  font-size: 1.2rem;
}

.colors-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.palette-actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  margin-bottom: 2rem;
  padding: 1rem;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.palette-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.info-card, .preview-card {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.info-card h3, .preview-card h3 {
  margin-bottom: 1rem;
  color: #333;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.preview-ui {
  border: 1px solid #eee;
  border-radius: 8px;
  overflow: hidden;
}

.preview-header {
  padding: 1rem;
  color: white;
}

.preview-body {
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.preview-button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
}

.preview-card {
  padding: 1rem;
  border: 2px solid;
  border-radius: 6px;
}

.notification {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  color: white;
  font-weight: 600;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  animation: slideIn 0.3s ease;
  z-index: 1000;
}

.notification.success {
  background-color: #28a745;
}

.notification.error {
  background-color: #dc3545;
}

.notification.info {
  background-color: #17a2b8;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .settings {
    grid-template-columns: 1fr;
  }
  
  .colors-grid {
    grid-template-columns: 1fr;
  }
  
  .palette-info {
    grid-template-columns: 1fr;
  }
  
  .notification {
    left: 1rem;
    right: 1rem;
    bottom: 1rem;
  }
}
</style>