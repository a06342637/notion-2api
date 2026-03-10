# 小龙虾 AI (OpenClaw) Windows 完整部署教程

## 搭配 notion-2api 使用，免费获得 Claude / GPT / Gemini

本教程教你在 Windows 上部署小龙虾 AI (OpenClaw)，并连接 notion-2api 作为 AI 后端，利用 Notion AI 订阅免费使用 Claude、GPT、Gemini 等模型。

---

## 一、前置准备

### 1. 安装 WSL2（Windows 子系统 Linux）

> OpenClaw 官方推荐使用 WSL2，原生 Windows 兼容性较差。

以**管理员身份**打开 PowerShell，执行：

```powershell
wsl --install
```

安装完成后**重启电脑**，重启后会自动弹出 Ubuntu 窗口，按提示设置用户名和密码。

### 2. 安装 Node.js 22+

在 WSL 终端中执行：

```bash
curl -fsSL https://fnm.vercel.app/install | bash
source ~/.bashrc
fnm install 22
fnm use 22
node -v  # 确认显示 v22.x.x
```

### 3. 安装 Git

```bash
sudo apt update && sudo apt install -y git
```

---

## 二、部署 notion-2api（AI 后端）

### 方案 A：在远程服务器上部署（推荐）

如果你已有 Linux 服务器（如阿里云、腾讯云），在服务器上执行：

```bash
curl -fsSL https://get.docker.com | sh
git clone https://github.com/a06342637/notion-2api.git
cd notion-2api
cp .env.example .env
nano .env  # 填入你的 Notion 凭证
docker compose up -d --build
```

国内服务器需先配 Docker 镜像加速：

```bash
mkdir -p /etc/docker
cat > /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": ["https://docker.1ms.run","https://docker.xuanyuan.me"]
}
EOF
systemctl daemon-reload && systemctl restart docker
```

部署成功后，API 地址为 `http://你的服务器IP:8088`

### 方案 B：在本地 WSL 中部署

```bash
# 安装 Docker
sudo apt update && sudo apt install -y docker.io docker-compose-v2
sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker

# 部署 notion-2api
git clone https://github.com/a06342637/notion-2api.git
cd notion-2api
cp .env.example .env
nano .env  # 填入你的 Notion 凭证
docker compose up -d --build
```

本地部署的 API 地址为 `http://localhost:8088`

### 获取 Notion 凭证

`.env` 文件中需要填写以下信息：

| 字段 | 获取方法 |
|------|---------|
| `NOTION_COOKIE` | 浏览器 F12 → Application → Cookies → `token_v2` 的值 |
| `NOTION_SPACE_ID` | 浏览器 F12 → Network → 找任意请求头中的 `x-notion-space-id` |
| `NOTION_USER_ID` | 浏览器 F12 → Network → 找请求头中的 `x-notion-active-user-header` |
| `NOTION_USER_NAME` | Notion 左上角显示的你的名字 |
| `NOTION_USER_EMAIL` | 你的 Notion 登录邮箱 |
| `API_MASTER_KEY` | 自己设一个密钥，如 `my-secret-key`（设为 `1` 则无需密钥） |

验证部署：

```bash
curl http://localhost:8088/
# 应返回: {"message":"欢迎来到 notion-2api v4.0.0..."}
```

---

## 三、安装小龙虾 AI (OpenClaw)

在 WSL 终端中执行：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

如果 GitHub 下载慢，可以先配置代理或使用镜像：

```bash
npm config set registry https://registry.npmmirror.com/
```

安装完成后运行引导向导：

```bash
openclaw onboard --install-daemon
```

---

## 四、配置 OpenClaw 连接 notion-2api

引导向导中选择 AI 模型提供商时，选择 **OpenAI Compatible**。

或者直接编辑配置文件 `~/.openclaw/openclaw.json`，在 `models.providers` 中添加：

```json
{
  "models": {
    "providers": {
      "mode": "merge",
      "list": [
        {
          "name": "notion-ai",
          "api": "openai-completions",
          "baseUrl": "http://你的服务器IP:8088/v1",
          "apiKey": "你的API_MASTER_KEY",
          "models": [
            "claude-sonnet-4.5",
            "gpt-5",
            "claude-opus-4.1",
            "gpt-4.1"
          ]
        }
      ]
    }
  }
}
```

> 如果 `API_MASTER_KEY` 设为 `1`，则 `apiKey` 随便填一个即可。

### 可用模型列表

| 模型名称 | 说明 |
|---------|------|
| `claude-sonnet-4.5` | Claude Sonnet 4.5（默认，推荐） |
| `gpt-5` | GPT-5 |
| `claude-opus-4.1` | Claude Opus 4.1 |
| `gpt-4.1` | GPT-4.1 |

---

## 五、启动使用

```bash
# 启动网关
openclaw gateway

# 另开一个终端，启动控制面板
openclaw dashboard
```

在浏览器中打开 `http://localhost:3000`，即可开始使用。

---

## 六、常见问题

### PowerShell 脚本执行被禁止

以管理员身份打开 PowerShell 执行：

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### WSL 中 Docker 启动失败

```bash
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo service docker restart
```

### GitHub 克隆超时

使用镜像地址克隆：

```bash
git clone https://ghproxy.net/https://github.com/a06342637/notion-2api.git
```

### notion-2api 504 超时

长回答可能触发 Cloudflare 超时。确保使用最新版代码（已修复）：

```bash
cd notion-2api
git pull origin main
docker compose up -d --build
```

### 检查 OpenClaw 健康状态

```bash
openclaw doctor
```

---

## 七、进阶玩法

- **Telegram 联动**：`openclaw configure` 中添加 Telegram Bot Token，随时随地用手机聊天
- **飞书/钉钉联动**：配置企业机器人，实现团队共享 AI 助手
- **文件处理**：让 AI 帮你分析文档、生成报告
- **定时任务**：配置自动化工作流，如每天早上自动生成新闻摘要
