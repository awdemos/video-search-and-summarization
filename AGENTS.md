# Agent Guidelines for NVIDIA AI Blueprint: Video Search and Summarization

This repo implements the NVIDIA AI Blueprint for Video Search and Summarization (VSS). It is a multi-service reference architecture combining vision-language models, RAG, NVIDIA NIM microservices, and agentic tools.

## Project Layout

```
services/agent/          # Python VSS agent (src/vss_agents/, tests/, stubs/)
services/ui/             # Next.js / Turbo monorepo frontend
services/analytics/      # Downstream analytics services
deploy/docker/           # Docker Compose services and developer profiles
deploy/helm/             # Helm charts
skills/                  # NVIDIA skill definitions and eval harness
assets/                  # Architecture diagrams and static assets
```

## Critical Rules

1. **Two independent codebases.** The Python agent and the UI monorepo have separate dependency and lint tooling. Do not mix commands from one in the other.
2. **DCO + SPDX headers are required.** Every commit must include `Signed-off-by:` (pre-commit enforces it), and Python files need SPDX copyright headers. Use `git commit -s`.
3. **License checks are gated.** Python deps use `services/agent/.github/scripts/check_python_licenses.sh`; UI deps use `check_ui_licenses.sh`. Avoid adding GPL/AGPL/SSPL/BUSL dependencies.
4. **Skills are evaluated automatically.** Changes under `skills/` may trigger the daily skill eval workflow; keep skill adapters deterministic.

## Agent Service — Build / Test / Lint

All commands assume `cd services/agent`.

Prerequisites: Python 3.13+, `uv`.

```bash
# Install
uv sync

# Tests
uv run pytest tests -q

# Lint
uv run ruff check .
uv run ruff format --check .

# Type check
uv run mypy src/vss_agents/
```

## UI Service — Build / Test / Lint

All commands assume `cd services/ui`.

Prerequisites: Node.js version in `.nvmrc` (use `nvm install && nvm use`), `npm`.

```bash
# Install
npm install

# Build all packages
npx turbo run build

# Lint / format
npx turbo run lint
npx turbo run format -- --check

# Tests
npx turbo run test

# Dev server
npm run dev
```

## Pre-commit

```bash
# From repo root
pre-commit install --hook-type pre-commit --hook-type commit-msg
pre-commit run --all-files
```

## Running Locally (Docker)

```bash
cd deploy/docker
# Follow the README there; developer profiles under developer-profiles/ configure the NIM endpoints.
```

## Gotchas

- `services/agent/` and `services/ui/` have separate `.github/workflows` steps but share the root `CONTRIBUTING.md` and DCO requirement.
- SPDX headers are checked by `.github/scripts/check_copyright_headers.py`; missing headers block CI.
- The `agent` service pins many dependencies in `uv.lock`; run `uv lock` after changing `pyproject.toml`.
- UI apps depend on `nemo-agent-toolkit-ui` shared packages; changes there may require a full monorepo rebuild.
