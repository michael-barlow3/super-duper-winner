<template>
  <header class="header">
    <div class="header-content">
      <div class="logo">
        <h1>📚 Content Browser</h1>
      </div>
      
      <div class="search-container">
        <div class="search-wrapper">
          <input 
            v-model="localSearchQuery"
            type="text"
            placeholder="Search content..."
            class="search-input"
            @input="emitSearch"
          />
          <span class="search-icon">🔍</span>
          <button 
            v-if="localSearchQuery"
            class="clear-btn"
            @click="clearSearch"
          >
            ✕
          </button>
        </div>
      </div>

      <div class="header-info">
        <span class="info-text">Infinite Scroll</span>
      </div>
    </div>
  </header>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

interface Props {
  searchQuery?: string
}

interface Emits {
  (e: 'search', query: string): void
}

defineProps<Props>()
const emit = defineEmits<Emits>()

const localSearchQuery = ref('')

const emitSearch = () => {
  emit('search', localSearchQuery.value)
}

const clearSearch = () => {
  localSearchQuery.value = ''
  emit('search', '')
}

watch(() => localSearchQuery, () => {
  emitSearch()
}, { deep: true })
</script>

<style scoped>
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 80px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  display: flex;
  align-items: center;
}

.header-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 20px;
  gap: 30px;
}

.logo h1 {
  color: white;
  font-size: 24px;
  font-weight: 600;
  white-space: nowrap;
  margin: 0;
}

.search-container {
  flex: 1;
  max-width: 400px;
}

.search-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.search-input {
  width: 100%;
  padding: 10px 40px 10px 16px;
  border: none;
  border-radius: 24px;
  font-size: 14px;
  background-color: rgba(255, 255, 255, 0.95);
  transition: background-color 0.3s;
}

.search-input:focus {
  outline: none;
  background-color: white;
}

.search-input::placeholder {
  color: #999;
}

.search-icon {
  position: absolute;
  right: 12px;
  color: #999;
  font-size: 18px;
  pointer-events: none;
}

.clear-btn {
  position: absolute;
  right: 35px;
  background: none;
  border: none;
  font-size: 16px;
  cursor: pointer;
  color: #999;
  padding: 0;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.2s;
}

.clear-btn:hover {
  color: #333;
}

.header-info {
  color: white;
  font-size: 14px;
  white-space: nowrap;
}

.info-text {
  background-color: rgba(255, 255, 255, 0.2);
  padding: 4px 12px;
  border-radius: 12px;
}

/* Responsive design */
@media (max-width: 768px) {
  .header-content {
    flex-wrap: wrap;
    gap: 15px;
  }

  .logo h1 {
    font-size: 18px;
    flex: 0 1 100%;
  }

  .search-container {
    flex: 0 1 100%;
    max-width: none;
  }

  .header-info {
    flex: 0 1 100%;
  }
}
</style>
