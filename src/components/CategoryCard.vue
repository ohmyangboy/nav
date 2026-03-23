<template>
  <div class="category-card">
    <h2 class="category-title">{{ category.name }}</h2>
    <div class="sites">
      <a
        v-for="site in category.sites"
        :key="site.url"
        :href="site.url"
        target="_blank"
        class="site-link"
        @click.prevent="$emit('open', site.url)"
      >
        <span class="site-icon" v-if="site.icon">{{ site.icon }}</span>
        <span class="site-icon default" v-else>
          {{ site.name.charAt(0).toUpperCase() }}
        </span>
        <span class="site-name">{{ site.name }}</span>
      </a>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CategoryCard',
  props: {
    category: {
      type: Object,
      required: true
    }
  },
  emits: ['open']
}
</script>

<style scoped>
.category-card {
  background: var(--bg-secondary);
  border-radius: 20px;
  padding: 24px;
  box-shadow: var(--shadow);
  transition: all 0.3s ease;
}

.category-card:hover {
  box-shadow: var(--shadow-hover);
  transform: translateY(-2px);
}

.category-title {
  font-family: 'Playfair Display', serif;
  font-size: 18px;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border);
}

.sites {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.site-link {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 12px;
  border-radius: 10px;
  text-decoration: none;
  color: var(--text-primary);
  transition: all 0.2s ease;
}

.site-link:hover {
  background: var(--accent-light);
  color: var(--accent);
}

.site-icon {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--bg-primary);
  border-radius: 8px;
  font-size: 14px;
  flex-shrink: 0;
}

.site-icon.default {
  font-family: 'Playfair Display', serif;
  font-weight: 600;
  color: var(--accent);
}

.site-name {
  font-size: 15px;
  font-weight: 400;
}
</style>
