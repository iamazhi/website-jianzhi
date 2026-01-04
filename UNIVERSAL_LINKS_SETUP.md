# Universal Links 配置完整步骤（GitHub Pages）

## 第一步：修改 apple-app-site-association 文件

1. 打开 `.well-known/apple-app-site-association` 文件
2. 找到这一行：
   ```json
   "appID": "TEAM_ID.com.yourcompany.jiandi",
   ```
3. 替换为你的实际信息：
   - `TEAM_ID`：你的 Apple Developer Team ID（10位字符，如：ABC123DEFG）
   - `com.yourcompany.jiandi`：你的 App Bundle ID（在 Xcode 项目设置中查看）
   
   **示例**：
   ```json
   "appID": "ABC123DEFG.com.manmanyouyou.jiandi",
   ```

## 第二步：提交到 GitHub

```bash
# 添加文件
git add .well-known/apple-app-site-association

# 提交
git commit -m "Add Universal Links configuration"

# 推送到 GitHub
git push
```

## 第三步：验证文件可访问

等待 GitHub Pages 部署完成后（通常1-2分钟），在浏览器访问：

```
https://jiandi.manmanyouyou.cn/.well-known/apple-app-site-association
```

**应该看到**：JSON 格式的内容（不是下载文件）

**如果看到 404**：
- 检查文件路径是否正确：`.well-known/apple-app-site-association`（注意：没有扩展名）
- 确认 GitHub Pages 已启用并部署成功
- 等待几分钟后重试

## 第四步：在微信开放平台填写

在应用平台信息中，Universal Links 填写：

```
https://jiandi.manmanyouyou.cn/app/
```

**或者**：

```
https://jiandi.manmanyouyou.cn/jiandi/
```

（两个路径在配置文件中都已包含，任选一个即可）

## 第五步：在 Xcode 中配置（如果还没配置）

1. 打开 Xcode 项目
2. 选择 Target > Signing & Capabilities
3. 点击 "+ Capability"
4. 添加 "Associated Domains"
5. 添加域名：`applinks:jiandi.manmanyouyou.cn`

## 验证清单

- [ ] `.well-known/apple-app-site-association` 文件已创建
- [ ] 文件中的 `appID` 已替换为实际值
- [ ] 文件已提交并推送到 GitHub
- [ ] 可以通过浏览器访问该文件（返回 JSON 内容）
- [ ] 在微信开放平台填写了 Universal Links 路径
- [ ] Xcode 项目中已配置 Associated Domains

## 常见问题

### Q: 文件返回 404？
A: 
- 确认文件路径是 `.well-known/apple-app-site-association`（没有扩展名）
- 确认文件已推送到 GitHub 主分支
- 等待 GitHub Pages 重新部署（可能需要几分钟）

### Q: 文件被下载而不是显示？
A: GitHub Pages 默认会正确设置 Content-Type，如果被下载，可能需要：
- 检查文件确实是纯文本 JSON，没有 BOM
- 确认文件没有 `.json` 扩展名

### Q: 如何测试 Universal Links？
A: 
- 在 Safari 中长按链接，应该看到"在'见地'中打开"选项
- 或者使用 Apple 的验证工具：https://search.developer.apple.com/appsearch-validation-tool/

## 重要提示

1. **文件必须没有扩展名**：`apple-app-site-association`（不是 `.json`）
2. **路径必须正确**：`.well-known/apple-app-site-association`
3. **必须通过 HTTPS 访问**：确保域名有 SSL 证书（GitHub Pages 自动提供）
4. **App ID 格式**：`TEAM_ID.BUNDLE_ID`（中间是点，不是斜杠）

