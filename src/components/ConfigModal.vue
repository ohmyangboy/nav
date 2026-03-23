<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal">
      <header class="modal-header">
        <h2>配置导航</h2>
        <div class="header-actions">
          <button class="icon-btn" @click="showJson = !showJson" title="切换JSON模式">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
              <polyline points="14,2 14,8 20,8"/>
              <line x1="16" y1="13" x2="8" y2="13"/>
              <line x1="16" y1="17" x2="8" y2="17"/>
            </svg>
          </button>
          <button class="icon-btn" @click="$emit('close')">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M18 6 6 18M6 6l12 12"/>
            </svg>
          </button>
        </div>
      </header>

      <div class="modal-body">
        <!-- JSON 模式 -->
        <div v-if="showJson" class="json-mode">
          <div class="json-tabs">
            <button
              :class="['tab', { active: jsonTab === 'edit' }]"
              @click="jsonTab = 'edit'"
            >编辑</button>
            <button
              :class="['tab', { active: jsonTab === 'import' }]"
              @click="jsonTab = 'import'"
            >导入</button>
            <button
              :class="['tab', { active: jsonTab === 'export' }]"
              @click="jsonTab = 'export'"
            >导出</button>
          </div>

          <div v-if="jsonTab === 'edit'" class="tab-panel">
            <textarea
              v-model="configJson"
              class="json-editor"
              spellcheck="false"
              @input="jsonError = ''"
            />
            <p class="error-text" v-if="jsonError">{{ jsonError }}</p>
          </div>

          <div v-if="jsonTab === 'import'" class="tab-panel">
            <div class="import-area">
              <input
                type="file"
                ref="fileInput"
                accept=".json"
                @change="handleFileImport"
                hidden
              >
              <button class="action-btn primary" @click="$refs.fileInput.click()">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M17 8l-5-5-5 5M12 3v12"/>
                </svg>
                选择配置文件
              </button>
            </div>
          </div>

          <div v-if="jsonTab === 'export'" class="tab-panel">
            <textarea
              :value="formattedConfig"
              class="json-editor readonly"
              readonly
            />
            <button class="action-btn primary" @click="downloadConfig">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M7 10l5 5 5-5M12 15V3"/>
              </svg>
              下载配置文件
            </button>
          </div>
        </div>

        <!-- 可视化模式 -->
        <div v-else class="visual-mode">
          <draggable
            v-model="localConfig.categories"
            item-key="name"
            handle=".category-drag"
            ghost-class="dragging"
            class="categories-list"
          >
            <template #item="{ element: category, index: catIndex }">
              <div class="category-item">
                <div class="category-header">
                  <button class="drag-btn category-drag" title="拖拽排序">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                      <circle cx="9" cy="12" r="1"/><circle cx="9" cy="5" r="1"/><circle cx="9" cy="19" r="1"/>
                      <circle cx="15" cy="12" r="1"/><circle cx="15" cy="5" r="1"/><circle cx="15" cy="19" r="1"/>
                    </svg>
                  </button>
                  <input
                    v-model="category.name"
                    class="category-name-input"
                    placeholder="分类名称"
                  >
                  <button class="icon-btn danger" @click="removeCategory(catIndex)" title="删除分类">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                      <path d="M18 6 6 18M6 6l12 12"/>
                    </svg>
                  </button>
                </div>

                <draggable
                  v-model="category.sites"
                  item-key="name"
                  handle=".site-drag"
                  ghost-class="dragging"
                  group="sites"
                  class="sites-list"
                >
                  <template #item="{ element: site, index: siteIndex }">
                    <div class="site-item">
                      <button class="drag-btn site-drag" title="拖拽排序">
                        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                          <circle cx="9" cy="12" r="1"/><circle cx="9" cy="5" r="1"/><circle cx="9" cy="19" r="1"/>
                          <circle cx="15" cy="12" r="1"/><circle cx="15" cy="5" r="1"/><circle cx="15" cy="19" r="1"/>
                        </svg>
                      </button>
                      <input
                        v-model="site.icon"
                        class="site-icon-input"
                        placeholder="📎"
                        maxlength="2"
                      >
                      <input
                        v-model="site.name"
                        class="site-name-input"
                        placeholder="网站名称"
                      >
                      <input
                        v-model="site.url"
                        class="site-url-input"
                        placeholder="https://example.com"
                      >
                      <button class="icon-btn danger" @click="removeSite(category, siteIndex)" title="删除网站">
                        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                          <path d="M18 6 6 18M6 6l12 12"/>
                        </svg>
                      </button>
                    </div>
                  </template>
                </draggable>

                <button class="add-site-btn" @click="addSite(category)">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <path d="M12 5v14M5 12h14"/>
                  </svg>
                  添加网站
                </button>
              </div>
            </template>
          </draggable>

          <button class="add-category-btn" @click="addCategory">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M12 5v14M5 12h14"/>
            </svg>
            添加分类
          </button>
        </div>
      </div>

      <footer class="modal-footer">
        <button class="action-btn" @click="$emit('close')">取消</button>
        <button class="action-btn primary" @click="saveConfig">
          保存
        </button>
      </footer>
    </div>
  </div>
</template>

<script>
import { ref, reactive, computed, watch } from 'vue'
import draggable from 'vuedraggable'

export default {
  name: 'ConfigModal',
  components: {
    draggable
  },
  props: {
    config: {
      type: Object,
      required: true
    }
  },
  emits: ['close', 'save'],
  setup(props, { emit }) {
    const showJson = ref(false)
    const jsonTab = ref('edit')
    const jsonError = ref('')
    const fileInput = ref(null)

    // 深拷贝配置用于本地编辑
    const localConfig = reactive({
      categories: JSON.parse(JSON.stringify(props.config.categories))
    })

    const configJson = ref(JSON.stringify(props.config, null, 2))

    watch(() => props.config, (newConfig) => {
      localConfig.categories = JSON.parse(JSON.stringify(newConfig.categories))
      configJson.value = JSON.stringify(newConfig, null, 2)
    }, { deep: true })

    const formattedConfig = computed(() => {
      return JSON.stringify(props.config, null, 2)
    })

    const addCategory = () => {
      localConfig.categories.push({
        name: '新分类',
        sites: []
      })
    }

    const removeCategory = (index) => {
      if (confirm('确定要删除这个分类吗？')) {
        localConfig.categories.splice(index, 1)
      }
    }

    const addSite = (category) => {
      category.sites.push({
        name: '',
        url: '',
        icon: '🔖'
      })
    }

    const removeSite = (category, index) => {
      category.sites.splice(index, 1)
    }

    const saveConfig = () => {
      if (showJson.value && jsonTab.value === 'edit') {
        // JSON 模式保存
        try {
          const parsed = JSON.parse(configJson.value)
          emit('save', parsed)
        } catch (e) {
          jsonError.value = 'JSON 格式错误：' + e.message
          return
        }
      } else {
        // 可视化模式保存
        emit('save', { categories: localConfig.categories })
      }
    }

    const handleFileImport = (e) => {
      const file = e.target.files[0]
      if (!file) return

      const reader = new FileReader()
      reader.onload = (event) => {
        try {
          const imported = JSON.parse(event.target.result)
          localConfig.categories = imported.categories || []
          configJson.value = JSON.stringify(imported, null, 2)
          showJson.value = false
          emit('save', imported)
        } catch (err) {
          alert('配置文件格式错误')
        }
      }
      reader.readAsText(file)
    }

    const downloadConfig = () => {
      const blob = new Blob([formattedConfig.value], { type: 'application/json' })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url
      a.download = `nav-config-${new Date().toISOString().split('T')[0]}.json`
      a.click()
      URL.revokeObjectURL(url)
    }

    return {
      showJson,
      jsonTab,
      jsonError,
      fileInput,
      localConfig,
      configJson,
      formattedConfig,
      addCategory,
      removeCategory,
      addSite,
      removeSite,
      saveConfig,
      handleFileImport,
      downloadConfig
    }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  z-index: 1000;
}

.modal {
  background: var(--bg-secondary);
  border-radius: 24px;
  width: 100%;
  max-width: 720px;
  max-height: 85vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px 0;
}

.modal-header h2 {
  font-family: 'Playfair Display', serif;
  font-size: 20px;
  font-weight: 600;
}

.header-actions {
  display: flex;
  gap: 8px;
}

.icon-btn {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 10px;
  background: var(--bg-primary);
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s ease;
}

.icon-btn:hover {
  background: var(--accent-light);
  color: var(--accent);
}

.icon-btn.danger:hover {
  background: #fee2e2;
  color: #dc2626;
}

.modal-body {
  padding: 20px 24px;
  flex: 1;
  overflow-y: auto;
}

/* JSON 模式 */
.json-tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
  padding: 4px;
  background: var(--bg-primary);
  border-radius: 12px;
}

.tab {
  flex: 1;
  padding: 8px 16px;
  border: none;
  border-radius: 8px;
  background: transparent;
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.tab.active {
  background: var(--bg-secondary);
  color: var(--text-primary);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.tab-panel {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.json-editor {
  width: 100%;
  height: 280px;
  padding: 16px;
  border: 1px solid var(--border);
  border-radius: 12px;
  background: var(--bg-primary);
  font-family: 'SF Mono', monospace;
  font-size: 13px;
  line-height: 1.6;
  resize: none;
  outline: none;
}

.json-editor:focus {
  border-color: var(--accent);
}

.json-editor.readonly {
  background: var(--bg-primary);
  color: var(--text-secondary);
}

.error-text {
  color: #dc2626;
  font-size: 13px;
}

.import-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  padding: 40px 20px;
  background: var(--bg-primary);
  border-radius: 12px;
  border: 2px dashed var(--border);
}

/* 可视化模式 */
.visual-mode {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.categories-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.category-item {
  background: var(--bg-primary);
  border-radius: 16px;
  padding: 16px;
  border: 1px solid var(--border);
}

.category-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

.drag-btn {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 6px;
  background: transparent;
  color: var(--text-secondary);
  cursor: grab;
  transition: all 0.2s ease;
}

.drag-btn:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

.drag-btn:active {
  cursor: grabbing;
}

.category-name-input {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid transparent;
  border-radius: 8px;
  background: transparent;
  font-size: 16px;
  font-weight: 600;
  font-family: inherit;
  color: var(--text-primary);
  outline: none;
}

.category-name-input:hover,
.category-name-input:focus {
  background: var(--bg-secondary);
  border-color: var(--border);
}

.sites-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-left: 36px;
}

.site-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px;
  background: var(--bg-secondary);
  border-radius: 10px;
  border: 1px solid var(--border);
}

.site-icon-input {
  width: 44px;
  padding: 8px;
  border: 1px solid transparent;
  border-radius: 8px;
  background: transparent;
  font-size: 16px;
  text-align: center;
  outline: none;
}

.site-icon-input:hover,
.site-icon-input:focus {
  background: var(--bg-primary);
  border-color: var(--border);
}

.site-name-input {
  flex: 0 0 120px;
  padding: 8px 12px;
  border: 1px solid transparent;
  border-radius: 8px;
  background: transparent;
  font-size: 14px;
  font-family: inherit;
  outline: none;
}

.site-name-input:hover,
.site-name-input:focus {
  background: var(--bg-primary);
  border-color: var(--border);
}

.site-url-input {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid transparent;
  border-radius: 8px;
  background: transparent;
  font-size: 14px;
  font-family: inherit;
  color: var(--text-secondary);
  outline: none;
}

.site-url-input:hover,
.site-url-input:focus {
  background: var(--bg-primary);
  border-color: var(--border);
  color: var(--text-primary);
}

.add-site-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-left: 36px;
  margin-top: 8px;
  padding: 8px 12px;
  border: none;
  border-radius: 8px;
  background: transparent;
  color: var(--accent);
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.add-site-btn:hover {
  background: var(--accent-light);
}

.add-category-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 16px;
  border: 2px dashed var(--border);
  border-radius: 12px;
  background: transparent;
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.add-category-btn:hover {
  border-color: var(--accent);
  color: var(--accent);
  background: var(--accent-light);
}

.dragging {
  opacity: 0.5;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 0 24px 24px;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  border: 1px solid var(--border);
  border-radius: 10px;
  background: var(--bg-primary);
  color: var(--text-primary);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.action-btn:hover {
  background: var(--accent-light);
  border-color: var(--accent);
  color: var(--accent);
}

.action-btn.primary {
  background: var(--accent);
  border-color: var(--accent);
  color: white;
}

.action-btn.primary:hover {
  opacity: 0.9;
}

@media (max-width: 640px) {
  .site-name-input {
    flex: 1;
  }

  .site-url-input {
    display: none;
  }

  .sites-list {
    margin-left: 0;
  }

  .add-site-btn {
    margin-left: 0;
  }
}
</style>
