# 官网部署指南

## 快速开始

### 1. 配置必要信息

在使用前，请在以下文件中修改占位符：

#### index.html
- 将 `[您的AppID]` 替换为实际的 App Store 应用 ID
- 如果 App Store 链接格式不同，请相应调整

#### about.html 和 privacy.html
- 将 `[您的联系邮箱]` 替换为实际的联系邮箱

### 2. 本地测试

在浏览器中打开 `index.html` 查看效果，或使用本地服务器：

```bash
cd website
python3 -m http.server 8000
# 然后访问 http://localhost:8000
```

### 3. 部署方式

#### 方式一：GitHub Pages（推荐）

1. 将代码推送到 GitHub 仓库
2. 在仓库 Settings > Pages 中：
   - Source: Deploy from a branch
   - Branch: 选择分支（如 main）
   - Folder: `/website`
3. 保存后，GitHub Pages 会自动构建
4. 访问地址：`https://username.github.io/repository-name/website/`

**注意**：如果希望直接访问根路径（不需要 `/website`），可以：
- 创建 `gh-pages` 分支，将 website 目录内容放到根目录
- 或使用 GitHub Actions 自动构建

#### 方式二：Netlify

1. 注册/登录 Netlify
2. 选择 "Add new site" > "Deploy manually"
3. 将 `website` 文件夹拖拽到部署区域
4. 或连接 GitHub 仓库，设置：
   - Base directory: `website`
   - Build command: 留空
   - Publish directory: `website`

#### 方式三：Vercel

1. 注册/登录 Vercel
2. 导入 GitHub 仓库
3. 设置：
   - Root Directory: `website`
   - Build Command: 留空
   - Output Directory: `.`

#### 方式四：其他静态托管服务

任何支持静态网站的托管服务都可以使用，只需要：
- 上传 `website` 目录下的所有文件
- 确保 `index.html` 在根目录

### 4. 自定义域名（可选）

1. 购买域名（如 `jianzhi.app`）
2. 在域名 DNS 中添加 CNAME 记录：
   - 名称：`@` 或 `www`
   - 值：GitHub Pages/Netlify/Vercel 提供的域名
3. 在托管服务设置中配置自定义域名

## 文件说明

- `index.html` - 首页
- `about.html` - 关于页面
- `privacy.html` - 隐私政策页面
- `css/style.css` - 样式文件
- `images/` - 图片资源目录
- `README.md` - 项目说明
- `DEPLOY.md` - 本部署指南

## 注意事项

1. **图片路径**：所有图片已在 `images/` 目录，路径已配置正确
2. **相对路径**：所有链接使用相对路径，可以在任何路径下正常工作
3. **响应式设计**：网站支持移动端和桌面端，建议在不同设备测试
4. **内容更新**：如需更新内容，直接修改 HTML 文件即可

## 微信开放平台审核

提交到微信开放平台时：
1. 确保网站可以正常访问
2. 确保所有链接都能正常打开
3. 确保包含应用介绍、下载链接等必要内容
4. 确保隐私政策页面可访问

## 后续维护

- 定期更新应用版本号
- 更新应用截图（如有新版本）
- 保持内容与 App Store 描述一致
- 及时更新隐私政策（如有变更）

