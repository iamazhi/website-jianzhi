# GitHub SSH 密钥配置指南

本指南将帮助您在新的 ECS 服务器上配置 GitHub SSH 密钥，以便使用 Git 克隆私有仓库或进行推送操作。

## 问题说明

当您在服务器上使用 Git 克隆私有仓库或进行推送时，如果出现以下错误：

```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

这通常是因为服务器上没有配置 GitHub SSH 密钥。

## 配置步骤

### 方法一：生成新的 SSH 密钥（推荐）

适用于全新的服务器，还没有配置过任何 SSH 密钥的情况。

#### 1. 生成 SSH 密钥对

在 ECS 服务器上执行：

```bash
# 生成密钥（替换为你的 GitHub 邮箱）
ssh-keygen -t ed25519 -C "chencongzhi520@gmail.com"

# 如果系统不支持 ed25519，使用 RSA
# ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**交互提示说明**：
- **Enter file in which to save the key**：直接按回车，使用默认路径 `~/.ssh/id_ed25519` 或 `~/.ssh/id_rsa`
- **Enter passphrase**：可以设置密码（推荐）或直接回车留空（方便但安全性稍低）
- **Enter same passphrase again**：确认密码

#### 2. 启动 SSH 代理并添加密钥

```bash
# 启动 ssh-agent
eval "$(ssh-agent -s)"

# 添加密钥到 ssh-agent
# 如果使用的是 ed25519
ssh-add ~/.ssh/id_ed25519

# 如果使用的是 rsa
# ssh-add ~/.ssh/id_rsa
```

#### 3. 查看公钥内容

```bash
# 查看 ed25519 公钥
cat ~/.ssh/id_ed25519.pub

# 或查看 rsa 公钥
# cat ~/.ssh/id_rsa.pub
```

复制输出的全部内容（从 `ssh-ed25519` 或 `ssh-rsa` 开始到邮箱结束）。

#### 4. 在 GitHub 上添加 SSH 密钥

1. 登录 GitHub
2. 点击右上角头像 > **Settings**（设置）
3. 左侧菜单选择 **SSH and GPG keys**（SSH 和 GPG 密钥）
4. 点击 **New SSH key**（新建 SSH 密钥）
5. 填写信息：
   - **Title**：给密钥起个名字，如 "ECS Server" 或 "阿里云服务器"
   - **Key type**：选择 **Authentication Key**
   - **Key**：粘贴刚才复制的公钥内容
6. 点击 **Add SSH key**（添加 SSH 密钥）
7. 如果需要，输入 GitHub 密码确认

#### 5. 测试连接

```bash
ssh -T git@github.com
```

如果配置成功，你会看到类似以下输出：

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

这表示 SSH 密钥配置成功！

#### 6. 克隆仓库

现在可以使用 SSH 方式克隆仓库了：

```bash
# 使用 SSH 地址克隆（不是 HTTPS）
git clone git@github.com:username/repository-name.git
```

### 方法二：使用已有的 SSH 密钥

如果你在本地电脑已经有配置好的 SSH 密钥，可以直接复制到服务器。

#### 1. 在本地电脑查看密钥

```bash
# 查看是否有密钥文件
ls -la ~/.ssh/

# 查看公钥内容
cat ~/.ssh/id_ed25519.pub
# 或
cat ~/.ssh/id_rsa.pub
```

#### 2. 复制密钥到服务器

**方法 A：使用 SCP 复制密钥对**

```bash
# 在本地电脑执行（复制私钥和公钥）
scp ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub root@your-ecs-ip:~/.ssh/

# 或使用 rsa
# scp ~/.ssh/id_rsa ~/.ssh/id_rsa.pub root@your-ecs-ip:~/.ssh/
```

**方法 B：手动复制公钥内容**

1. 在本地查看公钥：`cat ~/.ssh/id_ed25519.pub`
2. 复制输出内容
3. 在服务器上创建文件：

```bash
# 在服务器上执行
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/id_ed25519.pub
# 粘贴公钥内容，保存退出（Ctrl+X, Y, Enter）
```

4. 在本地复制私钥内容并粘贴到服务器（**注意：私钥非常敏感，确保传输安全**）

#### 3. 设置正确的文件权限

```bash
# 在服务器上执行
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519    # 私钥权限必须是 600
chmod 644 ~/.ssh/id_ed25519.pub # 公钥权限
```

#### 4. 添加到 ssh-agent 并测试

```bash
# 启动 ssh-agent
eval "$(ssh-agent -s)"

# 添加密钥
ssh-add ~/.ssh/id_ed25519

# 测试连接
ssh -T git@github.com
```

### 方法三：使用 HTTPS 方式（临时方案）

如果暂时不想配置 SSH 密钥，可以使用 HTTPS 方式克隆，但推送时需要输入用户名和密码（或使用 Personal Access Token）。

```bash
# 使用 HTTPS 地址克隆
git clone https://github.com/username/repository-name.git

# 推送时需要输入用户名和密码
# 如果启用了两步验证，需要使用 Personal Access Token 作为密码
```

**注意**：HTTPS 方式每次推送都需要输入凭证，不适合自动化部署。

## 配置多个 GitHub 账户（可选）

如果你的服务器需要访问多个 GitHub 账户的仓库，可以配置 SSH 配置文件。

### 1. 创建多个密钥

```bash
# 生成第一个账户的密钥
ssh-keygen -t ed25519 -C "account1@example.com" -f ~/.ssh/id_ed25519_account1

# 生成第二个账户的密钥
ssh-keygen -t ed25519 -C "account2@example.com" -f ~/.ssh/id_ed25519_account2
```

### 2. 配置 SSH config

编辑 `~/.ssh/config`：

```bash
nano ~/.ssh/config
```

添加配置：

```
# 第一个 GitHub 账户
Host github.com-account1
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_account1

# 第二个 GitHub 账户
Host github.com-account2
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_account2
```

### 3. 使用不同的 Host 克隆

```bash
# 使用第一个账户
git clone git@github.com-account1:username1/repo1.git

# 使用第二个账户
git clone git@github.com-account2:username2/repo2.git
```

## 常见问题

### 1. `Permission denied (publickey)` 错误

**可能原因**：
- 密钥未添加到 GitHub
- 密钥文件权限不正确
- SSH 代理未运行或未添加密钥

**解决方法**：
```bash
# 检查密钥是否存在
ls -la ~/.ssh/

# 检查文件权限（私钥必须是 600）
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# 重新添加密钥到 ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 测试连接
ssh -T git@github.com
```

### 2. `Bad permissions` 错误

```bash
# 设置正确的权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### 3. SSH 代理未运行

```bash
# 启动 ssh-agent
eval "$(ssh-agent -s)"

# 添加密钥
ssh-add ~/.ssh/id_ed25519

# 查看已添加的密钥
ssh-add -l
```

### 4. 每次重启服务器后需要重新添加密钥

**临时解决方案**：每次 SSH 登录后执行

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**永久解决方案**：将以下内容添加到 `~/.bashrc` 或 `~/.bash_profile`：

```bash
# 自动启动 ssh-agent 并添加密钥
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519
fi
```

然后执行：
```bash
source ~/.bashrc
```

### 5. 测试时提示 "Host key verification failed"

```bash
# 清除已知主机记录
ssh-keygen -R github.com

# 重新测试
ssh -T git@github.com
# 输入 yes 接受新的主机密钥
```

## 安全建议

1. **使用密码保护私钥**：生成密钥时设置 passphrase
2. **限制私钥权限**：确保私钥文件权限为 600，只有所有者可读
3. **定期轮换密钥**：建议每年更换一次 SSH 密钥
4. **使用 ed25519 算法**：比 RSA 更安全且更快（如果系统支持）
5. **不要共享私钥**：每个服务器使用独立的密钥对

## 验证配置

完成配置后，执行以下命令验证：

```bash
# 1. 检查密钥文件是否存在
ls -la ~/.ssh/

# 2. 检查权限是否正确
stat -c "%a %n" ~/.ssh/id_ed25519  # 应该显示 600
stat -c "%a %n" ~/.ssh/id_ed25519.pub  # 应该显示 644

# 3. 测试 GitHub 连接
ssh -T git@github.com

# 4. 尝试克隆仓库（替换为你的仓库地址）
git clone git@github.com:username/repository-name.git
```

## 参考链接

- [GitHub SSH 密钥文档](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [生成新 SSH 密钥](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [添加 SSH 密钥到 GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
