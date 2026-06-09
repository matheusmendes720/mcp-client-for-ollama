# Documentation Index

| Document | Audience | What it covers |
|----------|---------|---------------|
| [README.md](README.md) | All users | Installation, features, usage, CLI options, interactive commands, MCP setup, compatible models |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Developers | Module map, data flow diagrams, class responsibilities, transport types, state flags |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contributors | Setup, testing, coding standards, branch strategy, release process, common tasks |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | All users | Installation errors, runtime errors, configuration issues, dev environment fixes |
| [CHANGELOG.md](CHANGELOG.md) | All users | Version history and notable changes |
| [DEV.md](DEV.md) | Maintainers | Version bumping script, GitHub Actions release workflow, PyPI publishing |
| [SECURITY.md](SECURITY.md) | All users | Responsible disclosure policy |
| `pyproject.toml` | Developers | Dependencies, entry points, package metadata |
| `cli-package/README.md` | Package users | `ollmcp` PyPI package details |
| `.github/workflows/` | CI/CD | Automated testing, building, publishing |

## Decision Tree: Where to Read

```
I want to...
├── Run ollmcp for the first time          → README.md
├── Understand how it works               → ARCHITECTURE.md
├── Set up a dev environment               → CONTRIBUTING.md
├── Fix an error I'm seeing                → TROUBLESHOOTING.md
├── Know what changed in a version         → CHANGELOG.md
├── Publish a new release                  → DEV.md + CONTRIBUTING.md
└── Report a security issue               → SECURITY.md
```
