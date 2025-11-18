# uv Quick Reference Guide

This project uses **uv** for Python package and environment management. This guide provides quick reference commands for common development tasks.

## Why uv?

- **10-100x faster** than pip for dependency resolution and installation
- **Modern standards**: Native PEP 621 support via `pyproject.toml`
- **Deterministic builds**: Lock file ensures reproducible installations
- **Single tool**: Replaces pip + pip-tools + virtualenv

See [ADR-007](adr/ADR-007-package-management.md) for the complete rationale.

## Installation

### macOS / Linux
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Windows (PowerShell)
```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Alternative: pipx
```bash
pipx install uv
```

## Common Commands

### Initial Setup

```bash
# Create virtual environment
uv venv

# Activate virtual environment
# macOS/Linux:
source .venv/bin/activate
# Windows (PowerShell):
.\.venv\Scripts\Activate.ps1
# Windows (Git Bash):
source .venv/Scripts/activate

# Install project with dev dependencies
uv pip install -e ".[dev]"
```

### Running Commands

```bash
# Run Python directly (auto-activates venv)
uv run python -m src.main --dev

# Run pytest
uv run pytest

# Run specific test file
uv run pytest tests/test_monitor.py -v

# Type checking with mypy
uv run mypy src/

# Linting with ruff
uv run ruff check src/

# Auto-fix linting issues
uv run ruff check --fix src/
```

### Dependency Management

```bash
# Add a new dependency (edit pyproject.toml manually)
# Then sync:
uv pip install -e "."

# Generate lock file
uv pip compile pyproject.toml -o uv.lock

# Install from lock file (for reproducible builds)
uv pip sync uv.lock

# Update a specific package
uv pip install --upgrade-package fastapi

# Show installed packages
uv pip list

# Show dependency tree
uv pip show <package-name>
```

### Development Workflow

```bash
# 1. Clone and setup
git clone <repo-url>
cd down-detector-down-detector
uv venv
uv pip install -e ".[dev]"

# 2. Make changes to code
# ... edit files ...

# 3. Run tests
uv run pytest

# 4. Type checking
uv run mypy src/

# 5. Linting
uv run ruff check --fix src/

# 6. Run application
uv run python -m src.main --dev
```

### Production Deployment

```bash
# Create production virtual environment
uv venv

# Install production dependencies only (no dev extras)
uv pip install -e .

# Run scheduler
uv run python -m src.scheduler

# Run API (separate terminal)
uv run python -m src.api
```

## Project Structure

```
down-detector-down-detector/
├── pyproject.toml       # Dependencies and project metadata (PEP 621)
├── uv.lock              # Lock file (commit this to git!)
├── .python-version      # Python version specification
└── .venv/               # Virtual environment (gitignored)
```

## pyproject.toml Structure

```toml
[project]
name = "down-detector-down-detector"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.109.0",
    # ... production dependencies
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    # ... development dependencies
]

[project.scripts]
dddd = "src.main:main"
dddd-scheduler = "src.scheduler:main"
dddd-api = "src.api:main"
```

## Adding New Dependencies

### Production Dependency
1. Edit `pyproject.toml`
2. Add to `dependencies` list
3. Run `uv pip install -e "."`

### Development Dependency
1. Edit `pyproject.toml`
2. Add to `[project.optional-dependencies] dev` list
3. Run `uv pip install -e ".[dev]"`

### Example
```toml
dependencies = [
    "fastapi>=0.109.0",
    "new-package>=1.0.0",  # Add this
]
```

Then:
```bash
uv pip install -e "."
```

## CI/CD Integration

### GitHub Actions
```yaml
- name: Set up uv
  uses: astral-sh/setup-uv@v1

- name: Create venv and install dependencies
  run: |
    uv venv
    uv pip install -e ".[dev]"

- name: Run tests
  run: uv run pytest

- name: Type checking
  run: uv run mypy src/
```

## Troubleshooting

### Virtual environment not activating
```bash
# Make sure you created it first
uv venv

# Try manual activation
source .venv/bin/activate  # macOS/Linux
.\.venv\Scripts\Activate.ps1  # Windows PowerShell
```

### Package not found after installation
```bash
# Make sure you're in the right directory
pwd

# Reinstall
uv pip install -e ".[dev]"

# Check installation
uv pip list
```

### Dependency conflicts
```bash
# Try regenerating lock file
uv pip compile pyproject.toml -o uv.lock

# Install from fresh lock
uv pip sync uv.lock
```

## Performance Tips

1. **Use lock files**: Pre-resolved dependencies install much faster
2. **Leverage cache**: uv caches packages globally for reuse
3. **Run without activation**: Use `uv run` instead of activating venv
4. **Parallel operations**: uv automatically parallelizes when possible

## Migration from pip

### Convert requirements.txt to pyproject.toml
```bash
# Manual conversion needed
# Copy packages from requirements.txt to pyproject.toml [project.dependencies]

# Example:
# requirements.txt:
#   fastapi==0.109.0
#   uvicorn==0.27.0

# pyproject.toml:
#   dependencies = [
#       "fastapi>=0.109.0",
#       "uvicorn>=0.27.0",
#   ]
```

### Then setup with uv
```bash
uv venv
uv pip install -e ".[dev]"
```

## Resources

- [uv GitHub Repository](https://github.com/astral-sh/uv)
- [uv Documentation](https://github.com/astral-sh/uv/blob/main/README.md)
- [PEP 621 Specification](https://peps.python.org/pep-0621/)
- [Astral Blog](https://astral.sh/blog)
- [ADR-007: Package Management](adr/ADR-007-package-management.md)
