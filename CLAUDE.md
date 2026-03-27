# AgensGraph Multiplatform Docker Images

**IMPORTANT**: This context file (CLAUDE.md) should be updated after every significant change or request to keep the documentation current and accurate. Always update this file when:
- Making changes to the repository structure
- Modifying workflows or build processes
- Adding new features or tasks
- Fixing bugs or issues
- Releasing new versions
- Learning new information about the project

## Project Overview

This repository builds and publishes multiplatform Docker images for AgensGraph, a graph database extension for PostgreSQL. The upstream project at https://hub.docker.com/r/skaiworldwide/agensgraph only provides AMD64 images, so this repository extends support to ARM64 and other platforms.

## Upstream Information

- **Source Repository**: https://github.com/skaiworldwide-oss/agensgraph
- **Official Releases**: https://github.com/skaiworldwide-oss/agensgraph/releases
- **Latest Version**: v2.16.0 (as of 2025-09-12)

**IMPORTANT**: Always use official upstream release versions. Check the releases page before creating new tags.

## Repository Structure

```
.
├── Dockerfile                          # Multiplatform Dockerfile with version argument
├── Taskfile.yaml                      # Task runner (27 tasks for dev workflow)
├── .github/workflows/
│   └── build-multiplatform.yml        # GitHub Actions CI/CD pipeline
├── CLAUDE.md                          # Context file for Claude (this file)
├── README.md                          # User-facing documentation
└── .claude/                           # Claude-specific configs (if any)
```

### File Descriptions

**Dockerfile**
- Accepts `AGENSGRAPH_VERSION` build argument (defaults to v2.16.0)
- Based on postgres:16-bookworm
- Installs build dependencies and compiles AgensGraph from source
- Supports multiplatform builds via Docker Buildx

**Taskfile.yaml**
- 27 tasks organized into categories (build, test, run, version management, cleanup)
- Uses Task v3 syntax with proper shell operator handling
- Primary interface for local development
- Includes upstream checking and CI/CD monitoring tasks

**.github/workflows/build-multiplatform.yml**
- Matrix strategy for parallel builds (amd64, arm64)
- Digest-based approach for true multiplatform images
- Version extraction from Git tags
- GHCR publishing with semantic versioning
- Requires `packages: write` permission

**CLAUDE.md (this file)**
- Project overview and architecture
- Development workflows and best practices
- Troubleshooting guide
- Historical context and decision rationale
- Updated after every significant change

**README.md**
- User-facing quick start guide
- Installation and usage instructions
- Link to detailed documentation (this file)
- Public-facing project description

## Current Status

### Released Versions
- **v2.16.0**: Initial release with multiplatform support (linux/amd64, linux/arm64)
  - Currently building after re-tagging with latest changes
  - Build run: https://github.com/nishantapatil3/agensgraph/actions/runs/23505661585

### Repository
- **Organization**: https://github.com/nishantapatil3
- **Repository**: https://github.com/nishantapatil3/agensgraph
- **Container Images**: https://github.com/nishantapatil3/agensgraph/pkgs/container/agensgraph

### Recent Changes (2026-03-24)
1. **Simplified Docker tags** (latest)
   - Only generate full semantic version tags (vX.Y.Z)
   - Removed major/minor aliases (vX.Y, vX) for clarity
   - Removed latest tag to ensure explicit versioning
   - Prevents confusion with multiple tag aliases

2. **Added Taskfile.yaml** (commit 5efef92)
   - Comprehensive task runner for local development
   - 27 tasks for building, testing, running, and managing containers
   - Includes version management and upstream checking tasks

3. **Fixed Taskfile syntax** (commit 8becbd5)
   - Resolved YAML parsing issues with shell operators
   - Fixed echo commands with colons
   - Properly escaped awk commands

4. **Added documentation for Task installation**
   - Installation instructions for macOS, Linux, and Go
   - Updated README and CLAUDE.md with Taskfile usage

5. **Major documentation update** (commit 9da31c9)
   - Modernized README with Quick Start and badges
   - Added Project Summary to CLAUDE.md
   - Documented known limitations and future improvements

### Workflow History
- Initial workflow had syntax errors (invalid Docker Hub references, artifact naming issues)
- Fixed in commit c9abd97 by removing Docker Hub and fixing artifact names
- Added package permissions in commit 16ccc07
- Changed triggers to tags/releases only in commit cddb307
- Made version dynamic based on Git tags in commit 5075f02
- Added comprehensive context documentation in commit 7fbb60c
- Retagged v2.16.0 to include all fixes

## Key Technologies

- **Base Image**: postgres:16-bookworm
- **AgensGraph Version**: Dynamic, based on Git tag (defaults to v2.16.0)
- **Build System**: Docker Buildx with QEMU for cross-platform compilation
- **CI/CD**: GitHub Actions
- **Container Registry**: GitHub Container Registry (GHCR)

## Build Process

The Dockerfile:
1. Accepts `AGENSGRAPH_VERSION` build argument (defaults to v2.16.0)
2. Starts from PostgreSQL 16 on Debian Bookworm
3. Installs build dependencies (build-essential, libreadline-dev, zlib1g-dev, flex, bison, git, libicu-dev, pkg-config)
4. Clones AgensGraph from GitHub at the specified version tag
5. Compiles AgensGraph from source using `./configure && make -j$(nproc) && make install`

The version is dynamically set during the build based on the Git tag that triggered the workflow.

## Supported Platforms

- linux/amd64 (x86_64)
- linux/arm64 (ARM 64-bit, including Apple Silicon)

## GitHub Actions Workflow

The CI/CD pipeline (`build-multiplatform.yml`):

### Architecture
- **Matrix Strategy**: Builds linux/amd64 and linux/arm64 in parallel
- **Digest-Based Approach**: Each platform builds separately and uploads digests
- **Manifest Merging**: Final job combines digests into a single multi-arch manifest
- **QEMU Emulation**: Enables cross-platform compilation on GitHub runners
- **GitHub Actions Cache**: Speeds up subsequent builds

### Triggers
- **Version tags (v\*)**: Automatically builds matching AgensGraph version
- **GitHub Releases**: Builds when a release is published
- **Manual dispatch**: Can be triggered manually via Actions UI

**Note**: Does NOT trigger on pushes to main to conserve CI resources.

### Permissions
```yaml
permissions:
  contents: read
  packages: write
```
Required for pushing images to GitHub Container Registry (GHCR).

### Workflow Steps
1. **Extract version from tag**: Determines AgensGraph version from Git tag
2. **Build jobs** (parallel for each platform):
   - Checkout code
   - Set up QEMU for emulation
   - Configure Docker Buildx
   - Login to GHCR
   - Build image with version-specific build argument
   - Push by digest (not by tag yet)
   - Upload digest as artifact
3. **Merge job**:
   - Download all digests
   - Create multi-arch manifest combining all platforms
   - Push final manifest with proper tags
   - Inspect and verify the image

## Container Registry Configuration

### GitHub Container Registry (GHCR)
- **Authentication**: Automatically configured via `GITHUB_TOKEN`
- **Image Location**: `ghcr.io/nishantapatil3/agensgraph`
- **Visibility**: Public (matches repository visibility)
- **Tags Generated**:
  - `vX.Y.Z` - Full semantic version only (e.g., `v2.16.0`)
  - Only exact version tags are created (no major/minor aliases)
  - No `latest` tag (ensures reproducible builds with explicit versions)

### Docker Hub
Docker Hub publishing has been removed from the workflow to simplify the setup. GHCR is sufficient for public distribution.

## Project Summary

### What This Repository Does
1. **Builds multiplatform Docker images** for AgensGraph (amd64 + arm64)
2. **Tracks upstream versions** from skaiworldwide-oss/agensgraph
3. **Automates publishing** to GitHub Container Registry via GitHub Actions
4. **Provides developer tooling** via Taskfile with 27 tasks
5. **Maintains documentation** in CLAUDE.md and README.md

### Key Features
- ✅ Apple Silicon (ARM64) native support
- ✅ Automated CI/CD pipeline with version extraction
- ✅ Developer-friendly Taskfile for local workflows
- ✅ Semantic versioning with Git tags
- ✅ Build caching for faster iterations
- ✅ Comprehensive documentation and troubleshooting

### Current State (2026-03-24)
- **Latest Version**: v2.16.0 (building)
- **Platforms**: linux/amd64, linux/arm64
- **Registry**: ghcr.io/pinaka-io/agensgraph
- **Build Method**: Compile from source per platform
- **Workflow Status**: Tag-triggered, no main branch builds
- **Development Tools**: Taskfile with 27 tasks

### Known Limitations
- Build time is 20-40 minutes due to source compilation
- ARM64 builds are slower due to QEMU emulation
- No Docker Hub publishing (removed for simplicity)
- Requires manual tag creation for new releases
- No automatic upstream version detection

### Future Improvements
- Consider caching compiled binaries
- Add automated upstream version checking
- Implement release notes generation
- Add health checks to Docker image
- Consider multi-stage builds for smaller images

## Development Guidelines

### Making Changes to the Dockerfile
- Always test builds locally first using `docker buildx build --platform linux/amd64,linux/arm64`
- Keep build dependencies minimal to reduce image size
- Pin AgensGraph version in git clone command

### Updating AgensGraph Version

**CRITICAL**: Always check the upstream releases first: https://github.com/skaiworldwide-oss/agensgraph/releases

To release a new AgensGraph version:
1. Verify the version exists in upstream releases
2. Create and push a git tag matching the exact AgensGraph version
   ```bash
   # Example: Building v2.15.0
   git tag -a v2.15.0 -m "Release AgensGraph v2.15.0"
   git push origin v2.15.0
   ```
3. The workflow will automatically build that specific AgensGraph version
4. The Dockerfile accepts `AGENSGRAPH_VERSION` as a build argument (defaults to v2.16.0)
5. Test locally with: `docker build --build-arg AGENSGRAPH_VERSION=v2.15.0 -t agensgraph:test .`

Available upstream versions include: v2.16.0 (latest), v2.15.0, v2.14.1, v2.13.1, v2.13.0, v2.12.1, v2.12.0, v2.5.0, etc.

### GitHub Actions Workflow Modifications
- The workflow uses digest-based approach for true multiplatform images
- Each platform builds separately and uploads digests
- The merge job combines digests into a single manifest list
- Maintain this pattern for optimal build parallelization

## Local Development with Taskfile

This repository includes a Taskfile.yaml for streamlined local development. [Task](https://taskfile.dev/) is a task runner similar to Make but uses YAML. The Taskfile provides 27 tasks organized into categories for building, testing, running, and managing the Docker images.

### Installing Task

```bash
# macOS
brew install go-task

# Linux
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin

# Or with Go
go install github.com/go-task/task/v3/cmd/task@latest
```

### Quick Reference

```bash
# Build and test
task build              # Build for current platform
task test               # Build and run tests
task test:full          # Full integration tests
task verify             # Complete verification

# Run locally
task run                # Start container on port 5432
task psql               # Connect with psql
task shell              # Open bash shell
task logs               # View logs
task stop               # Stop and remove container

# Multiplatform
task build:multiplatform   # Build for amd64 and arm64
task build:push            # Build and push to registry

# Version management
task build:version -- v2.15.0  # Build specific version
task tag:create -- v2.16.0     # Create and push tag
task tag:delete -- v2.16.0     # Delete tag
task check:upstream            # Check upstream releases
task check:builds              # Check GitHub Actions status
task watch:build               # Watch latest build

# Cleanup
task clean              # Remove built images
task clean:test         # Remove test containers
task clean:all          # Remove everything

# Help
task --list            # List all tasks
task help              # Detailed help
```

### Taskfile Best Practices

- **All shell operators (`||`, `&&`)** must be wrapped in `sh -c` or use multi-line blocks
- **Echo commands with colons** must be quoted to avoid YAML key-value parsing
- **Pipeline commands** should properly escape variables (especially in awk)
- Use `task --list` to see all available tasks
- Use `task help` for detailed command examples

## Common Tasks

### Releasing a New Version

1. **Check upstream releases first**:
   ```bash
   gh release list --repo skaiworldwide-oss/agensgraph --limit 10
   ```

2. **Create and push a tag matching the upstream version**:
   ```bash
   # Example for v2.15.0
   git tag -a v2.15.0 -m "Release AgensGraph v2.15.0"
   git push origin v2.15.0
   ```

3. **Monitor the build**:
   ```bash
   gh run watch --repo nishantapatil3/agensgraph
   ```

4. **Verify the image**:
   ```bash
   docker pull ghcr.io/nishantapatil3/agensgraph:v2.15.0
   docker run --rm ghcr.io/nishantapatil3/agensgraph:v2.15.0 agens --version
   ```

### Building Locally

**Using Taskfile** (recommended):
```bash
# Quick build for testing
task build

# Build specific version
task build:version -- v2.15.0

# Multiplatform build
task build:multiplatform

# Build and test
task test
```

**Using Docker directly**:

Single platform (faster for testing):
```bash
docker build --build-arg AGENSGRAPH_VERSION=v2.16.0 -t agensgraph:test .
```

Multiplatform (requires buildx):
```bash
docker buildx create --use --name multiplatform-builder
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --build-arg AGENSGRAPH_VERSION=v2.16.0 \
  -t agensgraph:test \
  --load .
```

### Testing the Image

**Using Taskfile**:
```bash
# Basic tests
task test

# Full integration tests
task test:full

# Verify image works
task verify

# Manual testing
task run                # Start container
task psql               # Connect with psql
task logs               # View logs
task stop               # Clean up
```

**Using Docker directly**:
```bash
# Start AgensGraph container
docker run -d --name agensgraph-test \
  -e POSTGRES_PASSWORD=testpass \
  -p 5432:5432 \
  ghcr.io/nishantapatil3/agensgraph:v2.16.0

# Check logs
docker logs agensgraph-test

# Connect with psql
docker exec -it agensgraph-test psql -U postgres

# Test graph functionality
docker exec -it agensgraph-test psql -U postgres -c "SELECT version();"

# Clean up
docker stop agensgraph-test && docker rm agensgraph-test
```

### Triggering Manual Build
```bash
# Via GitHub CLI
gh workflow run build-multiplatform.yml --repo nishantapatil3/agensgraph

# Or via web UI:
# https://github.com/nishantapatil3/agensgraph/actions/workflows/build-multiplatform.yml
# Click "Run workflow"
```

### Checking Build Status
```bash
# List recent runs
gh run list --repo nishantapatil3/agensgraph --limit 5

# Watch a specific run
gh run watch <run-id> --repo nishantapatil3/agensgraph

# View logs for failed run
gh run view <run-id> --repo nishantapatil3/agensgraph --log-failed
```

### Managing Tags

**Using Taskfile**:
```bash
# Create and push a tag
task tag:create -- v2.16.0

# Delete a tag (local and remote)
task tag:delete -- v2.16.0

# Check upstream releases
task check:upstream

# Check build status
task check:builds
task watch:build
```

**Using Git directly**:
```bash
# List all tags
git tag -l

# Delete a local tag
git tag -d v2.16.0

# Delete a remote tag (this will NOT stop an in-progress build)
git push origin --delete v2.16.0

# Retag (if you need to fix something)
git tag -d v2.16.0
git push origin --delete v2.16.0
git tag -a v2.16.0 -m "Release AgensGraph v2.16.0"
git push origin v2.16.0
```

## Important Notes

### Build Performance
- **ARM64 builds**: Take significantly longer (15-30 minutes) due to QEMU emulation
- **Compilation**: The `make -j$(nproc)` step is CPU-intensive
- **GitHub Runners**: Free tier has 2-core CPUs, which limits parallelization
- **Total build time**: Expect 20-40 minutes for both platforms combined

### Registry & Permissions
- **GHCR visibility**: Images are public by default since the repository is public
- **Package permissions**: The workflow has `packages: write` permission to push to GHCR
- **Tag lifecycle**: Tags are immutable once pushed; use new versions for updates

### Version Management
- **Git tags MUST match upstream**: Always verify version exists at https://github.com/skaiworldwide-oss/agensgraph/releases
- **No automatic updates**: This repository does not auto-detect new upstream versions
- **Manual tagging required**: Create and push tags manually to trigger builds

### Troubleshooting
- **Workflow not triggering**: Ensure you pushed a tag starting with `v` (e.g., `v2.16.0`)
- **Build failures**: Check if the upstream AgensGraph version exists and builds successfully
- **Permission denied**: Verify the repository has the `packages: write` permission set
- **Artifact name errors**: Platform names are converted to job indexes to avoid slash characters in artifact names
