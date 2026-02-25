# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BrowserWing is a native browser automation platform with AI integration. It exposes 26+ HTTP API endpoints for browser control and supports three integration modes: HTTP API, MCP (Model Context Protocol), and Skills protocol.

## Common Commands

### Development
```bash
make install          # Install all frontend and backend dependencies
make dev              # Start both frontend and backend in dev mode
make backend          # Run backend only (port 8080)
make frontend         # Run frontend only (port 5173)
```

### Build
```bash
make build-embedded   # Build single binary with embedded frontend (current platform)
make build-all        # Build for all platforms (Linux/macOS/Windows, amd64+arm64)
```

### Test & Format
```bash
make test             # Run Go tests
make fmt              # Format Go code
cd frontend && pnpm lint              # Lint frontend
cd frontend && pnpm check-translations # Check i18n translation files
```

### Frontend (in `frontend/`)
```bash
pnpm dev              # Dev server
pnpm build            # Production build (tsc + vite)
```

## Architecture

### Backend (Go)
- `backend/main.go` — Entry point; initializes all modules
- `backend/api/` — Gin HTTP router and handlers
- `backend/executor/` — Core browser automation engine (rod-based); handles element finding via RefID (`@e1`, `@e2`...), accessibility tree extraction, and all browser operations
- `backend/mcp/` — MCP server implementation (HTTP + Streamable HTTP at `/api/v1/mcp/message`)
- `backend/agent/` — AI agent session management
- `backend/services/browser/` — Browser instance lifecycle (Local Chrome launch or Remote CDP)
- `backend/llm/` — Multi-provider LLM integration (OpenAI, Claude, DeepSeek, etc.)
- `backend/scheduler/` — Cron-based task scheduling
- `backend/storage/` — BoltDB embedded KV store (`./data/browserwing.db`)
- `backend/models/` — Shared data models

Key dependencies: `go-rod/rod` (CDP automation), `gin-gonic/gin` (HTTP), `mark3labs/mcp-go` (MCP), `bbolt` (storage), `gotoailab/llmhub` (LLM).

### Frontend (React + TypeScript)
- `frontend/src/pages/` — Page-level components
- `frontend/src/components/` — Reusable UI components
- `frontend/src/api/` — Axios-based API client
- `frontend/src/contexts/` — React Context providers
- `frontend/src/i18n/` — Internationalization (zh, en, ja, es, pt)

Built with Vite 5, TailwindCSS, React Router. In production, the frontend is embedded into the Go binary via `go:embed`.

### Key Design Concepts
- **RefID mechanism**: The executor assigns stable element references (`@e1`, `@e2`...) via the accessibility tree, enabling AI tools to reference elements without fragile CSS selectors.
- **Accessibility-first**: Page analysis uses the accessibility tree (not raw DOM) for AI-friendly element representation.
- **Multi-instance browsers**: Supports multiple named browser instances, each with independent config (proxy, user data dir, CDP endpoint).

## Configuration

`config.toml` at project root (auto-created on first run):
```toml
[server]
port = '8080'

[browser]
bin_path = '/path/to/chrome'
user_data_dir = './chrome_user_data'

[auth]
enabled = false
```

## API Structure

All browser control endpoints are under `/api/v1/executor/`. Key operations: `navigate`, `click`, `type`, `select`, `snapshot` (accessibility tree), `screenshot`, `evaluate` (JS), `batch`, `extract`, `fill-form`, `tabs`.

MCP endpoint: `POST /api/v1/mcp/message`
