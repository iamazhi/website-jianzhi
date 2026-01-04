# 见地应用官网

这是见地应用的官方网站，用于微信开放平台审核等用途。

## 文件结构

```
website/
├── index.html          # 首页
├── about.html          # 关于页面
├── privacy.html        # 隐私政策页面
├── css/
│   └── style.css       # 样式文件
├── js/                 # JavaScript 目录（当前未使用）
└── README.md           # 本文件
```

## 使用说明

### 本地预览

1. 直接在浏览器中打开 `index.html` 文件
2. 或使用本地服务器：
   ```bash
   # Python 3
   cd website
   python3 -m http.server 8000
   
   # 然后访问 http://localhost:8000
   ```

### 部署

#### 方式一：GitHub Pages（推荐）

1. 在 GitHub 仓库设置中启用 GitHub Pages
2. 选择 `website/` 目录作为源目录
3. 访问地址：`https://username.github.io/repository-name/website/`

#### 方式二：其他静态托管服务

- **Netlify**：直接拖拽 `website` 文件夹到 Netlify
- **Vercel**：连接 GitHub 仓库，设置构建目录为 `website`
- **其他**：任何支持静态网站的托管服务都可以

## 需要配置的内容

在使用前，请修改以下内容：

### 1. App Store 下载链接

在 `index.html` 中查找并替换：
```html
href="https://apps.apple.com/cn/app/见地/id[您的AppID]"
```
将 `[您的AppID]` 替换为实际的 App Store ID。

### 2. 联系邮箱

在以下文件中替换 `[您的联系邮箱]`：
- `index.html`（如果有）
- `about.html`
- `privacy.html`

### 3. 应用截图

当前截图路径指向 `../assets/branding/`，如果部署到独立域名，需要：
- 将截图复制到 `website/images/` 目录
- 修改 `index.html` 中的图片路径为 `images/screenshot_self.png` 等

### 4. Logo（可选）

如果需要使用 Logo，可以：
- 将 `assets/branding/logo.png` 复制到 `website/images/`
- 在导航栏或其他位置使用

## 自定义域名（可选）

如果需要使用自定义域名：

1. 购买域名（如 `jianzhi.app`）
2. 在域名 DNS 设置中添加 CNAME 记录指向 GitHub Pages 地址
3. 在 GitHub Pages 设置中配置自定义域名

## 样式说明

- 使用黑白配色，符合应用设计风格
- 响应式设计，支持桌面和移动端
- 使用系统字体，确保跨平台兼容性

## 更新记录

- 2025-01-XX：初始版本创建

## 注意事项

1. 确保所有链接都能正常访问
2. 定期更新应用版本号和信息
3. 保持内容与 App Store 描述一致
4. 确保隐私政策内容准确完整

