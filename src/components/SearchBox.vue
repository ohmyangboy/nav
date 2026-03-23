<template>
  <div class="search-box" :class="{ focused: isFocused }">
    <div class="search-icon">
      <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="11" cy="11" r="8"/>
        <path d="m21 21-4.35-4.35"/>
      </svg>
    </div>
    <input
      ref="input"
      type="text"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      @focus="isFocused = true; $emit('focus')"
      @blur="isFocused = false; $emit('blur')"
      placeholder="搜索网站或输入关键词..."
      class="search-input"
    >
    <div class="shortcut" v-if="!modelValue && !isFocused">
      <kbd>Cmd</kbd>
      <kbd>K</kbd>
    </div>
    <button
      v-if="modelValue"
      class="clear-btn"
      @click="$emit('update:modelValue', '')"
    >
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M18 6 6 18M6 6l12 12"/>
      </svg>
    </button>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'

export default {
  name: 'SearchBox',
  props: ['modelValue'],
  emits: ['update:modelValue', 'focus', 'blur'],
  setup() {
    const isFocused = ref(false)
    const input = ref(null)

    const handleKeydown = (e) => {
      if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
        e.preventDefault()
        input.value?.focus()
      }
      if (e.key === 'Escape') {
        input.value?.blur()
      }
    }

    onMounted(() => {
      document.addEventListener('keydown', handleKeydown)
    })

    onUnmounted(() => {
      document.removeEventListener('keydown', handleKeydown)
    })

    return {
      isFocused,
      input
    }
  }
}
</script>

<style scoped>
.search-box {
  position: relative;
  display: flex;
  align-items: center;
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 16px 20px;
  transition: all 0.3s ease;
  box-shadow: var(--shadow);
}

.search-box.focused {
  border-color: var(--accent);
  box-shadow: 0 0 0 4px var(--accent-light);
}

.search-icon {
  color: var(--text-secondary);
  margin-right: 16px;
  flex-shrink: 0;
}

.search-input {
  flex: 1;
  border: none;
  background: transparent;
  font-size: 18px;
  font-family: inherit;
  color: var(--text-primary);
  outline: none;
}

.search-input::placeholder {
  color: var(--text-secondary);
  opacity: 0.6;
}

.shortcut {
  display: flex;
  gap: 4px;
  margin-left: 16px;
}

kbd {
  padding: 4px 8px;
  background: var(--bg-primary);
  border: 1px solid var(--border);
  border-radius: 6px;
  font-size: 12px;
  font-family: inherit;
  color: var(--text-secondary);
}

.clear-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border: none;
  border-radius: 50%;
  background: var(--bg-primary);
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s ease;
  margin-left: 12px;
}

.clear-btn:hover {
  background: var(--accent-light);
  color: var(--accent);
}

@media (max-width: 768px) {
  .shortcut {
    display: none;
  }

  .search-input {
    font-size: 16px;
  }
}
</style>
