# Architecture

How `ollmcp` is structured, how data flows, and how the pieces fit together.

## Quick Reference

| Layer | Module | Purpose |
|-------|--------|---------|
| Entry point | `cli.py` → `client.py` | Typer CLI → `MCPClient` |
| Transport | `server/connector.py` | STDIO / SSE / Streamable HTTP |
| LLM | `models/manager.py` + `models/config_manager.py` | Ollama interface + param config |
| MCP primitives | `tools/`, `prompts/`, `resources/` | Three pillars of the MCP spec |
| UI | `client.py` + `utils/streaming.py` | TUI rendering + streaming output |
| Safety | `utils/hil_manager.py` | Human-in-the-Loop confirmation |
| Input | `utils/fzf_style_completion.py` | Slash-command fuzzy autocomplete |

---

## Module Map

```
mcp_client_for_ollama/
├── cli.py                        # Entry point: typer app()
├── client.py                     # MCPClient — main orchestration class (1925 lines)
├── __main__.py                   # python -m entry point
├── config/
│   ├── manager.py                # ConfigManager — save/load JSON configs
│   └── defaults.py               # Default config schema
├── models/
│   ├── manager.py                # ModelManager — list/switch Ollama models
│   └── config_manager.py         # ModelConfigManager — 15+ model parameters TUI
├── server/
│   ├── connector.py              # ServerConnector — MCP transport lifecycle (519 lines)
│   └── discovery.py              # Auto-discover servers from Claude's config
├── tools/
│   └── manager.py                # ToolManager — enable/disable per-tool, per-server
├── prompts/
│   ├── manager.py                # PromptManager — browse/invoke prompts from servers
│   ├── handler.py                # PromptHandler — argument collection, preview, rollback
│   ├── commands.py               # run_slash_command() — dispatch /command routing
│   ├── routing.py                # parse_user_input() — detect @uri, /prompts, @server://
│   ├── display.py                # Prompt browser TUI
│   ├── content.py                # Prompt message content types
│   └── injection.py              # History injection with rollback on abort
├── resources/
│   ├── manager.py                # ResourceManager — browse resources from servers
│   ├── handler.py                # ResourceHandler — fetch content, handle images
│   └── parser.py                 # extract_resource_refs() — find @uri in queries
├── utils/
│   ├── constants.py              # Defaults, ASCII art, completion styles
│   ├── connection.py            # preflight_ollama() — verify Ollama is reachable
│   ├── streaming.py              # StreamingManager — token-by-token rendering
│   ├── tool_display.py           # ToolDisplayManager — JSON syntax highlighting
│   ├── hil_manager.py            # HumanInTheLoopManager — approve/skip/session/abort
│   ├── fzf_style_completion.py  # FZFStyleCompleter — /command fuzzy autocomplete
│   ├── history.py                # HistoryManager — export/import JSON chat logs
│   ├── metrics.py                # MetricsPanel — tokens/sec, timing display
│   ├── input.py                  # get_input_no_autocomplete() + read_single_keypress()
│   └── version.py                # check_for_updates()
└── build/lib/                    # sdist artifact (do not edit)
```

---

## Data Flow

```
User types query
        │
        ▼
parse_user_input()            # Split @uri resources, /prompt invocations, bare queries
        │
        ├─ @uri detected ──► ResourceHandler.fetch() ──► buffer to pending_resources
        │
        ├─ /server:prompt ──► PromptHandler.invoke() ──► argument collection
        │                                       ──► preview + confirm
        │                                       ──► inject into chat_history
        │
        └─ bare query ──────────────────────────────► pending_resources injected as context
                                                           │
                                                           ▼
                                                    MCPClient.process_query()
                                                           │
                                          ┌────────────────┴────────────────┐
                                          ▼                                 ▼
                               models/manager.py              tools/manager.py
                               Ollama AsyncClient              MCP tool calls
                                          │                                 │
                                          └────────────────┬────────────────┘
                                                           ▼
                                                   ToolDisplayManager
                                                   JSON syntax highlighting
                                                           │
                                          ┌────────────────┴────────────────┐
                                          ▼                                 ▼
                              HIL enabled?                    HIL disabled
                                          │                                 │
                                          ▼                                 ▼
                              HumanInTheLoopManager           Execute directly
                              approve / skip / session / abort
                                                           │
                                                           ▼
                                                   StreamingManager
                                                   token-by-token console output
                                                           │
                                                           ▼
                                                   MetricsPanel (if enabled)
                                                   tokens/sec, durations
```

---

## MCPClient Orchestration

`MCPClient` (in `client.py`) owns all state and wires every subsystem.

### Initialization

```python
def __init__(self, model, host):
    self.ollama = ollama.AsyncClient(host=host)
    self.server_connector = ServerConnector(...)
    self.model_manager = ModelManager(...)
    self.model_config_manager = ModelConfigManager(...)
    self.tool_manager = ToolManager(...)
    self.prompt_manager = PromptManager(...)
    self.prompt_handler = PromptHandler(...)
    self.resource_manager = ResourceManager(...)
    self.resource_handler = ResourceHandler(...)
    self.streaming_manager = StreamingManager(...)
    self.tool_display_manager = ToolDisplayManager(...)
    self.hil_manager = HumanInTheLoopManager(...)
    self.prompt_session = self._create_chat_prompt_session(...)
    self.chat_history = []
    self.pending_resources = []
    self.loop_limit = 3  # Agent Mode iterations
```

### Key State Flags

| Attribute | Default | Meaning |
|-----------|---------|---------|
| `retain_context` | `True` | Chat history retained between queries |
| `thinking_mode` | `True` | Send think/extra content to model |
| `show_thinking` | `True` | Display thinking text after generation |
| `show_tool_execution` | `True` | Show tool execution panels |
| `show_metrics` | `False` | Show performance metrics after query |
| `answer_render_mode` | `"both"` | Plain stream → Markdown render |
| `input_mode` | `"single"` | Single-line or multiline chat input |
| `loop_limit` | `3` | Max Agent Mode tool-call iterations |
| `abort_current_query` | `False` | Signal to cancel in-progress generation |

---

## Server Transport

`ServerConnector` handles three MCP transport types:

| Type | Use case | Key code path |
|------|----------|---------------|
| STDIO | Local Python/JS server scripts | `asyncio.create_subprocess_exec` |
| SSE | HTTP server, server-sent events | `ServerSentEvents` from `mcp` SDK |
| Streamable HTTP | Modern MCP 1.10.1+ servers | `StreamableHttp` from `mcp` SDK |

Configuration is parsed from JSON (`--servers-json`) or auto-discovered from Claude's `claude_desktop_config.json`.

---

## Tool Execution

1. `ToolManager` receives the list of enabled tools from all servers
2. `MCPClient.process_query()` sends the query + tool schemas to Ollama
3. Ollama returns a tool call → `ToolManager.execute_tool()` calls the MCP server
4. `ToolDisplayManager` renders the result with JSON syntax highlighting
5. If images returned and model is vision-capable → images attached to next LLM message
6. Result sent back to Ollama; repeat up to `loop_limit` times (Agent Mode)

---

## Human-in-the-Loop (HIL)

`HumanInTheLoopManager` intercepts every tool call:

| Option | Behavior |
|--------|----------|
| `y` / `yes` | Execute this tool call |
| `n` / `no` | Skip tool, continue query without it |
| `s` / `session` | Auto-approve all remaining tools this query |
| `d` / `disable` | Turn off HIL for the session |
| `a` / `abort` | Cancel entire query, do not save to history |

---

## Configuration Persistence

`ConfigManager` saves/loads `~/.config/ollmcp/config.json` with:

- Selected model + all `ModelConfigManager` parameters
- Enabled/disabled tools per server
- `thinking_mode`, `show_thinking`, `show_metrics`, `show_tool_execution`
- `answer_render_mode`, `input_mode`, `retain_context`
- `loop_limit`, HIL enabled/disabled

---

## Version

Current: **0.29.0** (defined in `pyproject.toml` and `mcp_client_for_ollama/__init__.py`)

Bump both packages with:
```bash
python scripts/bump_version.py patch  # 0.29.0 → 0.29.1
```