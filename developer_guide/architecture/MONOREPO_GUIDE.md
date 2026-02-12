# Ascension Monorepo Architecture Guide

## Overview

This guide explains how Ascension uses a **monorepo with Git submodules** architecture to organize its codebase across multiple services.

## Repository Structure

```
Ascension/ (Main Repository)
│
├── .gitmodules              # Submodule configuration
├── docker-compose.yml       # Development orchestration
├── .env.example            # Environment template
├── README.md
├── justfile                # Common development commands
│
├── server/                 # Git Submodule
│   └── git@github.com:Ascension-EIP/server.git
│       ├── Cargo.toml
│       ├── Dockerfile
│       ├── src/
│       └── migrations/
│
├── ai/                     # Git Submodule
│   └── git@github.com:Ascension-EIP/ai.git
│       ├── requirements.txt
│       ├── Dockerfile
│       ├── src/
│       └── models/
│
├── mobile/                 # Git Submodule
│   └── git@github.com:Ascension-EIP/mobile.git
│       ├── README.md
│       └── app/                # Application (en développement)
│
├── docs/                   # Git Submodule
│   └── git@github.com:Ascension-EIP/docs.git
│       └── developer_guide/
│           └── architecture/  (you are here!)
│
└── benchmark/              # Git Submodule
    └── git@github.com:Ascension-EIP/benchmark.git
        ├── flutter_benchmark/
        └── react-benchmark/
```

## Why Submodules?

### Advantages

1. **Independent Versioning**
   - Each service has its own version control
   - Can tag releases independently (e.g., `server@v1.2.0`, `ai@v2.0.1`)

2. **Team Autonomy**
   - Teams can work on their service without affecting others
   - Separate pull requests and code reviews per service

3. **Flexible CI/CD**
   - Each submodule can have its own CI/CD pipeline
   - Only build/deploy the services that changed

4. **Unified Development**
   - Main repo provides docker-compose for local development
   - Single command to start all services

### Trade-offs

- **Complexity**: Requires understanding of Git submodules
- **Synchronization**: Must manually update submodules in main repo
- **Learning Curve**: Team members need to know submodule workflows

## Working with Submodules

### Initial Clone

```bash
# Clone main repo with all submodules
git clone --recursive https://github.com/Ascension-EIP/Ascension.git
cd Ascension

# If you forgot --recursive
git submodule update --init --recursive
```

### Daily Workflow

#### Working on a Specific Service

```bash
# Navigate to the submodule
cd server

# Create a branch (in the server repo)
git checkout -b feature/new-endpoint

# Make changes, commit
git add .
git commit -m "feat: add new endpoint"

# Push to server repo
git push origin feature/new-endpoint

# Create PR in server repo
```

#### Updating Main Repo with Submodule Changes

```bash
# After your PR is merged in server repo
cd Ascension/server
git checkout main
git pull

# Go back to main repo
cd ..

# Commit the submodule update
git add server
git commit -m "chore: update server submodule to v1.3.0"
git push
```

#### Updating All Submodules

```bash
# From main repo root
git submodule update --remote --merge

# Review changes
git status

# Commit if needed
git add .
git commit -m "chore: update all submodules"
```

### Common Commands

```bash
# Check submodule status
git submodule status

# Update specific submodule
cd server && git pull && cd ..

# Update all submodules to their latest commit
git submodule update --remote

# Reset submodule to main repo's recorded commit
git submodule update

# Add new submodule (rarely needed)
git submodule add git@github.com:Ascension-EIP/NEW_REPO.git path/to/new_repo
```

## Development Environment

### Docker Compose Setup

The main repository's `docker-compose.yml` references all submodules:

```yaml
# Located at: Ascension/docker-compose.yml

services:
  api:
    build:
      context: ./server      # References server submodule
      dockerfile: Dockerfile
    # ...

  worker:
    build:
      context: ./ai          # References ai submodule
      dockerfile: Dockerfile
    # ...
```

### Starting Development Environment

```bash
# From Ascension root
docker-compose up -d

# Verify all services are running
docker-compose ps
```

## CI/CD Architecture

### Per-Service Pipelines

Each submodule has its own CI:

**server/.github/workflows/ci.yml**:
```yaml
name: Server CI
on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: cargo test
      - run: cargo clippy
```

**ai/.github/workflows/ci.yml**:
```yaml
name: AI Worker CI
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: pytest
```

### Main Repo Orchestration

**Ascension/.github/workflows/deploy.yml**:
```yaml
name: Deploy All Services
on:
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive  # Fetch all submodules

      - name: Build API
        run: |
          cd server
          docker build -t ascension-api .

      - name: Build Worker
        run: |
          cd ai
          docker build -t ascension-worker .

      # Deploy both...
```

## Best Practices

### 1. Always Use `--recursive` When Cloning

```bash
git clone --recursive https://github.com/Ascension-EIP/Ascension.git
```

### 2. Update Submodules Before Starting Work

```bash
git pull
git submodule update --init --recursive
```

### 3. Commit Submodule Updates Separately

```bash
# Good
git add server
git commit -m "chore: update server to v2.1.0"

# Avoid mixing with code changes
```

### 4. Document Breaking Changes

If a submodule update requires changes in main repo (e.g., new env vars):

```bash
git commit -m "chore: update ai submodule to v3.0.0

BREAKING CHANGE: Requires new MODEL_VERSION env variable"
```

### 5. Pin Submodules in Production

For production deployments, reference specific commits:

```bash
cd server
git checkout v1.2.0
cd ..
git add server
git commit -m "chore: pin server to v1.2.0 for production"
```

## Troubleshooting

### Problem: Submodule Shows Changes After Pull

```bash
# You pulled main repo, submodules are outdated
git submodule update
```

### Problem: Submodule in Detached HEAD State

```bash
cd server
git checkout main
git pull
cd ..
```

### Problem: Merge Conflict in Submodule Reference

```bash
# Accept one version
git add server
git commit

# Or manually resolve
cd server
git checkout <desired-commit>
cd ..
git add server
git commit
```

### Problem: Submodule Not Initialized

```bash
git submodule update --init --recursive
```

## Migration Notes

If you're converting from a monolithic repo to submodules:

1. Extract each service to its own repo
2. Add as submodules to main repo
3. Update all paths in docker-compose.yml
4. Update CI/CD pipelines
5. Update documentation

## Additional Resources

- [Git Submodules Documentation](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [Deployment Guide](./deployment/README.md)
- [Architecture Overview](./README.md)

---

**Last Updated**: 2026-02-12
**Maintainer**: Ascension DevOps Team
