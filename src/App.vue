<template>
  <div class="app" :class="{ 'dark-theme': isDark }">
    <header class="header">
      <h1 class="logo">Nav</h1>
      <div class="header-actions">
        <button class="theme-btn" @click="toggleTheme" :title="isDark ? '切换浅色模式' : '切换深色模式'">
          <svg v-if="!isDark" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
          </svg>
          <svg v-else width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="5"/>
            <line x1="12" y1="1" x2="12" y2="3"/>
            <line x1="12" y1="21" x2="12" y2="23"/>
            <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/>
            <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
            <line x1="1" y1="12" x2="3" y2="12"/>
            <line x1="21" y1="12" x2="23" y2="12"/>
            <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/>
            <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
          </svg>
        </button>
        <button class="config-btn" @click="showConfig = true">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="3"/>
            <path d="M12 1v6m0 6v6m4.22-10.22l4.24-4.24M6.34 6.34L2.1 2.1m17.9 9.9h-6m-6 0H1.9"/>
          </svg>
          配置
        </button>
      </div>
    </header>

    <main class="main">
      <SearchBox
        v-model="searchQuery"
        @focus="isSearching = true"
        @blur="isSearching = false"
      />

      <div class="categories" v-if="!searchQuery">
        <CategoryCard
          v-for="category in categories"
          :key="category.name"
          :category="category"
          @open="openLink"
        />
      </div>

      <SearchResults
        v-else
        :sites="filteredSites"
        @open="openLink"
      />
    </main>

    <ConfigModal
      v-if="showConfig"
      :config="config"
      @close="showConfig = false"
      @save="saveConfig"
    />
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import SearchBox from './components/SearchBox.vue'
import CategoryCard from './components/CategoryCard.vue'
import SearchResults from './components/SearchResults.vue'
import ConfigModal from './components/ConfigModal.vue'
import { defaultConfig } from './config/default.js'

export default {
  name: 'App',
  components: {
    SearchBox,
    CategoryCard,
    SearchResults,
    ConfigModal
  },
  setup() {
    const searchQuery = ref('')
    const isSearching = ref(false)
    const showConfig = ref(false)
    const config = ref(defaultConfig)
    const isDark = ref(false)

    // 加载本地配置和主题
    onMounted(() => {
      const saved = localStorage.getItem('nav-config')
      if (saved) {
        try {
          config.value = JSON.parse(saved)
        } catch (e) {
          console.error('配置加载失败', e)
        }
      }
      // 加载主题设置
      const savedTheme = localStorage.getItem('nav-theme')
      if (savedTheme) {
        isDark.value = savedTheme === 'dark'
      } else {
        // 检测系统主题偏好
        isDark.value = window.matchMedia('(prefers-color-scheme: dark)').matches
      }
    })

    const toggleTheme = () => {
      isDark.value = !isDark.value
      localStorage.setItem('nav-theme', isDark.value ? 'dark' : 'light')
    }

    const categories = computed(() => config.value.categories || [])

    // 搜索过滤
    const filteredSites = computed(() => {
      if (!searchQuery.value) return []
      const query = searchQuery.value.toLowerCase()
      const results = []

      config.value.categories.forEach(cat => {
        cat.sites.forEach(site => {
          if (site.name.toLowerCase().includes(query) ||
              site.url.toLowerCase().includes(query)) {
            results.push({ ...site, category: cat.name })
          }
        })
      })

      return results
    })

    const openLink = (url) => {
      window.open(url, '_blank')
    }

    const saveConfig = (newConfig) => {
      config.value = newConfig
      localStorage.setItem('nav-config', JSON.stringify(newConfig))
      showConfig.value = false
    }

    return {
      searchQuery,
      isSearching,
      showConfig,
      config,
      categories,
      filteredSites,
      openLink,
      saveConfig,
      isDark,
      toggleTheme
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg-primary: #faf9f7;
  --bg-secondary: #ffffff;
  --text-primary: #1a1a1a;
  --text-secondary: #666666;
  --accent: #2d5a4a;
  --accent-light: #e8f0ed;
  --border: rgba(0, 0, 0, 0.08);
  --shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  --shadow-hover: 0 8px 30px rgba(0, 0, 0, 0.1);
  --overlay-bg: rgba(0, 0, 0, 0.3);
}

/* 暗色主题 */
.dark-theme {
  --bg-primary: #0d1117;
  --bg-secondary: #161b22;
  --text-primary: #c9d1d9;
  --text-secondary: #8b949e;
  --accent: #58a6ff;
  --accent-light: rgba(88, 166, 255, 0.15);
  --border: rgba(255, 255, 255, 0.1);
  --shadow: 0 2px 12px rgba(0, 0, 0, 0.3);
  --shadow-hover: 0 8px 30px rgba(0, 0, 0, 0.4);
  --overlay-bg: rgba(0, 0, 0, 0.7);
}

body {
  font-family: 'Source Sans 3', -apple-system, sans-serif;
  background: var(--bg-primary);
  color: var(--text-primary);
  min-height: 100vh;
  line-height: 1.6;
}

.app {
  min-height: 100vh;
  padding: 40px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 80px;
}

.logo {
  font-family: 'Playfair Display', serif;
  font-size: 28px;
  font-weight: 600;
  color: var(--text-primary);
  letter-spacing: -0.5px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.theme-btn {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid var(--border);
  border-radius: 50%;
  background: var(--bg-secondary);
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.3s ease;
}

.theme-btn:hover {
  border-color: var(--accent);
  color: var(--accent);
  transform: rotate(15deg);
}

.config-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  border: 1px solid var(--border);
  border-radius: 24px;
  background: var(--bg-secondary);
  color: var(--text-secondary);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.config-btn:hover {
  border-color: var(--accent);
  color: var(--accent);
}

.main {
  max-width: 900px;
  margin: 0 auto;
}

.categories {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
  margin-top: 60px;
}

@media (max-width: 768px) {
  .app {
    padding: 20px;
  }

  .header {
    margin-bottom: 40px;
  }

  .categories {
    grid-template-columns: 1fr;
    gap: 16px;
    margin-top: 40px;
  }
}
</style>
