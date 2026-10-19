# Himalaya v2 Configuration Reference

Configuration file location: `~/.config/himalaya/config.toml`

Override with `himalaya -c <PATH>`. Multiple paths separated by `:` are deep-merged.

## Global Options

```toml
display-name = "Your Name"
signature = "Best regards,\nYour Name"
signature-delim = "-- \n"
downloads-dir = "~/Downloads"
message.send.save-copy = "sent"
message.reply.posting-style = "top"    # top, bottom, or none
envelope.list.page-size = 50
```

## Minimal IMAP + SMTP Setup (v2)

```toml
[accounts.personal]
email = "user@example.com"
display-name = "Your Name"
default = true

# IMAP: URL scheme determines encryption
imap.server = "imaps://imap.example.com:993"
# Alternatives:
# imap.server = "imap://imap.example.com:143"   (plain, use imap.starttls = true)
# imap.server = "example.com"                    (bare authority, treated as imaps://)

# SASL LOGIN authentication
imap.sasl.login.username = "user@example.com"
imap.sasl.login.password.raw = "your-password"
# Or from a command (recommended):
# imap.sasl.login.password.command = "pass show email/imap"

# SMTP
smtp.server = "smtps://smtp.example.com:465"
# For STARTTLS:
# smtp.server = "smtp://smtp.example.com:587"
# smtp.starttls = true

smtp.sasl.login.username = "user@example.com"
smtp.sasl.login.password.raw = "your-password"
# smtp.sasl.login.password.command = "pass show email/smtp"
```

## Password / Secret Options

### Raw value (testing only)

```toml
imap.sasl.login.password.raw = "your-password"
```

### Command (recommended)

```toml
imap.sasl.login.password.command = "pass show email/imap"
# or as array:
# imap.sasl.login.password.command = ["pass", "show", "email/imap"]
```

### macOS Keychain via security command

```toml
imap.sasl.login.password.command = "security find-generic-password -a user@example.com -s imap -w"
```

> **Note:** Native keyring support was removed in v2. Use a password manager CLI instead.

## SASL Mechanisms

### SASL PLAIN

```toml
imap.sasl.plain.authcid = "user@example.com"
imap.sasl.plain.passwd.raw = "your-password"
```

### SASL LOGIN (most common for Chinese providers)

```toml
imap.sasl.login.username = "user@example.com"
imap.sasl.login.password.raw = "your-password"
```

### SASL OAUTHBEARER

```toml
imap.sasl.oauthbearer.username = "user@example.com"
imap.sasl.oauthbearer.host = "imap.example.com"
imap.sasl.oauthbearer.port = 993
imap.sasl.oauthbearer.token.command = ["ortie", "token", "read", "example"]
```

### SASL XOAUTH2 (Google)

```toml
imap.sasl.xoauth2.username = "user@example.com"
imap.sasl.xoauth2.token.raw = "***"
```

### SASL SCRAM-SHA-256

```toml
imap.sasl.scram-sha-256.username = "user@example.com"
imap.sasl.scram-sha-256.password.raw = "***"
```

## TLS Configuration

```toml
# TLS provider (build-time feature, not runtime config)
# Default: rustls-ring. Alternatives: native-tls, rustls-aws
# Build with: cargo build --features native-tls --no-default-features

# Custom certificate
# imap.tls.cert = "/path/to/custom/cert.pem"

# Force TLS provider
# imap.tls.provider = "rustls"    # or "native-tls"
# imap.tls.rustls.crypto = "ring" # or "aws"
```

## Gmail Configuration (v2 — Gmail REST API backend)

```toml
[accounts.gmail]
email = "you@gmail.com"
display-name = "Your Name"
default = true

# OAuth 2.0 bearer token from ortie
gmail.auth.token.command = ["ortie", "token", "show", "-a", "gmail"]
# or: gmail.auth.token.raw = "***"
```

> No IMAP/SMTP blocks needed. Labels are mailboxes, addressed by name.

## Microsoft Graph Configuration (v2)

```toml
[accounts.msgraph]
email = "you@outlook.com"
display-name = "Your Name"

msgraph.auth.token.command = ["ortie", "token", "show", "-a", "msgraph"]
```

> Sending goes through Graph API, no SMTP block needed.

## JMAP Configuration (v2)

```toml
[accounts.fastmail]
email = "you@fastmail.com"
display-name = "Your Name"

jmap.server = "fastmail.com"
# or: jmap.server = "https://api.fastmail.com/jmap/session"

# Bearer token
jmap.auth.bearer.token.command = ["pass", "show", "fastmail-api"]

# Optional: pin identity and drafts mailbox
# jmap.identity-id = "I0123abc"
# jmap.drafts-mailbox-id = "M0123abc"
```

## Maildir Configuration (v2)

```toml
[accounts.local]
email = "user@example.com"

maildir.root = "~/Mail/example"
```

## m2dir Configuration (v2, new)

```toml
[accounts.local]
email = "user@example.com"

m2dir.root = "~/Mail/example"
```

## Mailbox Aliases

```toml
[mailbox.alias]
inbox = "INBOX"
sent = "Sent"
drafts = "Drafts"
trash = "Trash"
```

Account-level aliases override global ones:

```toml
[accounts.work.mailbox.alias]
inbox = "INBOX"
sent = "Sent Items"
```

Alias names are case-insensitive. An entry named after a role (`inbox`, `sent`, `trash`) overrides the backend-reported role.

## Multiple Accounts

```toml
[accounts.personal]
email = "personal@example.com"
default = true
# ... imap/smtp config ...

[accounts.work]
email = "work@company.com"
# ... imap/smtp config ...
```

Switch with `-a`:

```bash
himalaya -a work envelope list
```

## Provider-Specific Quirks

### Coremail (163 Mail, 139 Mail)

```toml
imap.sasl-ir = false
imap.id.auto = true
```

### QQ Mail / Foxmail

Works with both TLS providers. Authorization code required (not QQ password).

### 139 Mail

Requires `native-tls` build (rustls-ring fails TLS handshake with `HandshakeFailure`).

## Table Customization

```toml
table.preset = "││──╞═╪╡┆    ┬┴┌┐└┘"
table.arrangement = "dynamic"    # or "dynamic-full-width", "disabled"
envelope.list.table.from-color = "Cyan"
```

## Removed in v2 (from v1)

- `backend.type = "imap"` / `backend.host` / `backend.port` → use `imap.server`
- `message.send.backend.type = "smtp"` → use `smtp.server`
- `backend.auth.keyring` → use `password.command`
- `backend.auth.type = "oauth2"` → use SASL `oauthbearer`/`xoauth2` with [ortie](https://github.com/pimalaya/ortie)
- `[folder.alias]` → `[mailbox.alias]`
- Notmuch backend → removed (may return in future)
- Sendmail backend → removed
- Native keyring → removed
- Interactive `write`/`reply`/`forward` → non-interactive, use [mml](https://github.com/pimalaya/mml)
