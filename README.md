---
title: README
description:
published: true
date: 2026-02-12T12:40:59.877Z
tags:
editor: markdown
dateCreated: 2026-02-11T14:56:22.797Z
---

# Ascension - Documentation

This project contains all the general documentation and resources of the project.

## Quick Start 🚀

Get the project running locally in under 5 minutes.

### Prerequisites

- **Docker** & **Docker Compose**
- **Git**
- [**Just**](https://github.com/casey/just) (Task runner)
  - macOS: `brew install just`
  - Linux: `curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /usr/local/bin` / `apt install just` / `pacman -S just`

### Installation

1. **Clone the repository** (with submodules):
   ```bash
   git clone --recursive https://github.com/Ascension-EIP/Ascension.git
   cd Ascension
   ```

2. **Initialize the environment**:
   ```bash
   just setup
   ```
   *This initializes submodules and creates your `.env` file.*

3. **Start the services**:
   ```bash
   just start
   ```
   *Starts Postgres, Redis, MinIO, API, and AI Worker.*

4. **Verify**:
   - API: [http://localhost:8080/health](http://localhost:8080/health)
   - MinIO Console: [http://localhost:9001](http://localhost:9001) (User/Pass: `minioadmin`)

### Common Commands

| Command | Description |
|---------|-------------|
| `just setup` | Initialize submodules & environment |
| `just start` | Start all services (detached) |
| `just logs` | View live logs |
| `just stop` | Stop all services |
| `just update-repos` | Pull latest code for all submodules |

For detailed setup instructions, see the [Development Guide](developer_guide/architecture/deployment/development.md).
