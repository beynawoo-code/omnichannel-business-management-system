# GitHub 使用流程指南

> 本文档记录了将本地项目文件推送到 GitHub 并生成外部访问链接的完整操作流程。

---

## 一、前置准备

### 1. 检查本地环境

```bash
# 检查 git 是否安装
git --version

# 检查是否已有 SSH 密钥
ls ~/.ssh/*.pub
```

### 2. 配置 SSH 密钥（首次使用）

若没有 SSH 密钥，执行以下命令生成：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

查看公钥内容：

```bash
cat ~/.ssh/id_ed25519.pub
```

将输出的公钥内容添加到 GitHub：
1. 打开 [github.com/settings/ssh/new](https://github.com/settings/ssh/new)
2. Title 填写设备名称（如 `MacBook`）
3. Key 粘贴上方公钥内容
4. 点击 **Add SSH key**

> 若提示 "Key is already in use"，说明该密钥已绑定到账号，无需重复添加。

### 3. 验证 SSH 连接

```bash
ssh -T git@github.com
# 成功输出：Hi username! You've successfully authenticated...
```

---

## 二、创建 GitHub 仓库

1. 打开 [github.com/new](https://github.com/new)
2. 填写仓库名称
3. 选择 **Public**（公开）或 **Private**（私有）
4. **不要**勾选初始化 README（保持空仓库）
5. 点击 **Create repository**

---

## 三、本地项目初始化与推送

### 1. 初始化本地 Git 仓库

```bash
cd /your/project/path
git init
```

### 2. 关联远程仓库（使用 SSH 地址）

```bash
git remote add origin git@github.com:用户名/仓库名.git

# 验证关联是否成功
git remote -v
```

> 推荐使用 SSH 地址而非 HTTPS，避免每次推送都需要输入账号密码。
> 若已用 HTTPS 添加，可用以下命令切换为 SSH：
> ```bash
> git remote set-url origin git@github.com:用户名/仓库名.git
> ```

### 3. 添加文件并提交

```bash
# 添加指定文件
git add path/to/your/file.html

# 或添加全部文件
git add .

# 查看暂存状态
git status

# 提交
git commit -m "提交说明"
```

### 4. 推送到 GitHub

```bash
git push -u origin main
```

> `-u` 参数表示将本地 `main` 分支与远程关联，后续推送只需 `git push`。

---

## 四、日常更新文件流程

每次修改文件后，执行以下命令更新到 GitHub：

```bash
git add .
git commit -m "描述本次修改内容"
git push
```

---

## 五、分享 HTML 文件的外部访问链接

### 方式一：htmlpreview（推荐，无需配置）

将以下 URL 格式发给对方，对方可直接在浏览器中查看渲染后的页面：

```
https://htmlpreview.github.io/?https://github.com/用户名/仓库名/blob/main/文件路径.html
```

**示例：**
```
https://htmlpreview.github.io/?https://github.com/beynawoo-code/omnichannel-business-management-system/blob/main/src/features/omnichannel-business-management-system/docs/guides/system_architecture_diagram.html
```

### 方式二：GitHub Pages（适合长期对外展示）

1. 打开仓库 → **Settings** → 左侧 **Pages**
2. Source 选择 **Deploy from a branch**
3. Branch 选 **main**，目录选 **/ (root)**
4. 点击 **Save**，等待约 1~2 分钟部署完成

访问地址格式：
```
https://用户名.github.io/仓库名/文件路径.html
```

### 方式三：本地临时共享（localtunnel）

适合临时分享，关机后链接失效：

```bash
# 在文件所在目录启动 HTTP 服务
python3 -m http.server 8766 &

# 生成外网隧道链接
npx localtunnel --port 8766
```

输出的 `https://xxx.loca.lt` 即为可分享的外部链接，首次访问需点击页面上的 **Click to Continue**。

---

## 六、常见问题

| 问题 | 原因 | 解决方法 |
|------|------|----------|
| `fatal: could not read Username` | 使用了 HTTPS 地址但未配置凭据 | 切换为 SSH 地址 |
| `Key is already in use` | 该 SSH 公钥已绑定到 GitHub 账号 | 无需重新添加，直接使用即可 |
| `Address already in use` | 端口被占用 | 换一个端口号，如 `8767` |
| `Permission denied (publickey)` | SSH 密钥未添加到 GitHub | 检查 `~/.ssh/id_ed25519.pub` 并添加到 GitHub |

---

*最后更新：2026-04-21*
