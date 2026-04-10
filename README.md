# side-view

A Claude Code plugin that displays Claude's responses in a separate window for easy side-by-side reading while continuing your conversation.

## Why

Claude Code runs in a terminal — when a response is long, you have to scroll back and forth between the response and your current work. This plugin opens a **separate window** with the selected response, so you can read it side by side with your conversation.

## Two Display Modes

| Mode | Command | Output | Best for |
|------|---------|--------|----------|
| **Terminal** (default) | `/side-view` | New terminal window | Quick reference, plain text content |
| **Browser** | `/side-view html` | Default browser tab | Tables, code blocks, rich formatting |

**Terminal mode** converts markdown to plain text. Since terminals cannot render markdown natively, headings, bold, tables etc. are approximated with plain text formatting. If you need better formatting, use **browser mode**.

**Browser mode** renders the response as styled HTML in your default browser, with proper headings, tables, code highlighting, and readable typography.

## Installation

Add it as a marketplace in your Claude Code settings (`~/.claude/settings.json`):

### From a Git repository

```json
{
  "extraKnownMarketplaces": {
    "jameswang-plugins": {
      "source": {
        "source": "git",
        "url": "https://github.com/jamesWangIt/side-view.git"
      }
    }
  },
  "enabledPlugins": {
    "side-view@jameswang-plugins": true
  }
}
```

### From a local directory

```json
{
  "extraKnownMarketplaces": {
    "local-plugins": {
      "source": {
        "source": "directory",
        "path": "/path/to/side-view"
      }
    }
  },
  "enabledPlugins": {
    "side-view@local-plugins": true
  }
}
```

## Usage

### Select from recent responses

```
/side-view
```

Presents a list of your recent substantive responses. Pick one, and it opens in a new terminal window.

### View most recent response (skip selection)

```
/side-view last
```

### View in browser with rich formatting

```
/side-view html
```

Opens the selected response as a styled HTML page in your default browser. Recommended for responses with tables, code blocks, or complex formatting.

## How It Works

1. Scans conversation history for recent substantive responses
2. Presents a selection list (unless `last` is specified)
3. Converts the response content (to plain text for terminal, to HTML for browser)
4. Terminal mode: writes to a temp file, detects current terminal, opens a new window to display the content, then cleans up the temp file
5. Browser mode: writes an HTML file to the temp directory and opens it in the default browser

## Supported Terminals

### macOS
- Terminal.app
- iTerm2
- WezTerm
- Fallback: opens Terminal.app

### Linux
- gnome-terminal, konsole, xfce4-terminal, alacritty, kitty
- Fallback: `x-terminal-emulator`

### Windows
- PowerShell, cmd, Windows Terminal

## Requirements

- Claude Code CLI

## Recommended Permissions

To avoid repeated confirmation prompts, add these to your `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(echo $TERM_PROGRAM)",
      "Bash(osascript*)",
      "Bash(rm -f /tmp/claude-view-*)",
      "Bash(open /tmp/claude-view-*)",
      "Bash(xdg-open /tmp/claude-view-*)",
      "Write(/tmp/.claude_side_view*)",
      "Write(/tmp/claude-view-*)"
    ]
  }
}
```

## License

MIT
