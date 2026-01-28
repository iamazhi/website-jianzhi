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

#### 方式四：阿里云 ECS

##### 前置准备

1. **购买 ECS 实例**
   - 操作系统：推荐 Ubuntu 22.04 LTS 或 CentOS 7/8
   - 最低配置：1核1G 即可（静态网站资源占用很小）
   - 确保有公网 IP

2. **域名准备（可选）**
   - 已有域名：可在阿里云域名控制台配置解析
   - 无域名：可直接使用 ECS 公网 IP 访问（不推荐生产环境）

##### 部署步骤

**1. 连接到 ECS 服务器**

```bash
ssh root@your-ecs-ip
# 或使用其他用户名
ssh username@your-ecs-ip
```

**2. 安装 Nginx（推荐）**

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nginx -y

# CentOS/RHEL
sudo yum install epel-release -y
sudo yum install nginx -y

# 启动 Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# 检查状态
sudo systemctl status nginx
```

**3. 上传网站文件**

**方法 A：使用 SCP 上传**

```bash
# 在本地电脑执行（替换为你的服务器 IP 和路径）
scp -r index.html about.html privacy.html css/ images/ js/ .well-known/ root@your-ecs-ip:/var/www/html/

# 如果有其他目录，也一并上传
```

**方法 B：使用 Git**

> **注意**：如果克隆私有仓库或使用 SSH 方式，需要先配置 GitHub SSH 密钥。详细步骤请参考：[GitHub SSH 密钥配置指南](./GITHUB_SSH_SETUP.md)

```bash
# 在服务器上安装 Git
sudo apt install git -y  # Ubuntu/Debian
# 或
sudo yum install git -y  # CentOS/RHEL

# 克隆仓库（根据仓库类型选择）
# 公开仓库可以使用 HTTPS
cd /var/www/
sudo git clone https://github.com/your-username/your-repo.git html

# 私有仓库或使用 SSH（需要先配置 SSH 密钥）
# sudo git clone git@github.com:your-username/your-repo.git html

# 或者直接下载 ZIP 文件并解压
```

**方法 C：使用 FTP/SFTP 工具**

- 使用 FileZilla、WinSCP 等工具
- 连接到服务器后，将文件上传到 `/var/www/html/` 目录

**4. 配置 Nginx**

创建或编辑 Nginx 配置文件：

```bash
sudo nano /etc/nginx/sites-available/default
# 或 CentOS
sudo nano /etc/nginx/conf.d/default.conf
```

配置内容示例：

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;  # 替换为你的域名
    # 如果暂时没有域名，可以注释掉 server_name，或使用 _ 通配符
    # server_name _;

    root /var/www/html;
    index index.html;

    # 支持 Universal Links 的 .well-known 目录
    location /.well-known {
        allow all;
    }

    # 静态文件缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 主页面
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 字符编码
    charset utf-8;

    # 日志
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}
```

测试配置并重载：

```bash
# 测试配置文件语法
sudo nginx -t

# 重载配置
sudo systemctl reload nginx
```

**5. 配置防火墙**

```bash
# Ubuntu (UFW)
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable

# CentOS (firewalld)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

**6. 配置阿里云安全组**

在阿里云控制台：

1. 进入 **ECS 实例** > **安全组** > **配置规则**
2. 添加入站规则：
   - **规则方向**：入方向
   - **授权策略**：允许
   - **协议类型**：自定义 TCP
   - **端口范围**：80/80
   - **授权对象**：0.0.0.0/0（如需 HTTPS，也添加 443/443）

**7. 配置域名解析（如有域名）**

在阿里云域名控制台：

1. 进入 **域名** > **解析设置**
2. 添加 A 记录：
   - **记录类型**：A
   - **主机记录**：@ 或 www
   - **记录值**：ECS 公网 IP
   - **TTL**：10分钟（默认）

**8. 配置 HTTPS（推荐，用于生产环境）**

**方法 A：使用 Let's Encrypt 免费证书**

```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx -y  # Ubuntu/Debian
# 或
sudo yum install certbot python3-certbot-nginx -y  # CentOS

# 获取证书（需要先配置好域名解析）
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# 自动续期测试
sudo certbot renew --dry-run
```

**方法 B：使用阿里云 SSL 证书**

1. 在阿里云控制台申请免费 SSL 证书
2. 下载证书文件（Nginx 版本）
3. 上传证书到服务器：`/etc/nginx/ssl/`
4. 修改 Nginx 配置：

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;  # HTTP 重定向到 HTTPS
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/nginx/ssl/your-cert.pem;
    ssl_certificate_key /etc/nginx/ssl/your-key.key;
    
    # SSL 配置优化
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    root /var/www/html;
    index index.html;
    
    # ... 其他配置同上
}
```

##### 验证部署

1. 访问网站：`http://your-domain.com` 或 `http://your-ecs-ip`
2. 检查所有页面是否正常：
   - 首页：`/index.html`
   - 关于页：`/about.html`
   - 隐私政策：`/privacy.html`
3. 检查 Universal Links：`/.well-known/apple-app-site-association`

##### 常见问题

1. **403 Forbidden 错误**
   - 检查文件权限：`sudo chmod -R 755 /var/www/html`
   - 检查 Nginx 用户权限：确保 `www-data` (Ubuntu) 或 `nginx` (CentOS) 有读取权限

2. **无法访问网站**
   - 检查安全组是否开放 80/443 端口
   - 检查防火墙设置
   - 检查 Nginx 是否运行：`sudo systemctl status nginx`

3. **静态资源加载失败**
   - 检查文件路径是否正确
   - 检查 Nginx 配置文件中的 `root` 路径

4. **更新网站内容**
   ```bash
   # 上传新文件后，无需重启 Nginx
   # 如需清除浏览器缓存，可以在文件名后加版本号
   ```

##### 文件权限建议

```bash
# 设置合适的文件权限
sudo chown -R www-data:www-data /var/www/html  # Ubuntu
sudo chown -R nginx:nginx /var/www/html         # CentOS
sudo chmod -R 755 /var/www/html
sudo find /var/www/html -type f -exec chmod 644 {} \;
sudo find /var/www/html -type d -exec chmod 755 {} \;
```

#### 方式五：其他静态托管服务

任何支持静态网站的托管服务都可以使用，只需要：
- 上传网站目录下的所有文件
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
- `docs/README.md` - 项目说明
- `docs/DEPLOY.md` - 本部署指南
- `docs/GITHUB_SSH_SETUP.md` - GitHub SSH 密钥配置指南
- `docs/UNIVERSAL_LINKS_SETUP.md` - Universal Links 配置指南

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

