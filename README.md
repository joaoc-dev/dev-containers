# Dev Containers

Dev Containers (Development Containers) are a way to use Docker containers as a full-featured development environment. Instead of installing tools and dependencies on your local machine, everything runs inside a container - ensuring consistent environments across different machines and team members.

## Template Configurations

This repository contains **Multiple DevContainer Configurations** that can be used as examples.

### Available Configurations

| Configuration | Base Image | Tools Included | Use Case |
|--------------|------------|----------------|----------|
| **react-dotnet-fullstack** | `dotnet:10.0-bookworm` | .NET 10 + Node 22 + pnpm + Docker + Tilt + `cz` | React frontend + .NET backend + K8s dev |
| **react-python-fullstack** | `python:3.12-bookworm` | Python 3.12 + Node 22 + pnpm + Poetry + Docker + Tilt + `cz` | React frontend + Python backend + K8s dev |
| **react-frontend** | `javascript-node:22-bookworm` | Node 22 + pnpm + `cz` | Frontend-only React/Vite projects |
| **nextjs** | `javascript-node:22-bookworm` | Node 22 + pnpm + `cz` | Next.js applications |
| **minimal** | `base:bookworm` | Shell + Docker + Node 22 + `cz` | Scripts, CLI tools, custom setups |

**Base Image Strategy:**
- **Language-Specific Images**: Each config uses an official image matching its primary stack
  - Node.js projects: `javascript-node:22-bookworm`
  - Python projects: `python:3.12-bookworm`
  - .NET projects: `dotnet:10.0-bookworm`
  - Minimal/CLI: `base:bookworm`
- **Features**: Additional tools via Dev Container Features (no custom Dockerfiles)
- **Version Pinning**: Major.minor versions for automatic security patches

## Commit tooling (Commitizen)

All templates install:
- **`commitizen`** (provides the `cz` / `git cz` command)
- **`cz-conventional-changelog`** (adapter that drives the interactive prompts)

The adapter is configured globally inside the container via `~/.czrc` so you can run `cz` in any git repo inside the container.

> Note: `cz-conventional-changelog` is a Commitizen adapter (prompt format). It does not itself generate a `CHANGELOG.md` file.

## Terminal (Zsh)

The templates include an **optional** Zsh terminal setup:
- Zsh + Oh My Zsh (via the official `common-utils` feature)
- `zsh-autosuggestions` (via the `zsh-plugins` feature)
- Oh My Zsh theme set to **`agnoster`**

To disable it, comment out the Zsh-related feature entries and the theme line in each template’s `devcontainer.json`.

### Docker Setup

These containers use **docker-outside-of-docker** - it mounts your host Docker socket instead of running a nested Docker daemon.

**Why?** For Tilt/Kubernetes orchestration testing, you need images built inside the container to be accessible to your host K8s cluster.

**Note:** Docker-in-Docker doesn't work reliably with Rancher Desktop due to buildx compatibility issues.

## Adding Services to Your Dev Container

**Why Docker Compose for databases/services?** I opted to keep databases/infra as separate containers, consistent across templates. Dev Container **Features** are still great for installing tooling *inside* the devcontainer, but Compose is the primary mechanism for multi-service stacks.

The following services can be opted-in via shared Docker Compose files. Just uncomment the services you need!

### Available Services

| Service | Default Credentials | Host Ports |
|---------|-------------------|------------|
| **SQL Server 2025** | `sa` / `YourStrong!Passw0rd` | 11433 |
| **PostgreSQL 18** | `postgres` / `postgres` | 15432 |
| **MongoDB 8** | `admin` / `password` | 37017 |
| **Redis 8** | Password: `redispassword` | 16379 |
| **RabbitMQ 4.2** | `admin` / `password` | 15672 (AMQP)<br>25672 (Management UI) |

**Note:** Host ports use a **+10000 offset** to avoid conflicts with locally installed services (e.g. Postgres \(5432 ? 15432\)).

### How to Add Services

Databases and other infra can be added as **opt-in Docker Compose services** by uncommenting the shared compose fragments in your chosen template?s `dockerComposeFile`.

See `.devcontainer/compose/README.md` for the full step-by-step instructions, ports, connection strings, persistence, and customization options.

## Getting Started

1. `Ctrl+Shift+P` **"Dev Containers: Reopen in Container"**
2. **Choose your configuration** when prompted (e.g., "react-dotnet-fullstack")
3. Wait for the build & startup
4. You're now inside the dev container
5. Delete the containers you don't need for that specific project.
6. Happy coding!

**IMPORTANT (avoids Compose collisions/orphans when copying templates):**
In a typical workflow, you will copy one of these templates into another repo. Make sure you rename the copied folder to something unique for that repo, e.g. `.devcontainer/scrumbanana/` instead of `.devcontainer/react-dotnet-fullstack/`. This helps avoid Docker Compose name/volume collisions across repos.

## Extensions

Each template?s `devcontainer.json` includes a small, opinionated set of recommended VS Code extensions under `customizations.vscode.extensions`, roughly the ?minimal? set I?d typically want for that environment.

## Customizing Your Configuration

Want to add more tools to a specific configuration? Edit that configuration's `devcontainer.json` and add features.

Browse available features: https://containers.dev/features

Then rebuild: `Ctrl+Shift+P` **"Dev Containers: Rebuild Container"**

## Important: Container Fails Or Dockerfile Update

You modified the Dockerfile, but Docker is reusing the old container instead of building a new one.

- Press `Ctrl+Shift+P` **"Dev Containers: Rebuild Container"**

**Alternative if that doesn't work:**

- `Ctrl+Shift+P` **"Dev Containers: Rebuild Container Without Cache"**

This forces Docker to:

- Stop the old container
- Remove it
- Build a fresh image from your Dockerfile
- Start a new container
  
## Cursor Container Build Issues

I did my best to get the configurations building in **Cursor**, but if you still encounter errors, try building them first in **VS Code** and then opening them in cursor.
