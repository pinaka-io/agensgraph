# AgensGraph Multiplatform Images

[![Build Status](https://github.com/pinaka-io/agensgraph/actions/workflows/build-multiplatform.yml/badge.svg)](https://github.com/pinaka-io/agensgraph/actions/workflows/build-multiplatform.yml)

Multiplatform Docker images for [AgensGraph](https://github.com/skaiworldwide-oss/agensgraph), a graph database extension for PostgreSQL.

## Why This Repository?

The upstream AgensGraph images at [skaiworldwide/agensgraph](https://hub.docker.com/r/skaiworldwide/agensgraph) **only support AMD64**. This repository provides:

✅ **Multi-architecture support** - linux/amd64, linux/arm64
✅ **Apple Silicon compatibility** - Native ARM64 builds
✅ **Automated builds** - CI/CD pipeline with version tracking
✅ **Developer tooling** - Taskfile for local development

## Quick Start

```bash
# Pull and run the latest version
docker pull ghcr.io/pinaka-io/agensgraph:v2.16.0
docker run -d \
  --name agensgraph \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -p 5432:5432 \
  ghcr.io/pinaka-io/agensgraph:v2.16.0

# Connect with psql
docker exec -it agensgraph psql -U postgres
```

## Available Versions

This repository tracks official upstream releases from [skaiworldwide-oss/agensgraph](https://github.com/skaiworldwide-oss/agensgraph/releases).

**Current latest**: v2.16.0

All released images: [ghcr.io/pinaka-io/agensgraph](https://github.com/pinaka-io/agensgraph/pkgs/container/agensgraph)

## Usage

### Pull the Image

```bash
# Specific version (recommended)
docker pull ghcr.io/pinaka-io/agensgraph:v2.16.0

# Or get the latest
docker pull ghcr.io/pinaka-io/agensgraph:latest
```

### Run AgensGraph

```bash
docker run -d \
  --name agensgraph \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -p 5432:5432 \
  ghcr.io/pinaka-io/agensgraph:v2.16.0
```

### Connect to AgensGraph

```bash
# Using psql
docker exec -it agensgraph psql -U postgres

# Check AgensGraph version
docker exec -it agensgraph psql -U postgres -c "SELECT version();"
```

## Local Development

### Using Task (Recommended)

This repository includes a comprehensive [Taskfile](https://taskfile.dev/) with 27 tasks for building, testing, and managing Docker images.

**Install Task:**

```bash
# macOS
brew install go-task

# Linux
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin

# Or with Go
go install github.com/go-task/task/v3/cmd/task@latest
```

**Common Tasks:**

```bash
# Build and test
task build                      # Build for current platform
task test                       # Run basic tests
task test:full                  # Full integration tests
task verify                     # Complete verification

# Run locally
task run                        # Start container on port 5432
task psql                       # Connect with psql
task logs                       # View container logs
task stop                       # Stop and remove container

# Multiplatform builds
task build:multiplatform        # Build for amd64 and arm64
task build:version -- v2.15.0   # Build specific version

# Version management
task tag:create -- v2.17.0      # Create and push new tag
task check:upstream             # Check upstream releases
task check:builds               # Check CI/CD status

# Help
task --list                     # List all 27 tasks
task help                       # Detailed help
```

### Using Docker Directly

**Single platform:**
```bash
docker build --build-arg AGENSGRAPH_VERSION=v2.16.0 -t agensgraph:latest .
```

**Multiplatform:**
```bash
docker buildx create --use
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --build-arg AGENSGRAPH_VERSION=v2.16.0 \
  -t agensgraph:latest \
  .
```

**Note**: Always use official upstream versions from https://github.com/skaiworldwide-oss/agensgraph/releases

## CI/CD Pipeline

Images are automatically built and published via GitHub Actions:

**Triggers:**
- Version tags (e.g., `v2.16.0`) → builds matching AgensGraph version
- GitHub releases → builds and publishes
- Manual workflow dispatch → on-demand builds

**Build Process:**
1. Parallel builds for linux/amd64 and linux/arm64 using QEMU
2. Compiles AgensGraph from source for each platform
3. Creates multi-arch manifest
4. Publishes to GitHub Container Registry (GHCR)

**Tags Created:**
- `vX.Y.Z` - Full semantic version only (e.g., `v2.16.0`)

Check build status: [GitHub Actions](https://github.com/pinaka-io/agensgraph/actions)

## Repository Structure

```
.
├── Dockerfile                     # Multiplatform build with version arg
├── Taskfile.yaml                  # Task runner (27 tasks)
├── .github/workflows/
│   └── build-multiplatform.yml   # CI/CD pipeline
├── CLAUDE.md                      # Detailed project documentation
└── README.md                      # This file
```

## Contributing

1. **Check upstream releases**: Always verify version exists at https://github.com/skaiworldwide-oss/agensgraph/releases
2. **Create a tag**: `task tag:create -- vX.Y.Z`
3. **Monitor build**: `task check:builds` or `task watch:build`
4. **Test locally**: `task build && task test:full`

## Documentation

- **[CLAUDE.md](./CLAUDE.md)** - Comprehensive project documentation, architecture, workflows, and troubleshooting
- **[Taskfile.yaml](./Taskfile.yaml)** - All available development tasks
- **[Upstream Releases](https://github.com/skaiworldwide-oss/agensgraph/releases)** - Official AgensGraph versions

## Links

- **Images**: [ghcr.io/pinaka-io/agensgraph](https://github.com/pinaka-io/agensgraph/pkgs/container/agensgraph)
- **Source**: [github.com/pinaka-io/agensgraph](https://github.com/pinaka-io/agensgraph)
- **Upstream**: [github.com/skaiworldwide-oss/agensgraph](https://github.com/skaiworldwide-oss/agensgraph)
- **Build Status**: [GitHub Actions](https://github.com/pinaka-io/agensgraph/actions)

## License

This repository follows the same license as the upstream AgensGraph project.
