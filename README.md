# qazuor-claude

Persistent [Claude Code](https://docs.anthropic.com/en/docs/claude-code) sessions via **tmux + ngrok**, controllable from your phone.

## What it does

- Runs Claude Code inside a **tmux** session that survives terminal disconnects
- Exposes it via **ngrok** so you can access it remotely (phone, tablet, another machine)
- Persists session metadata (URL, path, timestamps) to disk
- Sends notifications via [qz-notifier](https://github.com/qazuor/qz-notifier) when a session starts or recovers
- Provides a single CLI to create, attach, stop, list, recover, and inspect sessions

## How it works

```
┌─────────────────────────────────────────┐
│  tmux session: qazuor-claude::<name>    │
│                                         │
│  window 0: claude    → runs `claude`    │
│  window 1: ngrok     → runs `ngrok`     │
└─────────────────────────────────────────┘
        │
        ├── metadata saved to ~/.qazuor-claude/<name>.json
        └── notification sent via qz-notifier
```

- **tmux** owns the processes. Closing your terminal changes nothing.
- **ngrok** creates a public tunnel so you can reach Claude from anywhere.
- **metadata** is a simple JSON file for recovery and context.

## Requirements

| Dependency | Purpose |
|------------|---------|
| [tmux](https://github.com/tmux/tmux) | Session management |
| [jq](https://jqlang.github.io/jq/) | JSON processing |
| [curl](https://curl.se/) | HTTP requests |
| [ngrok](https://ngrok.com/) | Public tunnel |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | AI coding assistant |
| [qz-notifier](https://github.com/qazuor/qz-notifier) | Notifications (optional but recommended) |

Run `qazuor-claude doctor` to verify everything is installed.

## Installation

### Option 1: Symlink (recommended)

```bash
sudo ln -s /path/to/qz-claude-remote/qazuor-claude /usr/local/bin/qazuor-claude
```

### Option 2: Add to PATH

**fish:**
```fish
fish_add_path /path/to/qz-claude-remote
```

**bash/zsh:**
```bash
export PATH="/path/to/qz-claude-remote:$PATH"
```

### Verify

```bash
qazuor-claude doctor
```

## Commands

### `init [name]`

Create a new session. If `name` is omitted, uses the current directory name.

```bash
cd ~/projects/myapp
qazuor-claude init          # session named "myapp"
qazuor-claude init custom   # session named "custom"
```

If the session already exists, you get three choices:
- **(a)** attach to the existing session
- **(n)** create a new one with numeric suffix (`myapp-2`, `myapp-3`, ...)
- **(q)** cancel

What happens:
1. Creates tmux session `qazuor-claude::<name>`
2. Opens window `claude` running `claude`
3. Opens window `ngrok` running `ngrok http 7681`
4. Polls ngrok API for the public URL
5. Saves metadata to `~/.qazuor-claude/<name>.json`
6. Sends notification with URL and quick commands

### `attach [name]`

Attach to an existing session. Defaults to current directory name.

```bash
qazuor-claude attach myapp
```

Works both from inside tmux (switches client) and outside (attaches).

### `stop <name>`

Stop a session. Asks for confirmation before killing.

```bash
qazuor-claude stop myapp
```

Kills the tmux session and removes the metadata file.

### `list`

List all active `qazuor-claude` sessions.

```bash
qazuor-claude list
```

Shows session name, ngrok URL, and working directory.

### `rename <old> <new>`

Rename a session. Validates that the new name doesn't collide.

```bash
qazuor-claude rename myapp myapp-v2
```

### `info <name>`

Show full details for a session: tmux windows and persisted metadata.

```bash
qazuor-claude info myapp
```

### `open <name>`

Print the ngrok public URL. Useful for quick copy-paste.

```bash
qazuor-claude open myapp
```

### `recover <name>`

If ngrok died or the tunnel expired, this command:
1. Kills the old ngrok window
2. Creates a new one
3. Waits for the new tunnel URL
4. Updates metadata
5. Sends a new notification

```bash
qazuor-claude recover myapp
```

### `doctor`

Check all dependencies and show their versions.

```bash
qazuor-claude doctor
```

### `help`

Show usage and available commands.

```bash
qazuor-claude help
```

## Metadata

Session metadata is stored in `~/.qazuor-claude/<name>.json`:

```json
{
  "name": "myapp",
  "path": "/home/user/projects/myapp",
  "ngrokUrl": "https://abc123.ngrok-free.app",
  "startedAt": "2026-02-13T18:41:22-03:00"
}
```

## Notifications

Notifications are sent via [qz-notifier](https://github.com/qazuor/qz-notifier) using the `notify` command. Each notification includes:

- Session name
- Public ngrok URL
- Attach command
- Stop command

The URL is also set as the **click action**, so tapping the notification on your phone opens Claude directly.

### Setting up notifications

1. Install [qz-notifier](https://github.com/qazuor/qz-notifier)
2. Configure at least one channel (ntfy push, Telegram, or email)
3. That's it. `qazuor-claude` calls `notify` automatically

If `notify` is not installed, notifications are silently skipped.

## Using from your phone

### Quick access

1. Run `qazuor-claude init myproject` on your machine
2. You'll receive a notification with the ngrok URL
3. Tap the notification or run `qazuor-claude open myproject` to get the URL
4. Open it in your phone's browser
5. You now have Claude Code running in your browser, connected to your machine

### If the URL stops working

ngrok free tunnels expire or can drop. Just run:

```bash
qazuor-claude recover myproject
```

A new notification with the fresh URL will arrive on your phone.

### Recommended phone setup

- **ntfy app** (Android/iOS): Real-time push notifications
- **Telegram**: If you have qz-notifier configured with Telegram, notifications arrive in your chat
- Bookmark the ngrok URL for quick access

## Typical workflow

```bash
# Start your day
cd ~/projects/myapp
qazuor-claude init

# Work from your desk
qazuor-claude attach myapp

# Detach (Ctrl+B, D) and go grab coffee
# Open the URL on your phone
# Keep working from the couch

# Back at your desk
qazuor-claude attach myapp

# ngrok tunnel expired?
qazuor-claude recover myapp

# Done for the day
qazuor-claude stop myapp
```

## File structure

```
~/.qazuor-claude/              Metadata directory
  <name>.json                  Session metadata (one per session)

tmux sessions:
  qazuor-claude::<name>        Session namespace
    window: claude             Claude Code process
    window: ngrok              ngrok tunnel process
```

## License

MIT
