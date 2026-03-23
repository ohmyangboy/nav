<template>
  <div class="search-results">
    <div v-if="sites.length === 0" class="empty">
      <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
        <circle cx="11" cy="11" r="8"/>
        <path d="m21 21-4.35-4.35"/>
      </svg>
      <p>未找到匹配的网站</p>
    </div>
    <div v-else class="results-list">
      <a
        v-for="site in sites"
        :key="site.url"
        :href="site.url"
        target="_blank"
        class="result-item"
        @click.prevent="$emit('open', site.url)"
      >
        <span class="result-icon" v-if="site.icon">{{ site.icon }}</span>
        <span class="result-icon default" v-else>
          {{ site.name.charAt(0).toUpperCase() }}
        </span>
        <div class="result-info">
          <span class="result-name">{{ site.name }}</span>
          <span class="result-category">{{ site.category }}</span>
        </div>
        <span class="result-url">{{ formatUrl(site.url) }}</span>
      </a>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SearchResults',
  props: {
    sites: {
      type: Array,
      default: () => []
    }
  },
  emits: ['open'],
  setup() {
    const formatUrl = (url) => {
      try {
        return new URL(url).hostname
      } catch {
        return url
      }
    }

    return { formatUrl }
  }
}
</script>

<style scoped>
.search-results {
  margin-top: 40px;
}

.empty {
  text-align: center;
  padding: 60px 20px;
  color: var(--text-secondary);
}

.empty svg {
  margin-bottom: 16px;
  opacity: 0.5;
}

.results-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.result-item {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 16px 20px;
  background: var(--bg-secondary);
  border-radius: 12px;
  text-decoration: none;
  color: var(--text-primary);
  box-shadow: var(--shadow);
  transition: all 0.2s ease;
}

.result-item:hover {
  box-shadow: var(--shadow-hover);
  transform: translateX(4px);
}

.result-icon {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--bg-primary);
  border-radius: 10px;
  font-size: 16px;
  flex-shrink: 0;
}

.result-icon.default {
  font-family: 'Playfair Display', serif;
  font-weight: 600;
  color: var(--accent);
}

.result-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.result-name {
  font-size: 16px;
  font-weight: 500;
}

.result-category {
  font-size: 13px;
  color: var(--text-secondary);
}

.result-url {
  font-size: 13px;
  color: var(--text-secondary);
  opacity: 0.7;
}
</style>
