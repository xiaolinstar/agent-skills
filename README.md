# Agent Skills

A collection of [Agent Skills](https://skills.sh) for CLI tools.

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [himalaya](skills/himalaya/) | Himalaya v2 email CLI — list, read, compose, reply, forward, search emails via IMAP/SMTP/JMAP/Gmail/MSGraph | `npx skills add xiaolinstar/agent-skills --skill himalaya` |

## Usage

```bash
# Install a skill
npx skills add xiaolinstar/agent-skills --skill himalaya

# Install globally
npx skills add xiaolinstar/agent-skills --skill himalaya -g

# List available skills
npx skills add xiaolinstar/agent-skills --list
```

## Skills

### himalaya

Himalaya v2 email CLI skill. Covers:

- IMAP/SMTP/JMAP/Gmail/MSGraph/Maildir configuration (v2 format)
- Mailbox, envelope, message, flag, and attachment management
- Non-interactive compose/reply/forward with `--send`/`--save`
- MML rich composition via [pimalaya/mml](https://github.com/pimalaya/mml)
- Multiple accounts, search, JSON output, debugging

Compatible with himalaya v2.x.
