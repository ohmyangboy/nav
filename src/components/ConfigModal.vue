<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal">
      <header class="modal-header">
        <h2>配置导航</h2>
        <button class="close-btn" @click="$emit('close')">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M18 6 6 18M6 6l12 12"/>
          </svg>
        </button>
      </header>

      <div class="modal-body">
        <div class="config-tabs">
          <button
            :class="['tab', { active: activeTab === 'edit' }]"
            @click="activeTab = 'edit'"
          >编辑配置</button>
          <button
            :class="['tab', { active: activeTab === 'import' }]"
            @click="activeTab = 'import'"
          >导入</button>
          <button
            :class="['tab', { active: activeTab === 'export' }]"
            @click="activeTab = 'export'"
          >导出</button>
        </div>

        <!-- 编辑配置 -->
        <div v-if="activeTab === 'edit'" class="tab-content">
          <textarea
            v-model="configText"
            class="config-editor"
            spellcheck="false"
          />
          <p class="hint">编辑完成后点击保存，配置将以 JSON 格式存储在浏览器中</p>
        </div>

        <!-- 导入 -->
        <div v-if="activeTab === 'import'" class="tab-content">
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
            <p class="hint">支持导入 JSON 格式的配置文件</p>
          </div>
        </div>

        <!-- 导出 -->
        <div v-if="activeTab === 'export'" class="tab-content">
          <textarea
            :value="formattedConfig"
            class="config-editor readonly"
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

      <footer class="modal-footer">
        <button class="action-btn" @click="$emit('close')">取消</button>
        <button
          v-if="activeTab === 'edit'"
          class="action-btn primary"
          @click="saveConfig"
        >
          保存
        </button>
      </footer>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'

export default {
  name: 'ConfigModal',
  props: {
    config: {
      type: Object,
      required: true
    }
  },
  emits: ['close', 'save'],
  setup(props, { emit }) {
    const activeTab = ref('edit')
    const configText = ref(JSON.stringify(props.config, null, 2))
    const fileInput = ref(null)

    watch(() => props.config, (newConfig) => {
      configText.value = JSON.stringify(newConfig, null, 2)
    })

    const formattedConfig = computed(() => {
      return JSON.stringify(props.config, null, 2)
    })

    const saveConfig = () => {
      try {
        const newConfig = JSON.parse(configText.value)
        emit('save', newConfig)
      } catch (e) {
        alert('JSON 格式错误，请检查配置')
      }
    }

    const handleFileImport = (e) => {
      const file = e.target.files[0]
      if (!file) return

      const reader = new FileReader()
      reader.onload = (event) => {
        try {
          const imported = JSON.parse(event.target.result)
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
      activeTab,
      configText,
      fileInput,
      formattedConfig,
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
  max-width: 600px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 24px 24px 0;
}

.modal-header h2 {
  font-family: 'Playfair Display', serif;
  font-size: 22px;
  font-weight: 600;
}

.close-btn {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 50%;
  background: var(--bg-primary);
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s ease;
}

.close-btn:hover {
  background: var(--accent-light);
  color: var(--accent);
}

.modal-body {
  padding: 24px;
  flex: 1;
  overflow-y: auto;
}

.config-tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
  padding: 4px;
  background: var(--bg-primary);
  border-radius: 12px;
}

.tab {
  flex: 1;
  padding: 10px 16px;
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

.tab-content {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.config-editor {
  width: 100%;
  height: 300px;
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

.config-editor:focus {
  border-color: var(--accent);
}

.config-editor.readonly {
  background: var(--bg-primary);
  color: var(--text-secondary);
}

.hint {
  font-size: 13px;
  color: var(--text-secondary);
  text-align: center;
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
</style>
