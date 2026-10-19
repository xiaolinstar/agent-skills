---
name: himalaya
slug: himalaya
displayName: Himalaya
version: 2.2.1
description: "CLI to manage emails via IMAP/SMTP/JMAP/Gmail/MSGraph/Maildir. Use `himalaya` to list, read, compose, reply, forward, search, and organize emails from the terminal. Supports multiple accounts. Rich composition delegated to mml."
description_zh: "终端邮件管理 CLI 工具。使用 `himalaya` 在终端中列出、阅读、编写、回复、转发、搜索及整理邮件。支持多账户和 MML（MIME 元语言）编写邮件。Himalaya v2.2.1+ 版本，更新配置格式，支持非交互式撰写。"
homepage: https://github.com/pimalaya/himalaya
tags:
  - email
  - cli
  - imap
  - smtp
  - mail
  - mml
metadata:
  {
    "openclaw":
      {
        "emoji": "📧",
        "requires": { "bins": ["himalaya"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "himalaya",
              "bins": ["himalaya"],
              "label": "Install Himalaya (brew)",
            },
          ],
      },
  }
---

# Himalaya Email CLI (v2)

Himalaya is a CLI email client for managing emails from the terminal. It supports IMAP, SMTP, JMAP, Gmail REST API, Microsoft Graph, Maildir, m2dir, and mbox backends.

## References

- `references/configuration.md` — v2 config format, IMAP/SMTP/JMAP setup, SASL auth
- `references/message-composition.md` — MML syntax for rich emails, composition workflow

## Prerequisites

1. Himalaya CLI v2 installed (`himalaya --version` to verify)
2. A configuration file at `~/.config/himalaya/config.toml`
3. Credentials configured (password via command or raw value)

## Setup

Run the interactive wizard to discover and configure an account:

```bash
himalaya configure
```

Or create `~/.config/himalaya/config.toml` manually. See `references/configuration.md` for the full v2 config format.

## Common Operations

### List Mailboxes

```bash
himalaya mailbox list
himalaya mailbox list --counts
```

### List Emails

List envelopes in INBOX (default):

```bash
himalaya envelope list
```

List in a specific mailbox:

```bash
himalaya envelope list -m "Sent"
```

Paginate:

```bash
himalaya envelope list --page 1 --page-size 20
```

### Search Emails

```bash
himalaya envelope search -- from "alice@example.com"
himalaya envelope search -- subject "invoice"
himalaya envelope search -- from "alice" subject "meeting"
himalaya envelope search -- flag seen
himalaya envelope search -- "not flag seen"
himalaya envelope search --page-size 50 -- subject "report"
```

> **Note:** All flags (`--page-size`, `-m`, etc.) must come **before** the `--` separator and query terms. The query is a variadic argument that consumes everything after `--`.

Search supports: `date`, `after`, `from`, `to`, `subject`, `body`, `flag`, combined with `and`, `or`, `not`, parentheses. Sort with `order by date desc`.

### Read an Email

```bash
himalaya message read 42
```

Raw RFC 5322 output:

```bash
himalaya message read 42 --raw
```

### Compose a New Email

Non-interactive, from flags:

```bash
himalaya message compose --to "bob@example.com" --subject "Hello" --body "Hi there" --send
```

With attachments:

```bash
himalaya message compose --to "bob@example.com" -s "Report" --body "See attached" --attach report.pdf --send
```

Save a copy without sending:

```bash
himalaya message compose --to "bob@example.com" -s "Draft" --body "WIP" --save Drafts
```

For rich composition (MML, HTML, multipart), pipe through [mml](https://github.com/pimalaya/mml):

```bash
mml compile < message.mml | himalaya message send
```

### Send a Raw Message

From a file:

```bash
himalaya message send /path/to/message.eml
```

From stdin:

```bash
cat message.eml | himalaya message send
```

### Reply to an Email

```bash
himalaya message reply 42 --send
himalaya message reply 42 --body "Thanks!" --send
```

Reply-all:

```bash
himalaya message reply 42 --cc "carol@example.com" --send
```

### Forward an Email

```bash
himalaya message forward 42 --to "bob@example.com" --send
```

### Move/Copy Emails

```bash
himalaya message move 42 --to "Archive"
himalaya message copy 42 --to "Important"
```

With explicit source:

```bash
himalaya message move 42 --from INBOX --to "Archive"
```

### Delete an Email

```bash
himalaya message delete 42
```

### Manage Flags

```bash
himalaya flag add 42 --flag seen
himalaya flag set 42 --flag seen --flag flagged
himalaya flag remove 42 --flag seen
```

## Multiple Accounts

List accounts:

```bash
himalaya account list
```

Use a specific account:

```bash
himalaya -a work envelope list
```

Validate connections:

```bash
himalaya account check
```

## Attachments

List attachments:

```bash
himalaya attachment list 42
```

Download all attachments:

```bash
himalaya attachment download 42
```

Download to specific directory:

```bash
himalaya attachment download 42 --dir ~/Downloads
```

## Output Formats

JSON output:

```bash
himalaya envelope list --json
himalaya mailbox list --json
```

## Debugging

```bash
himalaya --log-level debug envelope list
himalaya --log-level trace account check
himalaya --log-level debug --log-file /tmp/himalaya.log envelope list
```

## Tips

- Use `himalaya --help` or `himalaya <command> --help` for detailed usage.
- Message IDs are relative to the current mailbox; re-list after mailbox changes.
- For rich emails with attachments and HTML, use [mml](https://github.com/pimalaya/mml) and pipe into `message send`.
- Store passwords securely using a password manager CLI (`pass`, `secret-tool`, `security`, etc.).
- v2 is not interactive: `compose`, `reply`, `forward` output to stdout. Use `--send` to send, `--save` to save a copy.
