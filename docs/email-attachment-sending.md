# 使用 AI 发送带附件邮件的经验总结

## 场景
通过 OpenClaw 使用 163 邮箱发送带图片附件的邮件到工作邮箱。

## 遇到的问题

### 1. curl 发送邮件附件失败
- **现象**: curl 命令发送邮件时，服务器连接中断 (`schannel: server closed abruptly`)
- **原因**: 163 SMTP 服务器对 curl 的大文件上传支持不稳定，容易超时或中断
- **尝试**: 多次重试、调整超时时间、使用不同 boundary 格式均失败

### 2. PowerShell .NET Mail 发送失败
- **现象**: `System.Net.Mail` 程序集加载失败，SMTP 连接异常
- **原因**: 
  - PowerShell 5.1 中 `System.Net.Mail` 不是独立的程序集
  - 163 邮箱需要 SSL/TLS 特殊配置
  - 端口 465 (SSL) 和 587 (STARTTLS) 行为不一致

### 3. 最终解决方案: Python smtplib
```python
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email import encoders

# 关键配置
smtp_server = 'smtp.163.com'
smtp_port = 465  # SSL 端口
sender_email = 'hanjianhua44@163.com'
sender_password = '授权码'  # 不是登录密码

# 使用 SMTP_SSL 而不是 STARTTLS
server = smtplib.SMTP_SSL(smtp_server, smtp_port)
server.login(sender_email, sender_password)
server.sendmail(sender_email, receiver_email, msg.as_string())
```

## 关键经验

### 163 邮箱配置
| 项目 | 值 |
|------|-----|
| SMTP 服务器 | smtp.163.com |
| SSL 端口 | 465 |
| 认证方式 | 授权码（非登录密码）|
| 加密方式 | SSL |

### 发送附件要点
1. **使用 MIMEMultipart**: 支持多部分邮件（正文+附件）
2. **正确编码附件**: 使用 base64 编码二进制文件
3. **设置 Content-Disposition**: `attachment; filename="xxx"`
4. **文件路径**: 使用绝对路径或正确展开用户目录

### 工具选择优先级
1. **Python smtplib** ✅ 最稳定，支持复杂邮件格式
2. **curl** ⚠️ 适合简单邮件，大附件容易失败
3. **PowerShell** ❌ 需要 .NET 支持，配置复杂

## 完整代码示例

```python
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email import encoders
import os

# 邮件配置
smtp_server = 'smtp.163.com'
smtp_port = 465
sender_email = 'hanjianhua44@163.com'
sender_password = 'PXPE2Cd7FGEhhwdU'  # 163 授权码
receiver_email = 'hanjianhua4@huawei.com'

# 创建邮件
msg = MIMEMultipart()
msg['From'] = sender_email
msg['To'] = receiver_email
msg['Subject'] = '图片 - tmp.jpg'

# 邮件正文
body = '''建华，

附件是图片：tmp.jpg

发件人：东海龙王'''
msg.attach(MIMEText(body, 'plain', 'utf-8'))

# 添加附件
file_path = os.path.expanduser('~/Documents/tmp/tmp.jpg')
with open(file_path, 'rb') as f:
    attachment = MIMEBase('application', 'octet-stream')
    attachment.set_payload(f.read())
    encoders.encode_base64(attachment)
    attachment.add_header('Content-Disposition', 'attachment', filename='tmp.jpg')
    msg.attach(attachment)

# 发送邮件
server = smtplib.SMTP_SSL(smtp_server, smtp_port)
server.login(sender_email, sender_password)
server.sendmail(sender_email, receiver_email, msg.as_string())
server.quit()
```

## 注意事项
- 163 邮箱需要开启 SMTP 服务并获取授权码
- 附件大小受限于邮箱服务商限制（通常 25MB-50MB）
- 中文附件名需要额外处理编码，建议使用英文文件名

---
记录时间: 2026-03-09
