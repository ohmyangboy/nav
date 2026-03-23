# Nav - 个人导航网站

一个简约高效的静态导航网站，支持自定义配置、导入导出，可部署到 GitHub Pages。

![Preview](https://user-images.githubusercontent.com/preview.png)

## 特性

- 🎯 **极简设计** - 大面积留白，专注于快速访问
- 🔍 **智能搜索** - 支持 Cmd/Ctrl + K 快捷键，实时过滤
- 📂 **分类管理** - 支持多类别网站分组
- 🎨 **优雅视觉** - Playfair Display 衬线字体 + Source Sans 3 正文字体
- 💾 **配置灵活** - 可视化编辑、导入导出 JSON 配置
- 📱 **响应式** - 完美适配桌面和移动设备
- 🚀 **快速部署** - 一键部署到 GitHub Pages

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/yourusername/nav.git
cd nav
```

### 2. 安装依赖

```bash
npm install
```

### 3. 本地开发

```bash
npm run dev
```

### 4. 构建

```bash
npm run build
```

## 自定义配置

点击右上角的「配置」按钮可以：

1. **编辑配置** - 直接修改 JSON 格式的网站列表
2. **导入配置** - 从本地 JSON 文件导入
3. **导出配置** - 将当前配置下载为 JSON 文件备份

### 配置格式示例

```json
{
  "categories": [
    {
      "name": "常用工具",
      "sites": [
        { "name": "Google", "url": "https://google.com", "icon": "🔍" },
        { "name": "GitHub", "url": "https://github.com", "icon": "💻" }
      ]
    }
  ]
}
```

## 部署到 GitHub Pages

### 1. 创建 GitHub 仓库

在 GitHub 创建新仓库，命名为 `nav`（或其他名称）。

### 2. 修改配置

编辑 `vite.config.js`，将 `base` 修改为仓库名：

```js
export default defineConfig({
  base: '/nav/',  // 如果仓库名是 nav
  // ...
})
```

如果使用用户名.github.io 作为主站，则设置为 `base: '/'`。

### 3. 推送代码

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/nav.git
git push -u origin main
```

### 4. 启用 GitHub Pages

1. 打开仓库 Settings → Pages
2. Source 选择 "GitHub Actions"

部署完成后，访问 `https://yourusername.github.io/nav/` 查看网站。

## 键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd/Ctrl + K` | 聚焦搜索框 |
| `Esc` | 取消搜索 |

## 技术栈

- Vue 3 - 渐进式 JavaScript 框架
- Vite - 下一代前端构建工具
- CSS Variables - 主题定制
- GitHub Actions - 自动部署

## 许可证

MIT
