# 用 Cursor + Opus 集成飞书 MCP 实现文档自动化

## 概述

通过在 Cursor 中配置飞书官方 MCP Server，可以让 AI 助手直接读写飞书文档、发送消息、操作多维表格、搜索知识库等，实现开发工作流与飞书的无缝集成。

**生成的飞书文档示例：** [Cursor 集成飞书 MCP Server 教程](https://aimy7b5ws0.feishu.cn/docx/R12ydDR0Xo3nXoxqLp9cwBDxnWe)

## 前置准备

### 1. 创建飞书应用

1. 登录 [飞书开放平台](https://open.feishu.cn/)
2. 进入「开发者后台」，点击「创建企业自建应用」
3. 填写应用名称和描述，完成创建
4. 在应用详情页获取 **App ID** 和 **App Secret**

### 2. 申请应用权限

在应用的「权限管理」中，根据需要申请以下权限：

| 权限 | 说明 |
|------|------|
| `docx:document` | 读取文档 |
| `docx:document:write` | 写入/创建文档 |
| `drive:drive` | 访问云空间 |
| `im:message` | 发送消息 |
| `im:chat` | 群聊管理 |
| `bitable:app` | 多维表格操作 |
| `wiki:wiki` | 知识库访问 |

### 3. 发布应用

- 在「版本管理与发布」中创建一个新版本
- 提交审核，等待企业管理员审批通过

## 安装配置

### 配置 Cursor MCP

编辑工作区下的 `.cursor/mcp.json` 文件（不存在则新建）：

```json
{
  "mcpServers": {
    "feishu": {
      "command": "npx",
      "args": ["-y", "@larksuiteoapi/lark-mcp", "mcp", "-a", "<你的App ID>", "-s", "<你的App Secret>"]
    }
  }
}
```

将 `<你的App ID>` 和 `<你的App Secret>` 替换为真实凭证。

保存后在 Cursor 设置 > MCP 面板中点击刷新/重启。

### 验证连接

重启 MCP 后，服务器状态显示为「已连接」，并列出 19 个可用工具，说明配置成功。

> **提示：** 首次运行时 npx 需要下载依赖，可能需要等待 1-2 分钟。

## 可用功能一览

成功连接后，共有 **19 个工具**可用，覆盖以下场景：

### 文档操作

| 工具 | 功能 |
|------|------|
| `docx_builtin_import` | 通过 Markdown 创建飞书云文档（最大 20MB） |
| `docx_v1_document_rawContent` | 读取文档纯文本内容 |
| `docx_builtin_search` | 搜索云文档（需用户授权） |

### 消息操作

| 工具 | 功能 |
|------|------|
| `im_v1_message_create` | 发送 P2P / 群聊消息 |
| `im_v1_message_list` | 获取消息列表 |
| `im_v1_chat_list` | 获取群聊列表 |
| `im_v1_chat_create` | 创建群聊 |
| `im_v1_chatMembers_get` | 获取群成员信息 |

### 知识库

| 工具 | 功能 |
|------|------|
| `wiki_v1_node_search` | 搜索知识库节点 |
| `wiki_v2_space_getNode` | 获取知识库节点详情 |

### 多维表格（Bitable）

| 工具 | 功能 |
|------|------|
| `bitable_v1_app_create` | 创建多维表格应用 |
| `bitable_v1_appTable_list` | 列出数据表 |
| `bitable_v1_appTable_create` | 创建数据表 |
| `bitable_v1_appTableField_list` | 列出字段 |
| `bitable_v1_appTableRecord_create` | 创建记录 |
| `bitable_v1_appTableRecord_search` | 搜索记录 |
| `bitable_v1_appTableRecord_update` | 更新记录 |

### 其他

| 工具 | 功能 |
|------|------|
| `contact_v3_user_batchGetId` | 通过邮箱/手机号批量获取用户 ID |
| `drive_v1_permissionMember_create` | 设置云文档权限 |

## 使用示例

### 创建飞书文档

在 Cursor 中告诉 AI：

> 「帮我在飞书创建一篇关于 XX 的文档」

AI 会通过 `docx_builtin_import` 将 Markdown 内容导入为飞书云文档，返回文档链接。

### 读取飞书文档

提供文档 URL 或 document_id：

> 「帮我读取这篇飞书文档的内容：https://xxx.feishu.cn/docx/xxxxx」

AI 通过 `docx_v1_document_rawContent` 获取纯文本内容。

### 发送飞书消息

指定收件人和消息内容：

> 「通过飞书给 xxx@company.com 发一条消息，内容是 ...」

AI 通过 `im_v1_message_create` 发送消息，支持通过邮箱、open_id、chat_id 等方式指定收件人。

### 操作多维表格

> 「帮我在飞书创建一个多维表格，记录项目任务」

AI 可以创建表格、添加字段、插入记录，一条龙完成。

### 搜索知识库

> 「帮我在飞书知识库里搜索关于 XX 的文档」

AI 通过 `wiki_v1_node_search` 搜索知识库内容。

## 常见问题

### 连接不上怎么办？

1. **检查网络**：确保可以正常访问飞书 API（open.feishu.cn）
2. **检查凭证**：确认 App ID 和 App Secret 正确无误
3. **检查权限**：确保应用已发布且权限已审批通过
4. **首次较慢**：npx 首次运行需要下载依赖，耐心等待
5. **手动调试**：在终端运行 npx 命令查看报错信息

### 搜索文档提示权限不足？

`docx_builtin_search` 仅支持 `user_access_token`，需要在调用时设置 `useUAT: true`，并确保已完成用户授权流程。

### 其他 MCP 方案

除了官方 `@larksuiteoapi/lark-mcp`，社区还有其他选择：

| 方案 | GitHub | 特点 |
|------|--------|------|
| Open Feishu MCP | ztxtxwd/open-feishu-mcp-server | OAuth 零配置，支持 Cloudflare 部署 |
| Lark-Tools-MCP | VienLi/lark-tools-mcp | 轻量，支持文档读取和合同审批 |
| MCP Server My Lark Doc | T0UGH/mcp_server_my_lark_doc | 支持 Wiki 文档搜索 |

## 工具环境

| 工具 | 版本/说明 |
|------|-----------|
| [Cursor](https://cursor.sh/) | AI 驱动的代码编辑器 |
| Claude Opus | Anthropic 大语言模型 |
| [@larksuiteoapi/lark-mcp](https://github.com/larksuite/lark-openapi-mcp) | 飞书官方 MCP Server |
| Node.js + npx | MCP Server 运行环境 |
