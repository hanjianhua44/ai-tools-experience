# 用 Cursor 在 Windows 上部署 OpenClaw 并连接飞书

在 Windows Server 上通过 Cursor AI 助手完成 OpenClaw 安装、配置 Kimi K2.5 Code 模型，并连接飞书机器人实现 AI 对话。

## 环境要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows 10/11 或 Windows Server（64 位） |
| Node.js | v22+（LTS 推荐） |
| Git | 已安装并可用 |
| 内存 | 2-4 GB 最低 |
| 磁盘 | ~500 MB |

## 一、安装 OpenClaw

### 1. 确认环境

```powershell
node --version   # 需要 v22+
git --version    # 需要已安装
```

> 如果 `git` 命令找不到但已安装，需要手动加入 PATH：
> ```powershell
> $env:PATH = "C:\Program Files\Git\cmd;" + $env:PATH
> ```

### 2. 全局安装

```powershell
npm install -g openclaw --ignore-scripts
```

> `--ignore-scripts` 跳过 `node-llama-cpp` 本地模型编译。如果只用 API 模型（Kimi、Claude 等），不影响使用。

### 3. 验证安装

```powershell
openclaw --version
# 输出: 2026.3.7
```

### 4. 升级到最新版

```powershell
# 如果遇到 SSH 权限问题，先配置 Git 走 HTTPS
git config --global url."https://github.com/".insteadOf "ssh://git@github.com/"
git config --global url."https://github.com/".insteadOf "git@github.com:"

npm install -g openclaw@latest --ignore-scripts
```

## 二、配置 Kimi K2.5 Code 模型

### 1. 获取 API Key

前往 [Kimi Code 控制台](https://www.kimi.com/code/console) 注册并获取 API Key。

### 2. 运行首次配置向导

```powershell
openclaw onboard
```

在交互式向导中：
- Gateway 运行位置选择 **Local**
- 认证方式选择 **kimi-code-api-key**
- 输入你的 Kimi API Key

### 3. 确认配置文件

配置文件位于 `~\.openclaw\openclaw.json`，核心内容如下：

```json
{
  "models": {
    "providers": {
      "kimi-coding": {
        "baseUrl": "https://api.kimi.com/coding/",
        "api": "anthropic-messages",
        "models": [{
          "id": "k2p5",
          "name": "Kimi for Coding",
          "reasoning": true,
          "contextWindow": 262144,
          "maxTokens": 32768
        }]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": { "primary": "kimi-coding/k2p5" }
    }
  }
}
```

### 4. 配置网络搜索

OpenClaw 2026.3.7 支持的搜索提供商：`brave`、`perplexity`、`grok`、`gemini`、`kimi`。

已有 Kimi 账号可直接使用 Kimi 搜索（免费）：

```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "kimi"
      }
    }
  }
}
```

## 三、启动 Gateway

### 前台运行

```powershell
openclaw gateway --port 18789
```

### 打开 Dashboard

```powershell
openclaw dashboard
```

Dashboard 地址：`http://127.0.0.1:18789`

### 健康检查

```powershell
openclaw health
```

## 四、连接飞书

### 1. 创建飞书应用

1. 登录 [飞书开放平台](https://open.feishu.cn/app)，点击「创建企业自建应用」
2. 记下 **App ID**（`cli_xxxx` 格式）和 **App Secret**

### 2. 开启机器人能力

「应用能力」→「添加应用能力」→ 找到「机器人」→ 点击「添加能力」

### 3. 配置事件订阅（WebSocket 长连接）

1. 左侧菜单「事件与回调」
2. 订阅方式选择 **「使用长连接接收事件」**
3. 添加事件：`im.message.receive_v1`（接收消息）

> 此时页面会提示「未检测到应用连接信息」，这是正常的，需要先在 OpenClaw 里配好再回来保存。

### 4. 开通权限

在「权限管理」中搜索并开通：

| 权限 | 说明 |
|------|------|
| `im:message` | 获取与发送消息 |
| `im:message:send_as_bot` | 以机器人身份发送消息 |
| `im:message.p2p_msg` | 读取用户发给机器人的单聊消息 |
| `im:message.group_at_msg` | 接收群聊中 @机器人消息 |

### 5. 发布应用

「版本管理与发布」→「创建版本」→「申请发布」→ 管理员审批通过

> **不发布的话事件订阅不会生效。**

### 6. 安装飞书 SDK 依赖

OpenClaw 的飞书插件依赖 `@larksuiteoapi/node-sdk`，如果安装时使用了 `--ignore-scripts`，需要手动补装：

```powershell
npm install @larksuiteoapi/node-sdk --prefix "$env:APPDATA\npm\node_modules\openclaw"
```

### 7. 配置 OpenClaw 飞书频道

编辑 `~\.openclaw\openclaw.json`，添加 `channels` 部分：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "dmPolicy": "pairing",
      "accounts": {
        "main": {
          "appId": "<你的飞书 App ID>",
          "appSecret": "<你的飞书 App Secret>",
          "botName": "OpenClaw AI"
        }
      }
    }
  }
}
```

### 8. 重启 Gateway

```powershell
# 停止旧进程（找到占用 18789 端口的 PID 并 kill）
netstat -ano | findstr "18789.*LISTENING"
Stop-Process -Id <PID> -Force

# 重新启动
openclaw gateway --port 18789
```

启动日志中看到以下内容表示飞书连接成功：

```
[feishu] feishu[main]: bot open_id resolved: ou_xxxx
[feishu] feishu[main]: WebSocket client started
[ws] ws client ready
```

### 9. 回飞书保存配置

Gateway 启动后，回到飞书开放平台的「事件订阅」页面刷新，检测到连接后点击保存。

### 10. 配对确认

在飞书里找到机器人并发送第一条消息，机器人会返回配对码。运行：

```powershell
openclaw pairing approve feishu <配对码>
```

配对成功后，再次发消息即可正常对话。

## 五、最终效果

| 项目 | 状态 |
|------|------|
| OpenClaw | v2026.3.7 |
| AI 模型 | Kimi K2.5 Code（免费） |
| 搜索引擎 | Kimi（免费） |
| 飞书频道 | WebSocket 长连接 |
| Dashboard | http://127.0.0.1:18789 |

在飞书中与机器人对话，即可享受 AI 编程助手的能力：代码生成、问题解答、文档搜索等。

## 六、常见问题

### Q: `npm install -g openclaw` 报 git 找不到？
将 Git 加入 PATH：`$env:PATH = "C:\Program Files\Git\cmd;" + $env:PATH`

### Q: 安装时 `node-llama-cpp` 编译失败？
加 `--ignore-scripts` 跳过。只影响本地模型，不影响 API 模型使用。

### Q: 升级时报 SSH 权限问题？
配置 Git 走 HTTPS：
```powershell
git config --global url."https://github.com/".insteadOf "ssh://git@github.com/"
```

### Q: 飞书插件报 `Cannot find module '@larksuiteoapi/node-sdk'`？
手动安装：`npm install @larksuiteoapi/node-sdk --prefix "$env:APPDATA\npm\node_modules\openclaw"`

### Q: Gateway 关闭后飞书断连？
Gateway 需要持续运行。如需后台常驻，可考虑注册为 Windows 服务或使用 `nssm` 工具。

## 工具版本

| 工具 | 版本 |
|------|------|
| Cursor | AI 代码编辑器 |
| OpenClaw | 2026.3.7 |
| Kimi K2.5 Code | k2p5 |
| Node.js | v24.14.0 |
| 飞书 SDK | @larksuiteoapi/node-sdk |
