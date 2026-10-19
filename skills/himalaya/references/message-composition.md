# Message Composition in Himalaya v2

Himalaya v2 made composition **non-interactive**. The `compose`, `reply`, and `forward` commands output RFC 5322 bytes to stdout. Use `--send` to send, `--save <mailbox>` to save a copy, or pipe to another tool.

Rich composition (MML, HTML, multipart, attachments beyond `--attach`) belongs to a standalone tool such as [mml](https://github.com/pimalaya/mml), piped into `message send` or `message add`.

## Simple Composition (from flags)

### Compose and send

```bash
himalaya message compose \
  --to "bob@example.com" \
  --subject "Meeting tomorrow" \
  --body "Hi Bob, let's meet at 3pm." \
  --send
```

### Compose with multiple recipients

```bash
himalaya message compose \
  --to "bob@example.com" \
  --to "carol@example.com" \
  --cc "dave@example.com" \
  -s "Team update" \
  --body "Here is the update..." \
  --send
```

### Compose with attachments

```bash
himalaya message compose \
  --to "bob@example.com" \
  -s "Report attached" \
  --body "Please see the attached report." \
  --attach report.pdf \
  --attach data.csv \
  --send
```

### Compose with custom signature

```bash
himalaya message compose \
  --to "bob@example.com" \
  -s "Hello" \
  --body "Hi there" \
  --signature "Best,\nAlice" \
  --send
```

### Save as draft (don't send)

```bash
himalaya message compose \
  --to "bob@example.com" \
  -s "WIP" \
  --body "Work in progress..." \
  --save Drafts
```

### Output to stdout (pipe to another tool)

```bash
himalaya message compose \
  --to "bob@example.com" \
  -s "Test" \
  --body "Hello"
```

## Reply

### Simple reply

```bash
himalaya message reply 42 --send
```

### Reply with custom body

```bash
himalaya message reply 42 --body "Thanks for the update!" --send
```

### Reply-all

```bash
himalaya message reply 42 --cc "carol@example.com" --send
```

### Reply and save copy

```bash
himalaya message reply 42 --body "Got it" --send --save Sent
```

## Forward

```bash
himalaya message forward 42 --to "bob@example.com" --send
```

With additional context:

```bash
himalaya message forward 42 \
  --to "bob@example.com" \
  --body "FYI, see below." \
  --send
```

## Send a Raw Message

Send from a file:

```bash
himalaya message send /path/to/message.eml
```

Send from stdin:

```bash
cat message.eml | himalaya message send
```

Send with save copy override:

```bash
himalaya message send message.eml --save "Sent Items"
himalaya message send message.eml --no-save
```

## Rich Composition with MML

For HTML emails, multipart messages, inline images, and advanced MIME features, use [mml](https://github.com/pimalaya/mml).

### MML message structure

An MML message is headers + body, with `<#part>` and `<#multipart>` directives for MIME:

```
From: alice@example.com
To: bob@example.com
Subject: Rich Email

Plain text fallback.

<#multipart type=alternative>
This is the plain text version.
<#part type=text/html>
<html><body><h1>HTML version</h1></body></html>
<#/multipart>
```

### Compile and send with mml

```bash
mml compile < message.mml | himalaya message send
```

### Attachments in MML

```
From: alice@example.com
To: bob@example.com
Subject: With Attachment

Please see attached.

<#multipart type=mixed>
<#part type=text/plain>
See the attached files.
<#part filename=/path/to/report.pdf>
<#part filename=/path/to/data.csv>
<#/multipart>
```

### Inline images in MML

```
<#multipart type=related>
<#part type=text/html>
<html><body><img src="cid:image1"></body></html>
<#part disposition=inline id=image1 filename=/path/to/photo.png>
<#/multipart>
```

## Posting Style

Control where quoted text appears in replies and forwards:

```toml
# In config.toml
message.reply.posting-style = "top"     # cursor above quoted text (default)
message.forward.posting-style = "top"   # cursor above forwarded content
```

Options: `top`, `bottom`, `none`.

## Save Copy Behavior

```toml
# In config.toml
message.send.save-copy = "sent"    # auto-save to Sent mailbox
```

Override per-command:

```bash
himalaya message compose --to "bob@example.com" -s "Test" --body "Hi" --send --save "Archive"
himalaya message compose --to "bob@example.com" -s "Test" --body "Hi" --send --no-save
```

## MML Tag Reference

### `<#multipart>`

Groups multiple MIME parts.

- `type=alternative` — different representations of same content (text + HTML)
- `type=mixed` — independent parts (text + attachments)
- `type=related` — parts that reference each other (HTML + inline images)

### `<#part>`

Defines a message part.

- `type=<mime-type>` — content type (e.g., `text/html`, `application/pdf`)
- `filename=<path>` — file to attach
- `name=<name>` — display name for attachment
- `disposition=inline` — display inline instead of as attachment
- `id=<cid>` — Content ID for referencing in HTML

## Differences from v1

| v1 | v2 |
|---|---|
| `message write` (interactive, opens $EDITOR) | `message compose` (non-interactive, flags) |
| `template send` | `message send` (reads from file/stdin) |
| `message reply 42` (opens $EDITOR) | `message reply 42 --send` (non-interactive) |
| `message forward 42` (opens $EDITOR) | `message forward 42 --to ... --send` |
| `message write -H "To:..." "body"` | `message compose --to ... --body ...` |
| Interactive editor workflow | Pipe through mml for rich composition |
