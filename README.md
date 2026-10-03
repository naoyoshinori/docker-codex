# Docker for Codex CLI

[![MIT License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A ready-to-use and convenient Docker environment for **[OpenAI Codex CLI](https://developers.openai.com/codex/cli)** (`@openai/codex`).

This project provides Docker images that allow you to use the Codex CLI without a local Node.js installation. Designed with security and convenience in mind, these images run as a non-root user and offer variants for different use cases, from minimal execution to a full development environment integrated with VS Code Dev Containers.

Images are published on [Docker Hub](https://hub.docker.com/r/naoyoshinori/codex) and rebuilt automatically when a new Codex CLI version or a new base image is released.

> This is an unofficial community project and is not affiliated with OpenAI.

## Quick Start

1. **Prerequisites**:

    - [Docker](https://www.docker.com/get-started)
    - An [OpenAI API key](https://platform.openai.com/api-keys)

2. **Initial Setup**:

    In your **home directory**, create a `.env.codex` file for your API key and a `.codex_cli` directory to persist Codex settings and history.

```bash
    echo "OPENAI_API_KEY=YOUR_API_KEY_HERE" > ~/.env.codex
    chmod 600 ~/.env.codex
    mkdir -p ~/.codex_cli
```

3. **Run Codex CLI**:

    Execute the following command in your project directory. It mounts the current directory and persists your settings.

```bash
    docker run -it --rm \
      --env-file ~/.env.codex \
      -v "$HOME/.codex_cli:/home/node/.codex" \
      -v "$(pwd):/workspace" \
      -w /workspace \
      naoyoshinori/codex:0-node \
      codex
```

    To pass an initial prompt, append it after `codex`:

```bash
    ... naoyoshinori/codex:0-node codex "Explain the structure of this repository"
```

## Non-interactive Usage

For scripts and CI/CD, use `codex exec` to run a single prompt and exit:

```bash
docker run --rm \
  --env-file ~/.env.codex \
  -v "$HOME/.codex_cli:/home/node/.codex" \
  -v "$(pwd):/workspace" \
  -w /workspace \
  naoyoshinori/codex:0-node \
  codex exec "Add type hints to the functions in src/utils.py"
```

See the [official documentation](https://developers.openai.com/codex/cli) for the available options.

## Docker Compose

Keep a long-running container and open shells in it:

```yaml
services:
  codex:
    image: naoyoshinori/codex:0-node
    working_dir: /workspace
    environment:
      - OPENAI_API_KEY
    volumes:
      - ~/.gitconfig:/home/node/.gitconfig
      - ~/.codex_cli:/home/node/.codex
      - .:/workspace
    command: ["sleep", "infinity"]
```

`OPENAI_API_KEY` is passed through from your host shell. Then:

```bash
docker compose up -d
docker compose exec codex codex
```

> `~/.gitconfig` must exist on the host. Otherwise Docker creates a directory with that name.

## VS Code Dev Containers

Add a `.devcontainer/devcontainer.json` to your project:

```json
{
  "image": "naoyoshinori/codex:0-typescript-node",
  "postCreateCommand": "codex --version",
  "remoteUser": "node"
}
```

Use `0-javascript-node` instead for plain JavaScript projects.

## Image Variants

This project offers three image variants, each designed for a different use case.

| Variant | Base image | Best for |
|---|---|---|
| `node` | [Node.js official image](https://hub.docker.com/_/node) | Running Codex directly, scripts and CI/CD. Minimal. |
| `javascript-node` | [Dev Containers `javascript-node`](https://github.com/devcontainers/images/tree/main/src/javascript-node) | VS Code Dev Containers, work that needs `git`, `zsh` and other shell tools. |
| `typescript-node` | [Dev Containers `typescript-node`](https://github.com/devcontainers/images/tree/main/src/typescript-node) | TypeScript projects: adds the TypeScript compiler and related tools. |

All variants run as the non-root `node` user. The Dev Containers based variants (`javascript-node`, `typescript-node`) inherit the base image's setup, in which the `node` user can use `sudo`.

### Tags

| Tag | Points to |
|---|---|
| `0-node`, `0-javascript-node`, `0-typescript-node` | Latest Codex CLI 0.x on the current Node.js LTS (currently Node.js 24) |
| `0.<minor>-<variant>` (e.g. `0.128-node`) | Latest patch release of that minor version |
| `0-node-26-bookworm`, `0-node-22-bookworm-slim`, ... | A specific Node.js version of the base image |
| `patch-<version>-<variant>-<node>-<os>` | A specific Codex CLI release, e.g. `patch-0.128.0-node-24-bookworm-slim` |

Available base image tags:

* `node`: `26-bookworm`, `26-bookworm-slim`, `24-bookworm`, `24-bookworm-slim`, `22-bookworm`, `22-bookworm-slim`
* `javascript-node`: `26-bookworm`, `24-bookworm`, `22-bookworm`
* `typescript-node`: `26-bookworm`, `24-bookworm`, `22-bookworm`

The short tags (`0-node`, `0-javascript-node`, `0-typescript-node`) point to `24-bookworm-slim`, `24-bookworm` and `24-bookworm` respectively. A `latest` tag is not provided, to encourage deliberate version selection. For maximum reproducibility (e.g., in CI/CD), pin a `patch-...` tag.

Supported architectures: `linux/amd64`, `linux/arm64`.

## Configuration

- Codex settings and history are persisted in `~/.codex_cli` on the host (mounted at `/home/node/.codex`).
- Your Git configuration can be shared from the host with `~/.gitconfig:/home/node/.gitconfig`.
- Project files are mounted at `/workspace`.

## Building Locally

```bash
git clone https://github.com/naoyoshinori/docker-codex.git
cd docker-codex

docker build \
  -f src/node/Dockerfile \
  --build-arg VARIANT=24-bookworm-slim \
  -t codex:local \
  src/node
```

## Related Projects

- [Official Codex CLI documentation](https://developers.openai.com/codex/cli)
- [docker-gemini-cli](https://github.com/naoyoshinori/docker-gemini-cli): the same kind of images for Google's Gemini CLI

## License

- The OpenAI Codex CLI is licensed under the [Apache 2.0 License](https://github.com/openai/codex/blob/main/LICENSE).
- The Dockerfile and associated scripts for this project are licensed under the [MIT License](LICENSE).
- As with all Docker images, these images contain other software under other licenses (e.g., the base OS distribution and its dependencies).
