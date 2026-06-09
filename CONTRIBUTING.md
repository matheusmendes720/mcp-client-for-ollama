# Contributing

Guide for developers who want to hack on `ollmcp`.

## Quick Setup

```bash
git clone https://github.com/matheusmendes720/mcp-client-for-ollama.git
cd mcp-client-for-ollama

# Create virtual environment
uv venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows

# Install in editable mode
uv pip install -e ".[dev]"

# Run
ollmcp
# or
uv run -m mcp_client_for_ollama
```

## Project Layout

| Path | What it is |
|------|-----------|
| `mcp_client_for_ollama/` | Main package source |
| `cli-package/` | Separate `ollmcp` PyPI package (CLI wrapper) |
| `tests/` | Unit tests (pytest) |
| `scripts/` | Release/versioning scripts |
| `.github/workflows/` | CI + publish GitHub Actions |
| `misc/` | Screenshots, logos, GIFs for docs |

## Running Tests

```bash
uv pip install -e ".[dev]"
pytest
```

## Branch Strategy

```
origin/main  ← upstream (jonigl/mcp-client-for-ollama)
upstream/main ← your fork (matheusmendes720/mcp-client-for-ollama)
main           ← your local development
```

### Keeping Your Fork in Sync

```bash
# Fetch original changes
git fetch origin

# Merge into your main
git checkout main
git merge origin/main

# Push to your fork
git push upstream main
```

## Coding Standards

| Rule | Tool | Config |
|------|------|--------|
| Linting | `ruff` | `ruff.toml` (auto-generated) |
| Line length | 88 chars | Ruff default |
| Import sorting | Ruff auto-fix | `ruff check --fix` |

Run before committing:
```bash
ruff check mcp_client_for_ollama/
ruff format mcp_client_for_ollama/
```

## Cross-Platform Notes

- `tty` / `termios` → Unix only (raw keypress input)
- `msvcrt` → Windows only (raw keypress input)
- Use `sys.platform != "win32"` to branch
- All file paths use `/` (forward slash) — Python handles this on Windows

See `utils/input.py` for the pattern.

## Release Process

> [!NOTE]
> This applies to the upstream maintainer (`jonigl`). Fork contributors don't need to run releases.

1. Bump version:
   ```bash
   python scripts/bump_version.py patch  # or minor / major
   ```
2. Commit and tag:
   ```bash
   git add -A && git commit -m "chore(release): bump to X.Y.Z"
   git tag -a vX.Y.Z -m "Version X.Y.Z"
   git push origin main --tags
   ```
3. GitHub Actions builds and publishes both packages to PyPI.

## Submitting Changes

1. Fork the repo and create a feature branch
2. Make your changes with tests
3. Ensure `ruff check` and `pytest` pass
4. Push to your fork
5. Open a PR against `jonigl/mcp-client-for-ollama`

## Two-Package Structure

This project ships two PyPI packages:

| Package | Purpose | `pyproject.toml` |
|---------|---------|-----------------|
| `mcp-client-for-ollama` | Main library | Root `pyproject.toml` |
| `ollmcp` | CLI alias + thin wrapper | `cli-package/pyproject.toml` |

The `bump_version.py` script keeps both in sync. Don't edit versions manually.

## Common Dev Tasks

### Add a new interactive command

1. Add command handler in `prompts/commands.py` (`run_slash_command()`)
2. Register shortcut in `client.py` command table
3. Add tests in `tests/test_slash_command_execution.py`

### Add a new MCP transport

1. Add transport type in `server/connector.py`
2. Update `ServerConfigurationFormat` in `README.md`
3. Add tests in `tests/test_connector.py`

### Add model parameters

1. Add parameter in `models/config_manager.py` (model params dict)
2. Update `ModelConfigManager` TUI interface
3. Pass through `Ollama.chat()` call in `client.py`
4. Add tests in `tests/test_config_manager.py`