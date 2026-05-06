# docker-codex

Docker images for **OpenAI Codex CLI** (`@openai/codex`) — run the powerful terminal-based AI coding agent in a clean, reproducible, isolated environment.

## Overview

This project provides ready-to-use Docker images with Node.js and the official [OpenAI Codex CLI](https://developers.openai.com/codex/cli) pre-installed.

**Why use these images?**

- No need to install Node.js or Codex globally on your host machine
- Consistent development environment across machines and team members
- Perfect for VS Code Dev Containers
- Non-root user (`node`) for better security
- Pre-installed useful tools (`git`, `curl`, `ca-certificates`, `bubblewrap`, etc.)
- Persistent configuration and cache via volumes

## Available Images

| Tag                        | Base Image                          | Description                          |
|---------------------------|-------------------------------------|--------------------------------------|
| `0-node` (latest)         | `node:22-bookworm-slim`            | Minimal Node.js environment         |
| `0-javascript-node`       | Microsoft Dev Container (JS)       | JavaScript-focused dev environment  |
| `0-typescript-node`       | Microsoft Dev Container (TS)       | TypeScript-focused dev environment  |

> Replace `0-` with the specific version (e.g. `0.128-`) for pinned versions.

## Quick Start

### 1. Using Docker Compose (Recommended)

Copy the provided `docker-compose.yaml`:

```yaml
services:
  codex:
    image: naoyoshinori/codex:0-node
    working_dir: /workspace
    volumes:
      - ~/.gitconfig:/home/node/.gitconfig
      - ~/.codex_cli:/home/node/.codex:cached
      - .:/workspace:cached
    command: ["sleep", "infinity"]
```

Start the container:

```bash
docker compose up -d
docker compose exec codex bash
```

### 2. One-liner (Manual)

```bash
docker run -it --rm \
  -v "$(pwd):/workspace" \
  -v ~/.codex_cli:/home/node/.codex \
  -v ~/.gitconfig:/home/node/.gitconfig \
  -w /workspace \
  naoyoshinori/codex:0-node bash
```

### 3. Inside the Container

```bash
# Set your API key (first time only)
export OPENAI_API_KEY=sk-...

# Run Codex
codex
```

Codex will guide you through authentication and setup if needed.

## Examples

### Example 1: Start a new project

```bash
mkdir my-ai-project && cd my-ai-project
docker compose up -d
docker compose exec codex bash

# Inside container:
codex "Create a FastAPI backend with PostgreSQL and basic CRUD for tasks"
```

### Example 2: Work on existing codebase

```bash
cd existing-project
docker compose up -d
docker compose exec codex codex "Refactor the user authentication module and add rate limiting"
```

### Example 3: Run Codex non-interactively

```bash
docker compose exec codex codex "Add TypeScript types and improve error handling" --yes
```

## VS Code Dev Containers Support

This repository is designed to work seamlessly with Dev Containers. Add a `.devcontainer/devcontainer.json` to your project:

```json
{
  "image": "naoyoshinori/codex:0-typescript-node",
  "workspaceFolder": "/workspace",
  "postCreateCommand": "codex --version",
  "remoteUser": "node"
}
```

## Configuration

- Codex configuration is persisted in `~/.codex_cli` on the host
- Git config is shared from the host
- All project files are mounted at `/workspace`

## Building Locally

```bash
git clone https://github.com/naoyoshinori/docker-codex.git
cd docker-codex

# Build specific variant
docker build -t codex:local -f src/node/Dockerfile .
```

## Related Projects

- [Official Codex CLI Documentation](https://developers.openai.com/codex/cli)
- [docker-gemini-cli](https://github.com/naoyoshinori/docker-gemini-cli) (Google Gemini CLI equivalent)

## License

MIT License — see [LICENSE](LICENSE) file.
