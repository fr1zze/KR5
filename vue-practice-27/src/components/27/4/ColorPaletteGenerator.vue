<template>
  <div class="palette-container">
    <h2>Пример 4: Генератор цветовых палитр</h2>

    <!-- Управление -->
    <div class="controls">
      <button @click="generatePalette" class="button primary">
        🎲 Случайная палитра
      </button>

      <select v-model="colorCount" class="select">
        <option :value="3">3 цвета</option>
        <option :value="5">5 цветов</option>
        <option :value="7">7 цветов</option>
      </select>

      <select v-model="format" class="select">
        <option value="hex">HEX</option>
        <option value="rgb">RGB</option>
      </select>
    </div>

    <!-- Палитра -->
    <div class="palette">
      <div
        v-for="(color, index) in palette"
        :key="index"
        class="color-card"
        :style="{ backgroundColor: color.hex }"
        @click="copyColor(color[format])"
      >
        <div class="color-info">
          <p>{{ color[format] }}</p>

          <button class="lock-button" @click.stop="toggleLock(index)">
            {{ color.locked ? '🔒' : '🔓' }}
          </button>
        </div>
      </div>
    </div>

    <!-- Уведомление -->
    <div v-if="copied" class="toast">Скопировано: {{ copied }}</div>

    <!-- Preview -->
    <div class="preview">
      <h3>Превью интерфейса</h3>

      <div class="preview-controls">
        <button @click="darkMode = !darkMode" class="button small">
          {{ darkMode ? 'Светлый фон' : 'Тёмный фон' }}
        </button>
      </div>

      <div class="preview-box" :class="{ dark: darkMode }">
        <button class="demo-btn" :style="{ backgroundColor: palette[0]?.hex }">
          Кнопка
        </button>

        <div class="demo-card" :style="{ backgroundColor: palette[1]?.hex }">
          Карточка
        </div>

        <h4 class="demo-title" :style="{ color: palette[2]?.hex }">
          Заголовок
        </h4>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch, onMounted } from 'vue'

export default {
  name: 'ColorPaletteGenerator',

  setup () {
    const colorCount = ref(5)
    const format = ref('hex')
    const darkMode = ref(false)
    const copied = ref('')

    // ВАЖНО: массив должен быть ref()
    const palette = ref([])

    const randomColor = () => {
      const r = Math.floor(Math.random() * 255)
      const g = Math.floor(Math.random() * 255)
      const b = Math.floor(Math.random() * 255)

      return {
        hex: `#${r.toString(16).padStart(2, '0')}${g
          .toString(16)
          .padStart(2, '0')}${b.toString(16).padStart(2, '0')}`.toUpperCase(),
        rgb: `rgb(${r}, ${g}, ${b})`,
        locked: false
      }
    }

    const generatePalette = () => {
      for (let i = 0; i < colorCount.value; i++) {
        if (!palette.value[i] || !palette.value[i].locked) {
          palette.value[i] = randomColor()
        }
      }
      palette.value.length = colorCount.value
      savePalette()
    }

    const toggleLock = index => {
      palette.value[index].locked = !palette.value[index].locked
      savePalette()
    }

    const copyColor = async value => {
      await navigator.clipboard.writeText(value)
      copied.value = value
      setTimeout(() => (copied.value = ''), 1200)
    }

    const savePalette = () => {
      localStorage.setItem('savedPalette', JSON.stringify(palette.value))
    }

    const loadPalette = () => {
      const saved = localStorage.getItem('savedPalette')
      if (saved) {
        palette.value = JSON.parse(saved)
      } else {
        generatePalette()
      }
    }

    watch(colorCount, generatePalette)

    onMounted(loadPalette)

    return {
      palette,
      colorCount,
      format,
      darkMode,
      copied,
      generatePalette,
      copyColor,
      toggleLock
    }
  }
}
</script>

<style scoped>
.palette-container {
  max-width: 900px;
  margin: 20px auto;
  padding: 20px;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.palette {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
}

.color-card {
  flex: 1;
  height: 120px;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: end;
  padding: 10px;
  color: white;
  font-weight: bold;
  position: relative;
}

.color-info {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.lock-button {
  background: rgba(0, 0, 0, 0.4);
  border: none;
  font-size: 20px;
  cursor: pointer;
  border-radius: 5px;
  padding: 5px;
  color: white;
}

.toast {
  background-color: #28a745;
  color: white;
  padding: 10px 15px;
  border-radius: 6px;
  display: inline-block;
  margin-bottom: 20px;
  animation: fade 1.2s ease-out forwards;
}

@keyframes fade {
  0% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

.preview {
  margin-top: 20px;
}

.preview-controls {
  margin-bottom: 10px;
}

.preview-box {
  padding: 20px;
  border-radius: 10px;
  background: #f2f2f2;
}

.preview-box.dark {
  background: #222;
}

.demo-btn {
  padding: 10px 20px;
  border-radius: 5px;
  color: white;
  border: none;
  margin-bottom: 10px;
}

.demo-card {
  padding: 20px;
  border-radius: 6px;
  margin-bottom: 10px;
  color: white;
}

.demo-title {
  font-size: 20px;
}
</style>
