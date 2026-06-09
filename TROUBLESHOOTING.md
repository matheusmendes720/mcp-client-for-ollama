# Troubleshooting

Solutions to common problems.

## Installation

### `ModuleNotFoundError: No module named 'termios'`

**Cause:** Running on Windows. The code imported Unix-only `tty`/`termios`.

**Fix:** Pull the latest version — this was fixed in `v0.29.1`.

```bash
pip install --upgrade ollmcp
# or
uvx ollmcp --version
```

---

### `uv: command not found`

**Fix:** Install UV:
```bash
# Linux/macOS
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

---

### `pip install` fails with dependency conflicts

**Fix:** Use UV instead — it resolves dependencies more aggressively:
```bash
uv pip install --upgrade ollmcp
```

---

## Runtime

### `Error: connection refused` / Ollama not running

**Cause:** Ollama is not running or not on the default port.

**Fix:**
```bash
# Start Ollama
ollama serve

# Use a different host
ollmcp --host http://localhost:11434
```

---

### `preflight_ollama: Ollama is not reachable`

**Cause:** The client checks Ollama connectivity on startup.

**Fix:** Ensure Ollama is running and responding:
```bash
curl http://localhost:11434/api/tags
```

---

### Model not found / wrong model used

**Cause:** Model not pulled, or wrong name.

**Fix:**
```bash
# List installed models
ollama list

# Pull a model
ollama pull qwen3:1.7b

# Use it
ollmcp --model qwen3:1.7b
```

---

### No MCP servers connected

**Cause:** No `--servers-json`, `--mcp-server`, `--mcp-server-url`, or `--auto-discovery` flag provided.

**Fix:**
```bash
# Auto-discover from Claude config
ollmcp --auto-discovery

# Or specify a config file
ollmcp --servers-json ~/.config/ollmcp/mcp-servers/config.json
```

---

### HIL confirmation always prompts even when disabled

**Cause:** HIL state persists per session only. Restarting the client resets it.

**Fix:** Use `/human-in-the-loop` or `/hil` to toggle, or pass `--hil` flag (if supported).

---

### Multiline input: Esc+Enter doesn't send

**Cause:** Terminal does not emit the expected escape sequence.

**Fix:**
- Try **Ctrl+J** followed by **Enter** as an alternative submit shortcut
- Some terminals require **Meta+Enter** (Alt+Enter)
- If all fail, stay in single-line mode (`/input-mode` → Single-line)

---

### Agent Mode runs forever / infinite loop

**Cause:** Model keeps requesting tool calls; loop limit may be too high.

**Fix:**
```bash
# Reduce loop limit to 1 (disable Agent Mode)
/loop-limit 1
# or
/ll 1
```

---

### `save-config` / `load-config` doesn't affect MCP servers

**Cause:** The config saves TUI preferences (model, tools, display settings). It does **not** register MCP server connections.

**Fix:** Use `--servers-json` at startup to configure MCP servers. Use `save-config` only for tool enable/disable and model preferences.

---

## Configuration

### Claude auto-discovery fails

**Cause:** Claude config not found or malformed.

**Fix:** Verify your Claude config exists:
```bash
# Linux/macOS
ls ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Windows
dir "%APPDATA%\Claude\claude_desktop_config.json"
```

---

### GitHub MCP server returns 401/403

**Cause:** Invalid or missing GitHub API token.

**Fix:** Add your token to the server config:
```json
{
  "mcpServers": {
    "github": {
      "type": "streamable_http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_ACTUAL_TOKEN_HERE"
      }
    }
  }
}
```

---

### `~/.config/ollmcp/config.json` not found

**Cause:** Default config is created on first `save-config`. The directory may not exist.

**Fix:** Run `save-config` from within the TUI, or create the directory:
```bash
mkdir -p ~/.config/ollmcp
```

---

## Development

### Tests fail after pulling

**Fix:** Install dev dependencies:
```bash
uv pip install -e ".[dev]"
pytest
```

---

### `ruff check` shows errors

**Fix:** Auto-fix lint issues:
```bash
ruff check --fix mcp_client_for_ollama/
```

---

### Windows: `source .venv/bin/activate` fails

**Fix:** On Windows, use:
```cmd
.venv\Scripts\activate
```

---

## Still Stuck?

- Search [existing issues](https://github.com/jonigl/mcp-client-for-ollama/issues)
- Open a [new issue](https://github.com/jonigl/mcp-client-for-ollama/issues/new)
- For security issues → use the **Security** tab → **Report a vulnerability**