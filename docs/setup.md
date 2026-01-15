# reShark Setup Guide with uv

This guide covers setting up the reShark development environment using [uv](https://github.com/astral-sh/uv), a fast Python package installer and resolver.

## Prerequisites

- **Python 3.11+** (uv will manage this for you)
- **Docker 24+** (for containerized execution)
- **uv** (install instructions below)

## Quick Start

### 1. Install uv

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or with Homebrew
brew install uv

# Or with pip
pip install uv
```

### 2. Clone and Setup Repository

```bash
# Clone repository
git clone https://github.com/your-org/reShark.git
cd reShark

# Create virtual environment and install dependencies
uv sync

# This installs:
# - Core dependencies (numpy, scipy, pandas, scapy, pyyaml, pydantic)
# - Development dependencies (pytest, black, mypy, ruff, etc.)
```

### 3. Install Tools (OpenHands, etc.)

```bash
# Install tool dependencies (optional)
uv sync --extra tools

# This installs:
# - openhands
# - ipython
# - jupyterlab
```

### 4. Install Phase 2 Dependencies (Optional)

```bash
# For Phase 2 development (LangGraph + Patternbook)
uv sync --extra phase2

# Or install everything
uv sync --all-extras
```

### 5. Verify Installation

```bash
# Activate the virtual environment
source .venv/bin/activate

# Or use uv run to run commands directly
uv run pytest --version
uv run python -c "import numpy, scipy, pandas, scapy; print('All imports successful')"

# Check tshark (should be installed via Docker)
docker run --rm reshark:latest tshark --version
```

## Development Workflow

### Running Tests

```bash
# Run all tests with coverage
uv run pytest

# Run specific test file
uv run pytest tests/test_observer.py

# Run with specific markers
uv run pytest -m unit           # Only unit tests
uv run pytest -m integration    # Only integration tests
uv run pytest -m "not slow"     # Skip slow tests

# Run in parallel (faster)
uv run pytest -n auto

# Watch mode (requires pytest-watch)
uv run ptw
```

### Code Quality

```bash
# Format code with black
uv run black reshark/ tests/

# Sort imports with isort
uv run isort reshark/ tests/

# Type check with mypy
uv run mypy reshark/

# Lint with ruff
uv run ruff check reshark/

# Auto-fix with ruff
uv run ruff check --fix reshark/

# Run all quality checks
uv run black --check reshark/ tests/
uv run isort --check reshark/ tests/
uv run mypy reshark/
uv run ruff check reshark/
```

### Pre-commit Hooks

```bash
# Install pre-commit hooks
uv run pre-commit install

# Run manually on all files
uv run pre-commit run --all-files
```

### Constitutional Compliance Check

```bash
# Verify constitutional compliance
uv run python scripts/check_compliance.py
```

## Adding Dependencies

### Core Dependency

```bash
# Add a new runtime dependency
uv add requests

# This updates pyproject.toml and uv.lock
```

### Development Dependency

```bash
# Add a dev dependency
uv add --dev pytest-mock

# Or manually add to pyproject.toml [project.optional-dependencies.dev]
# Then run:
uv sync
```

### Tool Dependency

```bash
# Install a tool globally (not in project venv)
uv tool install openhands --python 3.12

# Or add to project
uv add --optional tools openhands
```

## Docker Integration

### Build Docker Image with uv

See [docker/Dockerfile.uv](docker/Dockerfile.uv) for the uv-optimized Dockerfile.

```bash
# Build image
docker build -f docker/Dockerfile.uv -t reshark:latest .

# Run container
docker compose -f docker/docker-compose.yml up -d

# Enter container
docker compose exec reshark bash
```

## Common Commands Reference

| Task | uv Command |
|------|-----------|
| Install all dependencies | `uv sync` |
| Install with extras | `uv sync --extra phase2` |
| Install all extras | `uv sync --all-extras` |
| Add dependency | `uv add <package>` |
| Add dev dependency | `uv add --dev <package>` |
| Remove dependency | `uv remove <package>` |
| Update all packages | `uv lock --upgrade` |
| Run command in venv | `uv run <command>` |
| Run Python script | `uv run python script.py` |
| Show dependency tree | `uv tree` |
| Export requirements.txt | `uv export --format requirements-txt > requirements.txt` |

## Phase 1 Workflow

```bash
# 1. Setup environment
uv sync --extra dev

# 2. Activate virtual environment
source .venv/bin/activate

# 3. Run analysis (once implemented)
uv run reshark-analyze \
  --golden pcaps/golden/test.pcap \
  --validation pcaps/validation/*.pcap

# 4. Run tests
uv run pytest -n auto

# 5. Check compliance
uv run python scripts/check_compliance.py
```

## Phase 2 Workflow

```bash
# 1. Install Phase 2 dependencies
uv sync --all-extras

# 2. Start Qdrant (for Patternbook)
docker compose up -d qdrant

# 3. Run autonomous analysis (once implemented)
uv run python -m reshark.orchestration.langgraph_analyze \
  --sessions configs/multi_protocol.yaml

# 4. Monitor LangGraph execution
uv run jupyter lab  # Open notebooks/monitor.ipynb
```

## CI/CD Integration

See [.github/workflows/tests.yml](.github/workflows/tests.yml) for GitHub Actions configuration using uv.

Key points:
- uv is much faster than pip/poetry (~10-100x faster)
- Lock file ensures reproducible builds
- Cache uv downloads for even faster CI

## Troubleshooting

### uv command not found

```bash
# Reinstall uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or add to PATH manually
export PATH="$HOME/.cargo/bin:$PATH"
```

### Virtual environment not activating

```bash
# Recreate virtual environment
rm -rf .venv
uv sync
```

### Dependency conflicts

```bash
# Regenerate lock file
uv lock --upgrade

# Or force reinstall
rm uv.lock
uv sync
```

### Docker build fails

```bash
# Ensure uv.lock is committed
git add uv.lock pyproject.toml
git commit -m "chore: update dependencies"

# Rebuild without cache
docker build --no-cache -f docker/Dockerfile.uv -t reshark:latest .
```

## Migration from Poetry

If you're migrating from Poetry:

```bash
# 1. Remove Poetry files
rm poetry.lock pyproject.toml  # (backup first!)

# 2. Use the provided pyproject.toml in this repo

# 3. Generate lock file
uv lock

# 4. Install dependencies
uv sync
```

## Performance Comparison

| Operation | Poetry | uv | Speedup |
|-----------|--------|-----|---------|
| Install (cold) | ~45s | ~2s | 22x |
| Install (cached) | ~15s | ~0.3s | 50x |
| Lock file generation | ~30s | ~1s | 30x |
| Add package | ~10s | ~0.5s | 20x |

## Additional Resources

- [uv Documentation](https://github.com/astral-sh/uv)
- [pyproject.toml Reference](https://packaging.python.org/en/latest/specifications/pyproject-toml/)
- [reShark Constitution](docs/constitution-v2.md)
- [Implementation Plan](specs/implementation-plan.md)

## Support

For issues with:
- **uv setup**: See [uv GitHub Issues](https://github.com/astral-sh/uv/issues)
- **reShark development**: See [reShark Issues](https://github.com/your-org/reShark/issues)
- **Constitutional questions**: Review [Constitution v2.0.0](docs/constitution-v2.md)
