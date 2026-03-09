# AI Tools Experience

记录使用 AI 工具（Cursor + Claude Opus 等）的实践经验，涵盖代码生成、文档自动化、工作流搭建等场景。

## 目录

- [用 Cursor + Opus 生成专业 PPT](docs/generate-ppt-with-opus.md)
- [用 AI 工具自动发送邮件（含 OpenClaw 踩坑记录）](docs/send-email-with-opus.md)
- [用 Cursor + Opus 阅读论文并生成讲解 PPT](docs/read-paper-generate-ppt.md)
- [用 Cursor + Opus 集成飞书 MCP 实现文档自动化](docs/feishu-mcp-integration.md)
- [用 Cursor 在 Windows 上部署 OpenClaw 并连接飞书](docs/openclaw-feishu-setup.md)
- [使用 OpenClaw + Python 发送带附件邮件](docs/email-attachment-sending.md)
- [Cursor 使用指南](docs/cursor-usage-guide.md)

## 工具环境

| 工具 | 用途 |
|------|------|
| [Cursor](https://cursor.sh/) | AI 驱动的代码编辑器 |
| Claude Opus | Anthropic 的大语言模型，作为 Cursor 的后端 AI |
| [OpenClaw](https://github.com/openclaw/openclaw) | 开源 AI Agent，支持多平台聊天集成 |
| Python | 脚本执行环境 |
| [@larksuiteoapi/lark-mcp](https://github.com/larksuite/lark-openapi-mcp) | 飞书官方 MCP Server，支持文档读写、消息发送等 |

## 生成的文件示例

- [`AI_Pipeline_Proposal.pptx`](files/AI_Pipeline_Proposal.pptx) — 由 AI 生成的完整 PPT 方案
- [`RoboPocket_Paper_Presentation.pptx`](files/RoboPocket_Paper_Presentation.pptx) — 由 AI 阅读论文后生成的带原图讲解 PPT
- [Cursor 集成飞书 MCP Server 教程（飞书文档）](https://aimy7b5ws0.feishu.cn/docx/R12ydDR0Xo3nXoxqLp9cwBDxnWe) — 由 AI 自动生成并上传至飞书

## License

MIT
