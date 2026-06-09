# Changelog

All notable changes are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Fixed

- **Windows support**: `read_single_keypress()` now uses `msvcrt.getch()` on Windows instead of `termios`/`tty` (Unix-only modules). Runs cross-platform without `ModuleNotFoundError`.

---

## [0.29.0] — Previous release

> See the [GitHub Releases](https://github.com/jonigl/mcp-client-for-ollama/releases) for the full changelog of all past versions.

### Added

- Agent Mode with configurable loop limit
- MCP Prompts support (browse, invoke, argument collection, preview, rollback)
- MCP Resources support (`@uri` inline syntax, buffered injection, vision image forwarding)
- Ollama Cloud model support
- Multiline chat input mode
- Answer display modes (Plain / Markdown / Both)
- HIL session auto-approve and query abort options
- History management (export/import JSON)
- Thinking mode toggle
- Server hot-reload via `/reload-servers`
- Performance metrics panel

### Changed

- Migrated from `rich.prompt` to `prompt_toolkit` for input
- Rich TUI with fuzzy autocomplete
- Default model: `qwen2.5:7b`

### Fixed

- Vision image forwarding for non-vision models
- History rollback on abort
