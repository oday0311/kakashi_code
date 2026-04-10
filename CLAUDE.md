# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a fork of Anthropic's Claude Code CLI, reconstructed from the npm distribution source map (March 31, 2026). The fork removes all telemetry (OpenTelemetry, GrowthBook analytics, Sentry, custom event logging), strips security-prompt guardrails injected by Anthropic, and enables 45+ experimental features that are disabled in the public release.

The project is a TypeScript/Bun terminal application using React + Ink for the UI. It implements the Claude Code agent with slash commands, tools, skills, and MCP integration.

## Requirements

- Bun >=1.3.11
- macOS or Linux (Windows via WSL)
- Anthropic API key (`ANTHROPIC_API_KEY` environment variable) or Claude.ai OAuth

## Common Commands

### Development
```bash
# Run from source (slower startup)
bun run dev

# Build the production binary (./cli) with VOICE_MODE only
bun run build

# Build the dev binary (./cli-dev) with dev version stamp
bun run build:dev

# Build the dev binary with ALL experimental features enabled
bun run build:dev:full

# Alternative output path (./dist/cli)
bun run compile
```

### Running the binary
```bash
# Run the built binary
./cli

# One-shot mode with a prompt
./cli -p "list files"

# Specify model
./cli --model claude-sonnet-4-6-20250514

# Login via Claude.ai OAuth
./cli /login
```

### Feature flag builds
```bash
# Enable specific features on top of default build
bun run ./scripts/build.ts --feature=ULTRAPLAN --feature=ULTRATHINK

# Enable a flag on top of dev build
bun run ./scripts/build.ts --dev --feature=BRIDGE_MODE
```

## Build Variants

| Command | Output | Features | Notes |
|---------|--------|----------|-------|
| `bun run build` | `./cli` | `VOICE_MODE` only | Production-like binary |
| `bun run build:dev` | `./cli-dev` | `VOICE_MODE` only | Dev version stamp |
| `bun run build:dev:full` | `./cli-dev` | All 45+ experimental flags | Full unlock build |
| `bun run compile` | `./dist/cli` | `VOICE_MODE` only | Alternative output directory |

## Feature Flags

The project uses compile-time feature flags gated by `feature('FLAG')` calls. The build script (`scripts/build.ts`) can enable flags individually or in sets. See `FEATURES.md` for a full audit of 88 flags and their status.

**Notable experimental features:**
- `ULTRAPLAN`: Remote multi-agent planning on Claude Code web (Opus-class)
- `ULTRATHINK`: Deep thinking mode — type "ultrathink" to boost reasoning effort
- `VOICE_MODE`: Push-to-talk voice input and dictation
- `AGENT_TRIGGERS`: Local cron/trigger tools for background automation
- `BRIDGE_MODE`: IDE remote-control bridge (VS Code, JetBrains)
- `TOKEN_BUDGET`: Token budget tracking and usage warnings
- `BUILTIN_EXPLORE_PLAN_AGENTS`: Built-in explore/plan agent presets
- `VERIFICATION_AGENT`: Verification agent for task validation
- `BASH_CLASSIFIER`: Classifier-assisted bash permission decisions
- `EXTRACT_MEMORIES`: Post-query automatic memory extraction

## Project Structure

```
scripts/
  build.ts              # Build script with feature flag system

src/
  entrypoints/cli.tsx   # CLI entrypoint
  commands.ts           # Command registry (slash commands)
  tools.ts              # Tool registry (agent tools)
  QueryEngine.ts        # LLM query engine
  screens/REPL.tsx      # Main interactive UI

  commands/             # /slash command implementations
  tools/                # Agent tool implementations (Bash, Read, Edit, etc.)
  components/           # Ink/React terminal UI components
  hooks/                # React hooks
  services/             # API client, MCP, OAuth, analytics
  state/                # App state store
  utils/                # Utilities
  skills/               # Skill system
  plugins/              # Plugin system
  bridge/               # IDE bridge
  voice/                # Voice input
  tasks/                # Background task management
```

## Key Architectural Notes

- **Telemetry removed**: All outbound telemetry endpoints are dead-code-eliminated or stubbed. GrowthBook feature flag evaluation works locally but does not report back.
- **Security-prompt guardrails removed**: Injected system-level instructions that constrain Claude's behavior beyond the model's own safety training are stripped.
- **Experimental features enabled**: All 45+ feature flags that compile cleanly are enabled in the `build:dev:full` variant.
- **Binary output**: The built binary is named `cli` (production) or `cli-dev` (development). The release directory `kakaxi_release` contains a copy of the binary.

## Development Workflow

1. Make changes in the source (`src/`).
2. Test with `bun run dev` for quick iteration.
3. Build a binary with desired feature set using the appropriate build command.
4. Test the binary directly (`./cli` or `./cli-dev`).
5. Copy the binary to the release directory if needed.

## References

- README.md – Quick install, overview, and usage
- FEATURES.md – Complete audit of feature flags and their status
- package.json – Dependencies and script definitions