# AGENTS.md - mcp-servers

Canonical assistant policy for this repository. Read by Codex, Claude Code (via CLAUDE.md), Copilot, Cursor, Devin, Windsurf, and other coding agents.

## What This Repo Is

mcp-servers provides [Model Context Protocol (MCP)](https://modelcontextprotocol.io) servers that expose Eclipse S-CORE build, test, and project tooling to AI coding agents. Instead of agents guessing how to build or test S-CORE repos, they connect to these MCP servers and call tools directly.

This repo is NOT safety-critical code. It is developer tooling. Safety rules (ISO 26262, MISRA, AUTOSAR AP) do not apply here - they apply to the repos these MCP servers wrap.

### Context

- Eclipse S-CORE is a multi-company automotive onboard platform (C++/Rust/Bazel).
- Multiple companies contribute using different AI tools (Copilot, Devin, Claude Code, Cursor, etc.).
- Eclipse Foundation governance: Apache 2.0 license, Eclipse DCO, IP review for contributions >1000 lines.
- MCP is the industry-standard protocol for agent-to-tool integration.

## Prerequisites

- Python 3.11+ (primary implementation language)
- [uv](https://docs.astral.sh/uv/) (package manager)
- Git
- For integration tests: Bazel (to test against real S-CORE repos)

## Commands

```sh
# Install dependencies
uv sync --all-groups

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_bazel_server.py -xvs

# Lint
uv run ruff check src/ tests/

# Format
uv run ruff format src/ tests/

# Type check
uv run basedpyright src/

# Run the MCP server locally (for development)
uv run score-mcp-server
```

## Repository Structure

```text
mcp-servers/
|- src/
|  `- score_mcp_server/
|     |- __init__.py
|     |- server.py            - MCP server entry point and lifecycle
|     |- tools/               - Tool implementations (one file per tool group)
|     |  |- __init__.py
|     |  |- bazel.py          - build, test, query, coverage tools
|     |  |- lint.py           - lint and format tools
|     |  `- project.py        - repo discovery, manifest reading
|     |- config.py            - Server configuration and defaults
|     `- manifest.py          - repo-manifest.json parser
|- tests/
|  |- unit/                   - Unit tests (mocked, no Bazel required)
|  `- integration/            - Integration tests (require Bazel + real repo)
|- .github/
|  |- instructions/           - SCORE coding standards (from governance overlay)
|  |- references/             - Schemas (repo-manifest, agent-card)
|  |- score/                  - repo-manifest.json for THIS repo
|  `- workflows/              - CI/CD
|- pyproject.toml             - Project metadata and dependencies
|- AGENTS.md                  - This file
|- CLAUDE.md                  - Claude Code import -> AGENTS.md
|- AI_CONTRIBUTION_POLICY.md  - AI disclosure and accountability rules
`- README.md                  - User-facing documentation
```

## MCP Server Architecture

### Design Principles

1. One server, multiple tool groups. A single mcp-servers process exposes all tools. Tool groups (bazel, lint, project) are logical groupings, not separate servers.
2. Manifest-driven. Tools read .github/score/repo-manifest.json from the target repo to discover build/test/lint commands. Do not hardcode commands.
3. Stateless. Each tool call is independent. No session state between calls.
4. Safe by default. Tools that modify state (build, test) must operate in the caller's working directory. Never write outside the repo root.

### Tool Naming Convention

Tool names follow <group>_<action> pattern:

```text
bazel_build       - Build a Bazel target
bazel_test        - Run tests for a Bazel target
bazel_query       - Query the dependency graph
bazel_coverage    - Run tests with coverage
lint_check        - Run linter
lint_format       - Run formatter
project_manifest  - Read and return the repo-manifest.json
project_discover  - List available repos and their manifests
```

### Adding a New Tool

1. Create or edit the appropriate file in src/score_mcp_server/tools/
2. Implement the tool function with MCP tool decorator
3. Add unit tests in tests/unit/
4. Add integration test if the tool calls external programs
5. Update this AGENTS.md if adding a new tool group

## Coding Standards

### Python

- Python 3.11+ - use modern syntax (match/case, X | Y union types, etc.)
- Format with ruff format, lint with ruff check
- Type annotations on all public functions and methods
- Type check with basedpyright - zero errors
- No Any types unless wrapping genuinely untyped external APIs (document why)
- Async by default for MCP handlers (MCP protocol is async)

### General

- Instruction files in .github/instructions/ apply (coding style, security, testing, git workflow)
- 80% test coverage minimum
- No hardcoded secrets - use environment variables
- All public APIs documented with docstrings

## AI Disclosure

All AI-assisted commits MUST include an Assisted-by: trailer:

```text
feat: add bazel_coverage tool

Implement coverage collection via bazel coverage command
with lcov output parsing.

Assisted-by: GitHub Copilot <noreply@github.com>
```

Standard trailers:

| Tool | Trailer |
|------|---------|
| GitHub Copilot | Assisted-by: GitHub Copilot <noreply@github.com> |
| Claude Code | Assisted-by: Claude Code <noreply@anthropic.com> |
| Cursor | Assisted-by: Cursor <noreply@cursor.com> |
| Devin | Assisted-by: Devin <noreply@cognition.ai> |
| Codex | Assisted-by: Codex <noreply@openai.com> |
| Windsurf | Assisted-by: Windsurf <noreply@codeium.com> |

See AI_CONTRIBUTION_POLICY.md for full disclosure rules, scope, and legal details.

### Commit Format

```text
<type>: <summary>

<optional body>

Assisted-by: <tool> <email>   <- only if AI-assisted
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

## PR Conventions

- Reference an issue: Fixes #<number> or Relates to #<number>
- Check the AI disclosure box in the PR template if AI tools were used
- All CI must pass before review
- One approval required

## SCORE Governance

- .github/score/repo-manifest.json - machine-readable build/test/lint for this repo
- .github/references/repo-manifest.schema.json - schema for repo manifests
- .github/references/agent-card.schema.json - schema for work/handoff artifacts (not A2A AgentCard)
- Keep this repo focused on MCP server implementation - no prompt catalogs or agent frameworks
