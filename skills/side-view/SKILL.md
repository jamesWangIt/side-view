---
name: side-view
description: >
  Display a selected Claude response in a new window for easy side-by-side reading.
  Default: clean plain text in a new terminal window.
  Use 'html' argument for rendered markdown in browser.
  Trigger when user says /side-view or wants to view a response in another window.
argument-hint: "[optional: 'html' for browser mode, 'last' to skip selection]"
---

# Side View — Side-by-Side Response Viewer

## Behavior

When invoked, present a selection list of recent responses, let the user pick which one
to view, then display it in a new window.

**Display mode is determined by `$ARGUMENTS`:**
- No argument or `last` → **Terminal mode** (default)
- `html` → **Browser mode**

## Step 0: Clean up previous files

Before anything else, always clean up previous temp files from both modes:

```bash
rm -f /tmp/claude-view-*.html /tmp/.claude_side_view.txt 2>/dev/null || true
```

## Step 1: Select content

**CRITICAL: Check `$ARGUMENTS` FIRST.**

- If `$ARGUMENTS` contains "last" → skip the selection entirely, use your most recent substantive response, and go directly to Step 2. Do NOT show the AskUserQuestion selection list.
- Otherwise → use the `AskUserQuestion` tool to present a selection of your recent responses.

For the selection list: scan back through the conversation and find your last 3-4 **substantive**
responses (skip one-liners like "ok", "done", tool-only responses, and this skill's own output).
For each, create a short summary (under 15 words) as the option label.

The last option should always be "Most recent response" as a fallback.

## Step 2: Display

**CRITICAL: Check `$ARGUMENTS` FIRST before doing anything else in this step.**

- If `$ARGUMENTS` contains "html" → skip Terminal mode entirely, jump straight to **Browser mode** section below.
- Otherwise → go to **Terminal mode**.

**Do NOT detect terminal or write temp files if the mode is Browser.**

---

### Terminal mode (default)

**Before printing, convert the markdown content to clean plain text.** Strip all markdown
syntax and reformat for terminal readability:

| Markdown | Plain text |
|----------|-----------|
| `## Heading` | `Heading` followed by a line of `=====` |
| `### Subheading` | `Subheading` followed by a line of `-----` |
| `**bold text**` | `bold text` (just remove the `**`) |
| `` `inline code` `` | `inline code` (just remove the backticks) |
| Code blocks (triple backticks) | Indent each line with 4 spaces, no backtick fences |
| `- list item` | `  - list item` (keep as is) |
| `1. item` | `  1. item` (keep as is) |
| Markdown tables | Convert to aligned columns using spaces, no `|` or `---` borders |

**Table conversion example:**

Markdown:
```
| Name | Value |
|------|-------|
| foo  | 123   |
| bar  | 456   |
```

Plain text:
```
  Name    Value
  foo     123
  bar     456
```

**Add a blank line between sections for readability.**

Then write the converted plain text to a temp file, detect the current terminal, and open a new
window to display it.

**Step 2a: Write temp file**

Use the `Write` tool to write the converted plain text content:
- macOS / Linux: `/tmp/.claude_side_view.txt`
- Windows: `%TEMP%\claude-side-view.txt`

Do NOT use Bash heredoc — use the Write tool directly to avoid triggering shell security warnings
from `#` characters in the content.

**Step 2b: Detect terminal**

```bash
echo $TERM_PROGRAM
```

**Step 2c: Open new terminal window to display and clean up**

**macOS — Apple Terminal:**

```bash
osascript -e 'tell app "Terminal" to do script "cat /tmp/.claude_side_view.txt; rm -f /tmp/.claude_side_view.txt"'
```

**macOS — iTerm2:**

```bash
osascript -e 'tell app "iTerm" to create window with default profile command "cat /tmp/.claude_side_view.txt; rm -f /tmp/.claude_side_view.txt"'
```

**macOS — WezTerm:**

```bash
wezterm start -- bash -c 'cat /tmp/.claude_side_view.txt; rm -f /tmp/.claude_side_view.txt; echo "--- Press Enter to close ---"; read'
```

If `wezterm` is not in PATH, use `/Applications/WezTerm.app/Contents/MacOS/wezterm`.

**Linux:**

```bash
gnome-terminal -- bash -c 'cat /tmp/.claude_side_view.txt; rm -f /tmp/.claude_side_view.txt; echo "--- Press Enter to close ---"; read'
```

Other Linux terminals: same pattern with `konsole -e`, `xfce4-terminal -e`, `alacritty -e`, `kitty`.

**Windows — PowerShell:**

The temp file was already written by the Write tool to `%TEMP%\claude-side-view.txt`.

```powershell
Start-Process powershell -ArgumentList "-NoExit", "-Command", "Get-Content '$env:TEMP\claude-side-view.txt'; Remove-Item '$env:TEMP\claude-side-view.txt'"
```

**Windows — cmd:**

```cmd
start cmd /k "type "%TEMP%\claude-side-view.txt" & del "%TEMP%\claude-side-view.txt""
```

After opening, confirm:
> Opened in new terminal window.

---

### Browser mode (`/side-view html`)

Convert markdown to HTML, write to a temp file, and open it in the default browser.

Use the `Write` tool to write the HTML file. Do NOT use Bash to write the file.

**File path:**
- macOS / Linux: `/tmp/claude-view-{timestamp}.html`
- Windows: `%TEMP%\claude-view-{timestamp}.html`

**HTML template:**

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Side View</title>
<style>
  body {
    font-family: -apple-system, "PingFang SC", "Microsoft YaHei", "Segoe UI", sans-serif;
    max-width: 900px;
    margin: 40px auto;
    padding: 0 20px;
    color: #1a1a2e;
    line-height: 1.7;
    font-size: 15px;
  }
  h1, h2, h3 { color: #16213e; }
  h1 { border-bottom: 2px solid #0066cc; padding-bottom: 8px; }
  h2 { border-bottom: 1px solid #e8eaed; padding-bottom: 6px; margin-top: 32px; }
  table { width: 100%; border-collapse: collapse; margin: 16px 0; font-size: 14px; }
  th { background: #16213e; color: #fff; padding: 8px 12px; text-align: left; }
  td { padding: 8px 12px; border-bottom: 1px solid #e8eaed; }
  tr:nth-child(even) { background: #f8f9fa; }
  pre { background: #f6f8fa; border: 1px solid #e8eaed; border-radius: 6px; padding: 16px; overflow-x: auto; }
  code { font-family: "SF Mono", Menlo, monospace; font-size: 13px; }
  p code { background: #f0f2f5; padding: 2px 6px; border-radius: 3px; }
  blockquote { border-left: 4px solid #0066cc; background: #f0f7ff; padding: 12px 16px; margin: 12px 0; }
  strong { color: #16213e; }
  ul, ol { margin: 8px 0 16px 24px; }
</style>
</head>
<body>
{RENDERED_HTML_CONTENT}
</body>
</html>
```

**Convert markdown to HTML before inserting:**
- `## heading` → `<h2>heading</h2>`
- `**bold**` → `<strong>bold</strong>`
- `` `code` `` → `<code>code</code>`
- Markdown tables → HTML `<table>` tags
- Code blocks → `<pre><code>` tags
- Lists → `<ul><li>` or `<ol><li>` tags
- Paragraphs → `<p>` tags

**Open in browser:**
- macOS: `open {filepath}`
- Linux: `xdg-open {filepath}`
- Windows: `start "" "{filepath}"`

After opening, confirm:
> Opened in browser. File: `{filepath}`

---

## Rules

- **CRITICAL: Render the selected response faithfully.** Do not rewrite, translate, summarize, or enhance the content. If the original was in Chinese, keep Chinese. If English, keep English.
- **Terminal mode:** Convert markdown to clean plain text first. Write to temp file, display via `cat` in new terminal, then delete the temp file immediately after display.
- **Browser mode:** Convert markdown to HTML. Use the minimal template above. No sidebar, no collapsible sections, no JavaScript.
- Always detect the current terminal before opening — do not hardcode a specific terminal.
- If terminal detection fails, use the OS default fallback.
