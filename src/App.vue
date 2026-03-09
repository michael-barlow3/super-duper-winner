<template>
  <div class="app-container">
    <Header :search-query="searchQuery" @search="handleSearch" />
    
    <div class="panels-container">
      <!-- Navigation Panel -->
      <nav class="navigation-panel">
        <div class="nav-content">
          <h3>Navigation</h3>
          <ul class="nav-list">
            <li v-for="section in sections" :key="section.id">
              <button 
                class="nav-link"
                :class="{ active: activeSection === section.id }"
                @click="scrollToSection(section.id)"
              >
                {{ section.title }}
              </button>
            </li>
          </ul>
        </div>
      </nav>

      <!-- Content Panel -->
      <main class="content-panel" ref="contentPanel" @scroll="handleScroll">
        <div class="content-wrapper">
          <section 
            v-for="section in filteredSections" 
            :key="section.id"
            :id="`section-${section.id}`"
            class="content-section"
          >
            <h2>{{ section.title }}</h2>
            <div class="section-content">
              <p v-for="(item, idx) in section.items" :key="idx" class="item">
                {{ item }}
              </p>
            </div>
          </section>

          <!-- Loading indicator -->
          <div v-if="isLoading" class="loading">
            <span>Loading more content...</span>
          </div>

          <!-- End of content -->
          <div v-if="allItemsLoaded && sections.length > 0" class="end-message">
            <span>No more content</span>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import Header from './components/Header.vue'

interface Section {
  id: number
  title: string
  items: string[]
}

const contentPanel = ref<HTMLElement | null>(null)
const searchQuery = ref('')
const activeSection = ref(0)
const isLoading = ref(false)
const allItemsLoaded = ref(false)

const sections = ref<Section[]>([])
const itemsPerSection = ref(10)
const maxSections = ref(5)

// Generate random content items
const generateItems = (count: number, sectionIndex: number): string[] => {
  const items: string[] = []
  for (let i = 0; i < count; i++) {
    items.push(
      `Section ${sectionIndex} - Item ${i + 1}: ${generateLoremIpsum()}`
    )
  }
  return items
}

const generateLoremIpsum = (): string => {
  const words = [
    'lorem', 'ipsum', 'dolor', 'sit', 'amet', 'consectetur', 'adipiscing',
    'elit', 'sed', 'do', 'eiusmod', 'tempor', 'incididunt', 'ut', 'labore',
    'et', 'dolore', 'magna', 'aliqua', 'enim', 'ad', 'minim', 'veniam'
  ]
  const count = Math.floor(Math.random() * 15) + 10
  let text = ''
  for (let i = 0; i < count; i++) {
    text += words[Math.floor(Math.random() * words.length)] + ' '
  }
  return text.trim()
}

// Initialize sections
const initializeSections = () => {
  const newSections: Section[] = []
  for (let i = 0; i < maxSections.value; i++) {
    newSections.push({
      id: i,
      title: `Section ${i + 1}`,
      items: generateItems(itemsPerSection.value, i)
    })
  }
  sections.value = newSections
  allItemsLoaded.value = sections.value.length >= maxSections.value
}

// Filter sections based on search query
const filteredSections = computed(() => {
  if (!searchQuery.value.trim()) {
    return sections.value
  }

  const query = searchQuery.value.toLowerCase()
  return sections.value
    .map(section => ({
      ...section,
      items: section.items.filter(item => item.toLowerCase().includes(query))
    }))
    .filter(section => section.items.length > 0 || section.title.toLowerCase().includes(query))
})

// Handle search
const handleSearch = (query: string) => {
  searchQuery.value = query
}

// Scroll to section
const scrollToSection = (sectionId: number) => {
  activeSection.value = sectionId
  const element = document.getElementById(`section-${sectionId}`)
  if (element && contentPanel.value) {
    const scrollTop = element.offsetTop - 80 // Account for header height
    contentPanel.value.scrollTo({
      top: scrollTop,
      behavior: 'smooth'
    })
  }
}

// Handle scroll for infinite scrolling
const handleScroll = () => {
  if (!contentPanel.value) return

  const { scrollTop, scrollHeight, clientHeight } = contentPanel.value
  const scrollPercentage = (scrollTop + clientHeight) / scrollHeight

  // Load more when scrolled 80% down
  if (scrollPercentage > 0.8 && !isLoading.value && !allItemsLoaded.value) {
    loadMoreContent()
  }

  // Update active section based on scroll position
  updateActiveSection()
}

// Load more content
const loadMoreContent = async () => {
  if (isLoading.value || allItemsLoaded.value) return

  isLoading.value = true
  
  // Simulate API delay
  await new Promise(resolve => setTimeout(resolve, 500))

  if (sections.value.length < maxSections.value) {
    const newIndex = sections.value.length
    sections.value.push({
      id: newIndex,
      title: `Section ${newIndex + 1}`,
      items: generateItems(itemsPerSection.value, newIndex)
    })

    if (sections.value.length >= maxSections.value) {
      allItemsLoaded.value = true
    }
  }

  isLoading.value = false
}

// Update active section based on scroll position
const updateActiveSection = () => {
  if (!contentPanel.value) return

  const sections_elements = filteredSections.value
  for (let i = sections_elements.length - 1; i >= 0; i--) {
    const element = document.getElementById(`section-${sections_elements[i].id}`)
    if (element && contentPanel.value.scrollTop >= element.offsetTop - 100) {
      activeSection.value = sections_elements[i].id
      break
    }
  }
}

// Initialize on mount
onMounted(() => {
  initializeSections()
})
</script>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}

.panels-container {
  display: flex;
  flex: 1;
  overflow: hidden;
  margin-top: 80px;
}

/* Navigation Panel */
.navigation-panel {
  width: 250px;
  background-color: #ffffff;
  border-right: 1px solid #e0e0e0;
  overflow-y: auto;
  padding: 20px 0;
}

.nav-content {
  padding: 0 20px;
}

.nav-content h3 {
  font-size: 18px;
  margin-bottom: 20px;
  color: #333;
}

.nav-list {
  list-style: none;
}

.nav-link {
  display: block;
  width: 100%;
  padding: 12px 16px;
  margin-bottom: 8px;
  background: none;
  border: none;
  border-left: 3px solid transparent;
  color: #666;
  font-size: 14px;
  cursor: pointer;
  text-align: left;
  transition: all 0.3s ease;
}

.nav-link:hover {
  background-color: #f0f0f0;
  color: #333;
}

.nav-link.active {
  border-left-color: #007bff;
  background-color: #e7f1ff;
  color: #007bff;
  font-weight: 600;
}

/* Content Panel */
.content-panel {
  flex: 1;
  overflow-y: auto;
  background-color: #ffffff;
  scroll-behavior: smooth;
}

.content-wrapper {
  padding: 40px;
  max-width: 900px;
  margin: 0 auto;
}

.content-section {
  margin-bottom: 60px;
  scroll-margin-top: 100px;
}

.content-section h2 {
  font-size: 28px;
  color: #333;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 2px solid #007bff;
}

.section-content {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.item {
  line-height: 1.6;
  color: #555;
  font-size: 15px;
  padding: 12px;
  background-color: #f9f9f9;
  border-radius: 4px;
  transition: background-color 0.2s;
}

.item:hover {
  background-color: #f0f0f0;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px 20px;
  color: #999;
  font-size: 16px;
}

.loading::before {
  content: '';
  display: inline-block;
  width: 20px;
  height: 20px;
  margin-right: 10px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #007bff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.end-message {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px 20px;
  color: #999;
  font-size: 14px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

/* Scrollbar styling */
.navigation-panel::-webkit-scrollbar,
.content-panel::-webkit-scrollbar {
  width: 8px;
}

.navigation-panel::-webkit-scrollbar-track,
.content-panel::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.navigation-panel::-webkit-scrollbar-thumb,
.content-panel::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}

.navigation-panel::-webkit-scrollbar-thumb:hover,
.content-panel::-webkit-scrollbar-thumb:hover {
  background: #555;
}

/* Responsive design */
@media (max-width: 768px) {
  .panels-container {
    flex-direction: column;
  }

  .navigation-panel {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #e0e0e0;
    max-height: 200px;
  }

  .content-wrapper {
    padding: 20px;
  }
}
</style>
