# 用 AI 工具自动发送邮件

## 背景

在 Cursor / OpenClaw 中完成了文件生成后，经常需要把结果发给同事或自己。与其手动打开邮箱、写正文、添附件，不如让 AI 直接帮你发出去。

通过 Cursor + Claude Opus 或 OpenClaw，可以用 Python SMTP 脚本一键发送带附件的邮件。

## 前置准备

### 环境

- **Cursor** 编辑器（配置 Claude Opus 模型）
- **Python 3.x**（标准库自带 `smtplib`，无需额外安装）

### 开启邮箱 SMTP 服务

以 163 邮箱为例：

1. 登录网页版 [163 邮箱](https://mail.163.com)
2. 进入 **设置** → **POP3/SMTP/IMAP**
3. 开启 **SMTP 服务**
4. 按提示完成手机验证，获取**授权码**（一串字母，不是登录密码）

> 其他邮箱的 SMTP 配置：
> | 邮箱 | SMTP 服务器 | 端口 |
> |------|------------|------|
> | 163 | smtp.163.com | 465 (SSL) |
> | QQ | smtp.qq.com | 465 (SSL) |
> | Gmail | smtp.gmail.com | 587 (TLS) |
> | Outlook | smtp.office365.com | 587 (TLS) |

## 操作流程

### 第一步：告诉 Opus 你要发邮件

在 Cursor 中直接说：

> "帮我把刚刚生成的 PPT 通过邮件发给 xxx@company.com"

Opus 会询问你所需的 SMTP 配置信息（如果之前没提供过的话）。

### 第二步：AI 生成发送脚本

Opus 会自动生成完整的 Python 邮件发送脚本，包括：
- SMTP 连接与认证
- 邮件正文构建
- 附件编码与挂载
- 发送与错误处理

### 第三步：确认并执行

AI 生成脚本后会直接运行，发送成功会有提示。

## 关键代码解析

### 基本发送框架

```python
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email import encoders
import os

SMTP_SERVER = "smtp.163.com"
SMTP_PORT = 465
SENDER = "your_email@163.com"
AUTH_CODE = "your_auth_code"       # SMTP 授权码，非登录密码
RECIPIENT = "recipient@company.com"
```

### 构建邮件

```python
msg = MIMEMultipart()
msg["From"] = SENDER
msg["To"] = RECIPIENT
msg["Subject"] = "邮件主题"

# 正文
msg.attach(MIMEText("邮件正文内容", "plain", "utf-8"))
```

### 添加附件

```python
attachment_path = "path/to/file.pptx"

with open(attachment_path, "rb") as f:
    part = MIMEBase(
        "application",
        "vnd.openxmlformats-officedocument.presentationml.presentation"
    )
    part.set_payload(f.read())
    encoders.encode_base64(part)
    part.add_header(
        "Content-Disposition",
        f'attachment; filename="{os.path.basename(attachment_path)}"'
    )
    msg.attach(part)
```

### 发送

```python
server = smtplib.SMTP_SSL(SMTP_SERVER, SMTP_PORT)
server.login(SENDER, AUTH_CODE)
server.sendmail(SENDER, RECIPIENT, msg.as_string())
server.quit()
print("发送成功！")
```

## 进阶用法

### 发给多个收件人

```python
recipients = ["a@company.com", "b@company.com"]
msg["To"] = ", ".join(recipients)
server.sendmail(SENDER, recipients, msg.as_string())
```

### 发送 HTML 格式正文

```python
html_body = """
<h2>项目周报</h2>
<p>详见附件。</p>
"""
msg.attach(MIMEText(html_body, "html", "utf-8"))
```

### 多个附件

```python
for filepath in ["report.pptx", "data.xlsx", "summary.pdf"]:
    with open(filepath, "rb") as f:
        part = MIMEBase("application", "octet-stream")
        part.set_payload(f.read())
        encoders.encode_base64(part)
        part.add_header(
            "Content-Disposition",
            f'attachment; filename="{os.path.basename(filepath)}"'
        )
        msg.attach(part)
```

## 踩坑记录（OpenClaw 实测）

在 OpenClaw 中尝试发送带附件邮件时，依次踩了以下坑，最终确认 **Python smtplib 是最可靠的方案**。

### curl 发送邮件附件失败

- **现象**：curl 命令发送邮件时，服务器连接中断（`schannel: server closed abruptly`）
- **原因**：163 SMTP 服务器对 curl 的大文件上传支持不稳定，容易超时或中断
- **尝试**：多次重试、调整超时时间、使用不同 boundary 格式均失败
- **结论**：curl 适合发简单纯文本邮件，带附件（尤其是图片/大文件）不推荐

### PowerShell .NET Mail 发送失败

- **现象**：`System.Net.Mail` 程序集加载失败，SMTP 连接异常
- **原因**：
  - PowerShell 5.1 中 `System.Net.Mail` 不是独立可加载的程序集
  - 163 邮箱需要 SSL/TLS 特殊配置
  - 端口 465（SSL）和 587（STARTTLS）行为不一致
- **结论**：Windows Server 上用 PowerShell 发邮件坑多，不如直接用 Python

### 工具选择优先级

| 方式 | 推荐度 | 说明 |
|------|--------|------|
| Python smtplib | ✅ 推荐 | 最稳定，支持复杂邮件格式，零额外依赖 |
| curl | ⚠️ 慎用 | 适合简单邮件，大附件容易失败 |
| PowerShell .NET | ❌ 不推荐 | 需要 .NET 支持，在 Server 环境配置复杂 |

### 附件发送要点

1. **使用 MIMEMultipart**：支持多部分邮件（正文 + 附件）
2. **正确编码附件**：使用 base64 编码二进制文件
3. **设置 Content-Disposition**：`attachment; filename="xxx"`
4. **文件路径**：使用绝对路径或 `os.path.expanduser()` 展开用户目录
5. **中文附件名**：建议使用英文文件名避免编码问题

## 安全提醒

- **授权码**不要硬编码在公开仓库的代码中，建议使用环境变量
- SMTP 授权码和登录密码是两回事，授权码仅用于第三方客户端登录
- 如果不再使用，记得在邮箱设置中关闭 SMTP 或重置授权码
- 附件大小受限于邮箱服务商限制（通常 25MB–50MB）

## 小结

- **全程自动化**：从文件生成到邮件发送，无需离开 Cursor / OpenClaw
- **零依赖**：Python 标准库自带 `smtplib`，不需要额外安装
- **可复用**：配置好一次后，后续发送只需一句话
- **适用场景**：发送报告、PPT、数据文件给同事，日常工作流自动化
- **跨工具通用**：Cursor 和 OpenClaw 都可以用同样的 Python 方案
