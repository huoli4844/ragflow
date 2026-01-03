# SMTP 安全配置

<cite>
**本文档引用的文件**
- [service_conf.yaml](file://conf/service_conf.yaml)
- [email.py](file://agent/tools/email.py)
- [user_app.py](file://api/apps/user_app.py)
- [email_templates.py](file://api/utils/email_templates.py)
- [__init__.py](file://api/apps/__init__.py)
</cite>

## 目录
1. [简介](#简介)
2. [SMTP 配置参数详解](#smtp-配置参数详解)
3. [安全邮件传输配置](#安全邮件传输配置)
4. [安全相关邮件功能](#安全相关邮件功能)
5. [认证最佳实践](#认证最佳实践)
6. [主流邮件服务集成](#主流邮件服务集成)
7. [常见连接问题解决方案](#常见连接问题解决方案)
8. [防止滥用措施](#防止滥用措施)
9. [结论](#结论)

## 简介

RAGFlow 系统通过 SMTP 协议实现安全的邮件传输功能，主要用于发送密码重置、账户验证等安全相关邮件。系统通过 `service_conf.yaml` 配置文件中的 SMTP 参数进行配置，并结合 TLS/SSL 加密确保邮件传输的安全性。本文档详细说明了如何配置和使用 SMTP 邮件服务，包括安全配置、认证最佳实践以及与主流邮件服务的集成。

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml)
- [user_app.py](file://api/apps/user_app.py)

## SMTP 配置参数详解

在 `service_conf.yaml` 配置文件中，SMTP 相关参数定义了邮件服务器的连接信息和安全设置。以下是关键参数的详细说明：

```yaml
smtp:
  mail_server: ""  # SMTP 服务器地址
  mail_port: 465   # SMTP 端口
  mail_use_ssl: true  # 是否使用 SSL 加密
  mail_use_tls: false # 是否使用 TLS 加密
  mail_username: ""   # 邮件用户名
  mail_password: ""   # 邮件密码
  mail_default_sender:
    - "RAGFlow"  # 发件人显示名称
    - ""         # 发件人邮箱地址
  mail_frontend_url: "https://your-frontend.example.com"  # 前端 URL
```

这些参数在系统启动时被读取，并用于初始化 SMTP 邮件服务器连接。`mail_server` 和 `mail_port` 定义了邮件服务器的基本连接信息，而 `mail_use_ssl` 和 `mail_use_tls` 控制加密协议的使用。

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml#L138-L148)

## 安全邮件传输配置

### TLS/SSL 加密配置

RAGFlow 系统支持 SSL 和 TLS 两种加密协议来保护邮件传输过程中的数据安全。在 `service_conf.yaml` 中，可以通过设置 `mail_use_ssl` 和 `mail_use_tls` 参数来选择合适的加密方式。

当 `mail_use_ssl` 设置为 `true` 时，系统将使用 SSL 加密连接到 SMTP 服务器。SSL 通常在端口 465 上运行，提供端到端的加密通信。代码实现中，系统使用 `smtplib.ssl.create_default_context()` 创建安全上下文，并通过 `starttls()` 方法启用 TLS 加密：

```python
context = smtplib.ssl.create_default_context()
with smtplib.SMTP(self._param.smtp_server, self._param.smtp_port) as server:
    server.ehlo()
    server.starttls(context=context)
    server.ehlo()
```

这种配置确保了邮件内容在传输过程中不会被窃听或篡改，符合现代网络安全标准。

### 邮件内容安全性

系统通过 HTML 模板来生成安全的邮件内容，避免直接在代码中硬编码敏感信息。邮件模板存储在 `api/utils/email_templates.py` 文件中，使用 Jinja2 风格的占位符来动态填充内容：

```python
RESET_CODE_EMAIL_TMPL = """
<p>Hello,</p>
<p>Your password reset code is: <b>{{ code }}</b></p>
<p>This code will expire in {{ ttl_min }} minutes.</p>
"""
```

这种设计不仅提高了代码的可维护性，还确保了邮件内容的一致性和专业性。所有敏感信息（如验证码）都通过安全的变量替换机制插入，避免了潜在的安全风险。

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml#L140-L142)
- [email.py](file://agent/tools/email.py#L147-L151)
- [email_templates.py](file://api/utils/email_templates.py#L30-L35)

## 安全相关邮件功能

### 密码重置流程

系统通过安全的密码重置流程来保护用户账户。当用户请求密码重置时，系统会生成一个一次性验证码（OTP），并通过邮件发送给用户。验证码的生成和验证过程如下：

1. 生成 8 位大写字母组成的 OTP
2. 使用盐值（salt）对 OTP 进行哈希处理
3. 将哈希值和盐值存储在 Redis 中，设置 5 分钟的过期时间
4. 通过 SMTP 发送包含验证码的邮件

```python
otp = "".join(secrets.choice(string.ascii_uppercase) for _ in range(OTP_LENGTH))
salt = os.urandom(16)
code_hash = hash_code(otp, salt)
REDIS_CONN.set(k_code, f"{code_hash}:{salt.hex()}", OTP_TTL_SECONDS)
```

这种设计确保了即使邮件被截获，攻击者也无法直接获取有效的验证码，因为存储在服务器端的是哈希值而非明文。

### 账户验证机制

账户验证功能与密码重置类似，都依赖于 OTP 机制。系统在发送验证邮件时，会检查 Redis 中是否存在未过期的验证码，防止频繁发送邮件。同时，设置了 60 秒的冷却时间，避免滥用：

```python
if last_ts:
    try:
        elapsed = now - int(last_ts)
    except Exception:
        elapsed = RESEND_COOLDOWN_SECONDS
    remaining = RESEND_COOLDOWN_SECONDS - elapsed
    if remaining > 0:
        return get_json_result(data=False, code=RetCode.NOT_EFFECTIVE, message=f"you still have to wait {remaining} seconds")
```

这种防滥用机制有效防止了邮件轰炸攻击，保护了邮件服务的稳定性和用户的收件箱。

**Section sources**
- [user_app.py](file://api/apps/user_app.py#L915-L922)
- [user_app.py](file://api/apps/user_app.py#L906-L913)

## 认证最佳实践

### 应用专用密码

对于 Gmail 等现代邮件服务，推荐使用应用专用密码（App Password）而非账户主密码。应用专用密码具有以下优势：

- **有限权限**：仅授予邮件发送权限，不访问其他账户功能
- **可撤销性**：可以单独撤销某个应用的密码而不影响主账户
- **审计能力**：可以追踪每个应用专用密码的使用情况

在配置中，`mail_password` 字段应填写应用专用密码，而不是账户的主登录密码。

### OAuth2 邮件认证

系统支持 OAuth2 认证机制，提供更高级别的安全性。通过 OAuth2，应用无需存储用户的明文密码，而是使用访问令牌进行认证。这减少了密码泄露的风险，并允许用户随时撤销应用的访问权限。

虽然当前代码主要使用传统用户名/密码认证，但系统架构支持 OAuth2 集成。在 `web/src/pages/user-setting/data-source/component/gmail-token-field.tsx` 文件中，可以看到 OAuth2 相关的前端组件，表明系统具备 OAuth2 集成的能力。

```typescript
const credentialHasRefreshToken = (content: string) => {
    try {
        const parsed = JSON.parse(content);
        return Boolean(parsed?.refresh_token);
    } catch {
        return false;
    }
};
```

这种设计允许用户上传 OAuth2 凭据文件，实现更安全的邮件认证。

**Section sources**
- [email.py](file://agent/tools/email.py#L71)
- [gmail-token-field.tsx](file://web/src/pages/user-setting/data-source/component/gmail-token-field.tsx#L27-L33)

## 主流邮件服务集成

### Gmail 集成

集成 Gmail SMTP 服务时，推荐配置如下：

```yaml
smtp:
  mail_server: "smtp.gmail.com"
  mail_port: 465
  mail_use_ssl: true
  mail_use_tls: false
  mail_username: "your-email@gmail.com"
  mail_password: "your-app-password"
  mail_default_sender:
    - "RAGFlow"
    - "your-email@gmail.com"
```

关键要点：
- 使用 `smtp.gmail.com` 作为邮件服务器
- 端口 465 配合 SSL 加密
- 启用两步验证并生成应用专用密码
- 确保账户允许"不够安全的应用"访问（如果未使用应用专用密码）

### Outlook 集成

对于 Outlook/Hotmail 服务，配置如下：

```yaml
smtp:
  mail_server: "smtp-mail.outlook.com"
  mail_port: 587
  mail_use_ssl: false
  mail_use_tls: true
  mail_username: "your-email@outlook.com"
  mail_password: "your-password"
```

关键要点：
- 使用 `smtp-mail.outlook.com` 作为邮件服务器
- 端口 587 配合 TLS 加密
- 在 Outlook 账户设置中启用 SMTP 身份验证
- 考虑使用应用密码而非主密码

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml#L138-L148)
- [email.py](file://agent/tools/email.py#L68-L71)

## 常见连接问题解决方案

### 连接超时

当出现连接超时问题时，可能的原因和解决方案包括：

1. **防火墙阻止**：检查服务器防火墙是否允许出站 SMTP 连接
2. **网络延迟**：增加连接超时时间
3. **服务器不可达**：验证邮件服务器地址和端口是否正确

系统在代码中捕获 `SMTPConnectError` 异常，并记录详细的错误信息：

```python
except smtplib.SMTPConnectError:
    error_msg = f"Failed to connect to SMTP server {self._param.smtp_server}:{self._param.smtp_port}"
    logging.error(error_msg)
```

### 认证失败

认证失败的常见原因和解决方案：

1. **密码错误**：检查 `mail_password` 是否正确
2. **应用专用密码未启用**：对于 Gmail，确保已生成应用专用密码
3. **账户安全设置**：检查邮件服务是否允许第三方应用访问

系统提供清晰的错误消息帮助诊断问题：

```python
except smtplib.SMTPAuthenticationError:
    error_msg = "SMTP Authentication failed. Please check your email and authorization code."
    logging.error(error_msg)
```

### 加密协议不匹配

当 SSL/TLS 配置与服务器要求不匹配时，会出现加密协议错误。解决方案是根据邮件服务的要求正确配置 `mail_use_ssl` 和 `mail_use_tls` 参数。

**Section sources**
- [email.py](file://agent/tools/email.py#L192-L197)
- [email.py](file://agent/tools/email.py#L199-L202)

## 防止滥用措施

系统实施了多层次的防滥用措施来保护邮件服务：

### 速率限制

通过 Redis 实现邮件发送的速率限制，防止邮件轰炸攻击：

```python
RESEND_COOLDOWN_SECONDS = 60  # 60秒内不能重复发送
```

### 尝试次数限制

对验证码验证尝试次数进行限制，防止暴力破解：

```python
ATTEMPT_LIMIT = 5  # 最多5次尝试
ATTEMPT_LOCK_SECONDS = 30 * 60  # 30分钟后解锁
```

### 一次性验证码

使用一次性验证码（OTP）机制，验证码在使用后立即失效，即使被截获也无法重复使用。

### 会话管理

通过 Redis 存储验证码和相关状态，确保每个验证码只能使用一次，并在过期后自动清除。

这些措施共同构成了一个健壮的防滥用系统，保护了邮件服务的安全性和可用性。

**Section sources**
- [web_utils.py](file://api/utils/web_utils.py#L38-L42)
- [user_app.py](file://api/apps/user_app.py#L985-L992)

## 结论

RAGFlow 系统的 SMTP 邮件服务配置提供了安全可靠的邮件传输功能。通过合理的 TLS/SSL 加密配置、应用专用密码和 OAuth2 认证最佳实践，系统能够安全地发送密码重置、账户验证等敏感邮件。与主流邮件服务（如 Gmail、Outlook）的集成简单直接，同时系统内置了完善的防滥用措施，确保邮件服务的稳定性和安全性。建议管理员根据本文档的指导进行配置，以最大化系统的安全性和可靠性。