# CTX Layer Usage And Command Reference

This page keeps command detail out of the README while preserving the practical
workflow for users who want the full local loop.

## Requirements

- Python 3.11 or newer
- Git
- A terminal with access to the project repository

Optional:

- Codex or another coding agent that reads `AGENTS.md`
- Claude Code, Cursor, or another MCP-compatible client for tool integration
- A virtual environment, `pipx`, or another isolated Python environment

## Install

```powershell
python -m pip install --upgrade "https://github.com/abhilashsblai/ctxlayer-release/releases/download/v0.2.0a8/ctxlayer-0.2.0a8-py3-none-any.whl"
ctxlayer --version
```

Expected output:

```text
ctxlayer 0.2.0a8
```

Avoid creating a new `.venv` inside the target project before setup unless you
plan to exclude it from indexing. A virtual environment outside the project is
usually cleaner.

## Configure A Project

Run setup from the root of the repository where you want to use CTX Layer.

For a new folder that is not already a Git repository:

```powershell
ctxlayer --repo . setup --agents codex,claude,cursor --init-git --install-hooks --configure-mcp --absolute-mcp-repo
ctxlayer --repo . doctor
ctxlayer --repo . workflow recommend --task "initial setup smoke test"
```

For an existing Git repository, omit `--init-git`:

```powershell
ctxlayer --repo . setup --agents codex,claude,cursor --install-hooks --configure-mcp --absolute-mcp-repo
ctxlayer --repo . doctor
ctxlayer --repo . plan --help
ctxlayer --repo . workflow lint
```

Setup may create or update:

- `AGENTS.md`
- `CLAUDE.md`
- `.cursor/rules/ctxlayer.mdc`
- `ctxlayer.capabilities.yaml`
- `.ctxlayer/`
- `.claude/`
- `.cursor/mcp.json`
- `.mcp.json`
- local Git hooks, when `--install-hooks` is used
- MCP snippets under `.ctxlayer/mcp/`

Usually commit:

- `AGENTS.md`
- `CLAUDE.md`, if Claude Code setup is used
- `.cursor/rules/ctxlayer.mdc`, if Cursor setup is used
- `ctxlayer.capabilities.yaml`

Usually do not commit:

- `.ctxlayer/`
- machine-specific MCP snippets or local database files

`.ctxlayer/` contains local workspace state, databases, generated context, and
machine-specific MCP snippets.

## Daily Agent Workflow

After setup, the generated `AGENTS.md` asks agents to use this lifecycle. The
workflow-oriented path is:

```powershell
ctxlayer --repo . workflow recommend --task "<task>"
ctxlayer --repo . workflow start --task "<task>"
ctxlayer --repo . workflow next --task-session-id <task_session_id>
```

The legacy-compatible lower-level path is still available and remains useful
for manual scripts:

```powershell
ctxlayer --repo . task start --task "<task>"
ctxlayer --repo . pack --task "<task>"
```

Create a structured task plan:

```powershell
ctxlayer --repo . plan create --task-session-id <task_session_id> --pack-id <pack_id> --file <plan.json>
```

Checkpoint changed files:

```powershell
ctxlayer --repo . plan checkpoint <plan_id> --step-id <step_id> --path <changed_path>
```

Run preflight checks before declaring work complete:

```powershell
git diff HEAD | ctxlayer --repo . check-diff --pack-id <pack_id>
git diff --name-only HEAD | ctxlayer --repo . check-diff --pack-id <pack_id> --paths-from-stdin
git diff --name-only HEAD | ctxlayer --repo . impact --paths-from-stdin
```

Record the outcome:

```powershell
ctxlayer --repo . outcome --pack-id <pack_id> --result pass --summary "<what changed and why>"
```

The outcome summary becomes durable project memory that future tasks can
retrieve.

## Common Commands

Workflow:

```powershell
ctxlayer --repo . workflow recommend --task "describe the change"
ctxlayer --repo . workflow start --task "describe the change" --mode write
ctxlayer --repo . workflow next --task-session-id <task_session_id>
ctxlayer --repo . workflow lint
python -m ctxlayer.workflow.export_features --check
```

Project health:

```powershell
ctxlayer --repo . doctor
ctxlayer --repo . index
ctxlayer --repo . gc
ctxlayer --repo . db-stats
ctxlayer --repo . gc --deep --dry-run
ctxlayer --repo . gc --deep
ctxlayer --repo . maintenance history
```

Context, impact, and search:

```powershell
ctxlayer --repo . pack --task "describe the change"
ctxlayer --repo . impact --paths src/app/service.py
git diff --name-only HEAD | ctxlayer --repo . check-diff --pack-id <pack_id> --paths-from-stdin
ctxlayer --repo . search "business rule"
```

Memory and business knowledge:

```powershell
ctxlayer --repo . memory candidates
ctxlayer --repo . memory list
ctxlayer --repo . business entity list
ctxlayer --repo . business link list
```

Learning and CIE:

```powershell
ctxlayer --repo . learning status
ctxlayer --repo . learning loop-verify --all
ctxlayer --repo . cie status
ctxlayer --repo . cie cycle --task "review recent work"
ctxlayer --repo . cie score
```

Decision support:

```powershell
ctxlayer --repo . decision start --question "Which approach should we use?" --option "A" --option "B"
ctxlayer --repo . decision analyze <decision_id>
ctxlayer --repo . decision record <decision_id> --chosen <option_id> --rationale "why"
```

Security, semantic, and org knowledge:

```powershell
ctxlayer --repo . security scan
ctxlayer --repo . semantic index
ctxlayer --repo . semantic search "checkout flow"
ctxlayer --repo . org ingest --source local --path docs
```

Dashboard, rollup, and MCP:

```powershell
ctxlayer --repo . dashboard render --output .ctxlayer/dashboard.html
ctxlayer --repo . rollup --output .ctxlayer/rollup.json
ctxlayer --repo . mcp
```

HTTP server:

```powershell
ctxlayer --repo . serve --host 127.0.0.1 --port 8765 --auth token
```

## MCP Integration

Setup can generate MCP snippets for supported clients:

```powershell
ctxlayer --repo . setup --agents codex,claude,cursor --configure-mcp --absolute-mcp-repo
```

The MCP server exposes the same local project intelligence used by the CLI:
context packs, impact, memory, learning, workflow routing, CIE, decision
support, dashboard data, and governance flows.

For local use:

```powershell
ctxlayer --repo . mcp
```

Review generated `.ctxlayer/mcp/` files before copying them into a client
configuration.

## CI And Release Checks

Recommended CI checks for source repositories using the current workflow
system:

```powershell
ctxlayer --repo . ci check
ctxlayer --repo . workflow lint
python -m ctxlayer.workflow.export_features --check
```

`workflow lint` catches broken catalog/playbook/router contracts. The workbook
check catches uncommitted drift between the source catalog and
`features/features.xlsx`.
