# Everything Claude Code (ECC) — Codebase Research

## What It Is

Everything Claude Code (ECC) is a comprehensive, production-ready **plugin for Claude Code** (and other AI coding harnesses like Cursor, Codex, and OpenCode). It packages a curated collection of agents, skills, hooks, commands, rules, and MCP server configurations that transform a bare Claude Code installation into a structured, opinionated software development environment.

Created by Affaan Mustafa, an Anthropic hackathon winner, the project has evolved over 10+ months of intensive daily use. It is published as `ecc-universal` on npm (v1.9.0 as of March 2026) and licensed under MIT.

**Repository:** https://github.com/affaan-m/everything-claude-code

---

## How It Works

ECC operates as a layered system of composable components that plug into Claude Code's extension points. Each layer serves a distinct role in the development workflow.

### 1. Agents (28 total)

Specialized subagents that Claude Code delegates tasks to. Each agent is a Markdown file with YAML frontmatter (`name`, `description`, `tools`, `model`) and a structured body defining its workflow, review priorities, and diagnostic commands.

**Categories:**
- **Language-specific reviewers** — TypeScript, Python, Go, Rust, Java, Kotlin, C++ code review agents with deep knowledge of language idioms, security patterns, and framework best practices.
- **Language-specific build resolvers** — Agents that diagnose and fix build failures for each language ecosystem.
- **General purpose** — Planner (implementation planning), architect (system design), TDD guide, E2E runner (Playwright), code reviewer, security reviewer, refactor-cleaner, doc-updater, docs-lookup, database reviewer, and more.
- **Operational** — Chief-of-staff (multi-channel comms triage), loop-operator (autonomous monitoring), harness-optimizer (configuration tuning).

**How delegation works:** When a user invokes a command like `/code-review` or `/typescript-review`, Claude Code loads the corresponding agent definition and adopts its specialized role, priorities, and workflow.

### 2. Skills (116 total)

Domain knowledge modules that provide context, patterns, and best practices. Each skill lives in its own directory as a `SKILL.md` file with sections: name/description, core concepts, when to activate, code examples, and best practices.

**Categories:**
- **Coding standards** — TypeScript/JavaScript/React/Node.js patterns, immutability, error handling, testing, performance.
- **Backend/frontend patterns** — API design, database patterns, caching, auth, React, Next.js, state management.
- **Language/framework specific** — PyTorch, Bun runtime, Next.js Turbopack, MCP server patterns.
- **Operational** — Claude API integration, multi-agent orchestration (devfleet), autonomous loops, verification loops, prompt optimization.
- **Domain-specific** — Article writing, content engine, market research, deep research, video editing, data scraping, investor materials, social media (X API).

Skills are activated contextually — Claude Code loads the relevant skill when it detects matching imports, frameworks, or user intent.

### 3. Commands (59 total)

Slash commands that users invoke directly (e.g., `/tdd`, `/plan`, `/e2e`). Each command is a Markdown file with a description in frontmatter and a prompt body that orchestrates one or more agents and skills.

**Key workflows:**
- `/tdd` — Test-driven development (red-green-refactor cycle)
- `/plan` — Implementation planning with risk analysis and phased delivery
- `/code-review` — Multi-dimensional quality review
- `/build-fix` — Diagnose and fix build errors
- `/e2e` — Generate and run Playwright end-to-end tests
- `/learn` — Extract reusable patterns from the current session
- `/skill-create` — Generate new skills from git history
- `/loop-start`, `/loop-status` — Autonomous recurring task execution
- `/orchestrate`, `/devfleet` — Multi-agent parallel delegation
- `/checkpoint`, `/save-session`, `/resume-session` — Session management
- Language-specific commands — `/typescript-review`, `/python-review`, `/go-review`, etc.

### 4. Hooks (20+ lifecycle hooks)

Event-driven automations defined in `hooks/hooks.json`. Hooks fire at specific lifecycle events and run JavaScript or shell scripts from `scripts/hooks/`.

**Lifecycle events:**
| Event | Purpose | Example Hooks |
|-------|---------|---------------|
| `PreToolUse` | Block or warn before tool execution | Block `--no-verify` on git, auto-start dev servers in tmux, git push review reminder |
| `PostToolUse` | React after tool execution | Log PR URLs, auto-format with Prettier, TypeScript type-checking, console.log warnings |
| `PreCompact` | Before context compaction | Save state snapshot |
| `SessionStart` | Session initialization | Load previous context, detect package manager |
| `Stop` | After each response | Check for debug statements, persist state, extract patterns, track token costs |
| `SessionEnd` | Session closure | Final state persistence |

**Hook profiles** (`ECC_HOOK_PROFILE`): `minimal`, `standard` (default), or `strict` — controls which hooks are active. Individual hooks can be disabled via `ECC_DISABLED_HOOKS`.

### 5. Rules (12 languages × 5 categories)

Language-specific coding guidelines stored in `rules/<language>/`. Each language has five rule files:

1. `coding-style.md` — Formatting, naming conventions, file organization
2. `patterns.md` — Idiomatic patterns and common pitfalls
3. `testing.md` — Testing strategy and coverage requirements
4. `security.md` — Vulnerability patterns and secret management
5. `hooks.md` — Language-specific hook configurations

**Supported languages:** TypeScript, Python, Go, Rust, C++, Java, Kotlin, Swift, C#, Perl, PHP, plus a `common/` set of universal rules.

### 6. MCP Configurations (19+ servers)

Pre-configured Model Context Protocol servers in `mcp-configs/mcp-servers.json`:

- **Code/Git** — GitHub operations
- **Data** — Supabase, ClickHouse, SQLite
- **Search** — Exa web search, Context7 documentation lookup
- **Browsers** — Playwright, Browserbase, Browser-Use
- **Media** — FAL.ai (image/video/audio generation)
- **Scraping** — Firecrawl
- **Infrastructure** — Vercel, Railway, Cloudflare Workers
- **Security** — InsAIts (23 anomaly types, OWASP MCP Top 10)
- **Reasoning** — Sequential-thinking
- **Optimization** — Token-optimizer

Each entry includes command/args, required environment variables, and usage guidance. The project recommends keeping under 10 MCPs enabled to preserve context window.

### 7. Installation System

ECC uses a modular, profile-based installation system:

**Profiles:**
- `core` — Minimal baseline (rules, core agents, core commands, hooks)
- `developer` — Default for most users (core + framework/language/database support)
- `security` — Security-focused setup
- `research` — Research and content-oriented
- `full` — Everything installed

**Mechanics:**
- `install.sh` (bash) / `install.ps1` (PowerShell) resolve symlinks and delegate to `scripts/install-apply.js`
- Manifests in `manifests/` define modules, profiles, and components with dependency tracking
- SQLite state store tracks what is installed, enabling `doctor` (diagnose drift), `repair` (restore drifted files), and `uninstall` operations
- Supports 6 install targets: Claude Code (default), Cursor, Codex, OpenCode, Antigravity, and project-local

### 8. Scripts and Utilities

The `scripts/` directory contains the project's runtime infrastructure:

- **CLI entrypoint** (`ecc.js`) — Dispatcher for install, plan, doctor, repair, status, sessions, uninstall
- **Hook implementations** (`scripts/hooks/`) — JavaScript handlers for each lifecycle hook
- **CI validators** (`scripts/ci/`) — Validate agents, skills, rules, commands, hooks, and install manifests for structural correctness
- **Core library** (`scripts/lib/`) — Package manager detection, install orchestration, target-specific handlers, tmux/worktree management

### 9. Testing

Tests live in `tests/` and run via `node tests/run-all.js`. The test suite covers:

- Hook validation and individual hook behavior (15+ hook test files)
- Utility functions (package manager detection, general utils)
- CI validators
- Integration tests

Code coverage is tracked with c8 with an 80% minimum threshold.

### 10. Cross-Platform / Cross-Harness Support

ECC is not limited to Claude Code. It provides configurations for:

- **Cursor** (`.cursor/`) — IDE-specific settings
- **Codex** (`.codex/`) — OpenAI Codex platform
- **OpenCode** (`.opencode/`) — TypeScript/Node.js-based commands and plugins
- **Antigravity** — IDE support
- **OpenAI agents** (`.agents/`) — YAML agent definitions for cross-platform compatibility

---

## Architecture Summary

```
everything-claude-code/
├── agents/              # 28 specialized subagents (Markdown + YAML frontmatter)
├── skills/              # 116 domain knowledge modules (SKILL.md per skill)
├── commands/            # 59 slash commands (Markdown with description frontmatter)
├── hooks/               # Event-driven automations (hooks.json + JS handlers)
├── rules/               # 12 languages × 5 rule categories
├── mcp-configs/         # 19+ MCP server configurations
├── scripts/
│   ├── ecc.js           # CLI entrypoint
│   ├── hooks/           # Hook handler implementations
│   ├── ci/              # CI validation scripts
│   └── lib/             # Core utilities (install, package manager, etc.)
├── manifests/           # Install profiles, modules, and components
├── schemas/             # JSON schemas for validation
├── tests/               # Test suite (20+ test files, 80% coverage minimum)
├── contexts/            # Project context configurations (dev, research, review)
├── examples/            # Template CLAUDE.md files for various project types
├── docs/                # Additional documentation
├── .claude/             # Claude Code project metadata and generated configs
├── .claude-plugin/      # Plugin marketplace metadata
├── .agents/             # OpenAI-compatible agent definitions
├── .cursor/             # Cursor IDE configurations
├── .codex/              # Codex platform integration
├── .opencode/           # OpenCode platform support
├── install.sh           # Unix installer
├── install.ps1          # Windows installer
└── package.json         # npm package (ecc-universal v1.9.0)
```

---

## Use Cases

### 1. Individual Developer Productivity
A solo developer installs ECC to get immediate access to structured workflows — TDD cycles, implementation planning, code review, build fixing — without manually configuring each workflow. The hook system automatically enforces quality (formatting, type-checking, no debug statements) on every interaction.

### 2. Team Standardization
Teams install ECC to enforce consistent coding standards, security practices, and review processes across all developers using Claude Code. The rules system ensures every team member's AI assistant follows the same conventions for naming, patterns, testing, and security across 12 supported languages.

### 3. Multi-Language Projects
Projects spanning multiple languages (e.g., TypeScript frontend + Go backend + Python ML pipeline) benefit from ECC's language-specific agents, reviewers, and build resolvers that understand each ecosystem's idioms and tooling.

### 4. Security-Conscious Development
The security review agent, InsAIts MCP integration, and security rules provide defense-in-depth: blocking `--no-verify` on git hooks, scanning for OWASP vulnerabilities, enforcing secret management practices, and validating inputs at system boundaries.

### 5. Autonomous Workflows
The loop operator, checkpoint system, and session persistence enable long-running autonomous tasks — monitoring deployments, running recurring quality checks, or executing multi-step migrations across sessions.

### 6. Multi-Agent Orchestration
Commands like `/devfleet` and `/orchestrate` enable parallel delegation to multiple specialized agents, useful for large-scale refactors, multi-service coordination, or simultaneous frontend/backend development.

### 7. Learning and Pattern Extraction
The `/learn` command and `evaluate-session` hook automatically extract reusable patterns from development sessions, turning ad-hoc solutions into documented skills for future use.

### 8. Project Bootstrapping
The `examples/` directory provides template CLAUDE.md files for common project types (Django API, Go microservice, Laravel API, Rust API, Next.js SaaS), giving new projects a head start with appropriate conventions.

### 9. CI/CD Integration
The CI validation scripts ensure that all ECC components (agents, skills, rules, commands, hooks) maintain structural integrity. The harness audit scores reliability. These integrate into existing CI pipelines via npm scripts.

### 10. Cross-Harness Portability
Organizations using multiple AI coding tools (Claude Code, Cursor, Codex, OpenCode) can use ECC as a single source of truth for development conventions, with target-specific install handlers adapting the configuration for each platform.

---

## Core Design Principles

1. **Agent-first** — Delegate to specialized agents rather than relying on generic prompts.
2. **Test-driven** — Write tests before implementation; enforce via TDD workflow commands.
3. **Security-first** — Never compromise on security; validate at system boundaries.
4. **Immutability** — Create new objects; never mutate existing data.
5. **Plan before execute** — Use planning agents for complex features before writing code.
6. **Continuous learning** — Auto-extract patterns from sessions into reusable skills.
7. **Hook-driven automation** — Event-based validation and quality gates, not manual checks.
8. **State persistence** — SQLite state store for cross-session memory and install tracking.
9. **Cross-platform** — Windows, macOS, Linux support via Node.js scripts.
10. **Modular installation** — Install only what you need via profiles and selective modules.

---

## Statistics

| Component | Count |
|-----------|-------|
| Agents | 28 |
| Skills | 116 |
| Commands | 59 |
| Rules (languages) | 12 |
| Rule categories per language | 5 |
| MCP server configs | 19+ |
| Lifecycle hook events | 6 types, 20+ individual hooks |
| Install targets | 6 platforms |
| Install profiles | 5 |
| Test files | 20+ |
| Supported languages | 12 |
