# ADR-007: Package Management with uv

**Status**: Accepted

**Date**: 2025-11-18

## Context

Need a Python package and environment management solution that:
- Fast dependency resolution and installation
- Modern Python packaging standards (PEP 621)
- Virtual environment management
- Lock file support for reproducible builds
- Compatible with standard Python tooling

Traditional options include pip, pip-tools, poetry, and pdm. However, a newer tool, `uv`, has emerged offering significant performance improvements.

## Decision

Use **uv** for Python package and environment management with:
- `pyproject.toml` for dependency declaration (PEP 621 standard)
- `uv.lock` for deterministic dependency resolution
- uv virtual environment management
- Standard pip-compatible installation flow

### Tool: uv
- **Created by**: Astral (makers of Ruff)
- **Written in**: Rust
- **Speed**: 10-100x faster than pip
- **Standards**: Full PEP 621 support

## Consequences

### Positive
- **Extreme Performance**: Dependency resolution and installation 10-100x faster than pip
- **Modern Standards**: Native `pyproject.toml` support (PEP 621)
- **Deterministic Builds**: Lock file ensures reproducible installations
- **Drop-in Replacement**: Compatible with pip commands (`uv pip install`)
- **Virtual Environment Management**: Built-in venv creation and activation
- **Active Development**: Backed by Astral, same team behind Ruff
- **Caching**: Aggressive caching reduces repeated downloads
- **Single Tool**: Replaces pip + pip-tools + virtualenv

### Negative
- **Newer Tool**: Less mature than pip/poetry (but actively maintained)
- **Learning Curve**: Different commands than traditional pip-tools workflow
- **Community Size**: Smaller community compared to pip/poetry
- **CI/CD Setup**: Requires uv installation in build pipelines

## Implementation Details

### Installation
```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# Alternative: pipx
pipx install uv
```

### Project Setup
```bash
# Create virtual environment
uv venv

# Activate (Linux/macOS)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Install dependencies
uv pip install -e ".[dev]"

# Or use uv directly (auto-activates venv)
uv run python -m src.main
```

### Dependency Management

**pyproject.toml** (PEP 621 standard):
```toml
[project]
name = "down-detector-down-detector"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.109.0",
    "uvicorn[standard]>=0.27.0",
    "httpx>=0.26.0",
    "duckdb>=0.10.0",
    "apscheduler>=3.10.0",
    "pydantic>=2.5.0",
    "pydantic-settings>=2.1.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-asyncio>=0.23.0",
    "pytest-cov>=4.1.0",
    "mypy>=1.8.0",
    "ruff>=0.1.0",
    "httpx[test]",
]
```

### Lock File
```bash
# Generate lock file
uv pip compile pyproject.toml -o uv.lock

# Install from lock file
uv pip sync uv.lock
```

### Common Commands
```bash
# Install package
uv pip install fastapi

# Install with extras
uv pip install ".[dev]"

# Run commands in venv
uv run pytest

# Update dependencies
uv pip install --upgrade-package fastapi

# Show installed packages
uv pip list
```

## Alternatives Considered

### pip + pip-tools
- **Pros**: Standard, ubiquitous, well-understood
- **Cons**: Slow dependency resolution, separate tools for different tasks
- **Rejected**: Performance matters for developer experience

### Poetry
- **Pros**: Mature, comprehensive, good dependency resolution
- **Cons**: Slower than uv, non-standard lock file format, complexity
- **Rejected**: uv provides similar benefits with better performance

### pdm
- **Pros**: PEP 621 compliant, modern, decent performance
- **Cons**: Less adoption, slower than uv, additional complexity
- **Rejected**: uv offers better performance with similar features

### Pipenv
- **Pros**: Official PyPA project, integrated Pipfile
- **Cons**: Slow, known reliability issues, declining popularity
- **Rejected**: Community moving away, performance concerns

## Migration Path

### From pip + requirements.txt
```bash
# Convert requirements.txt to pyproject.toml dependencies
# (manual conversion needed)

# Then:
uv venv
uv pip install -e ".[dev]"
```

### From poetry
```bash
# pyproject.toml structure compatible
# Convert [tool.poetry.dependencies] to [project.dependencies]

uv venv
uv pip install -e ".[dev]"
```

## CI/CD Integration

### GitHub Actions
```yaml
- name: Set up uv
  uses: astral-sh/setup-uv@v1

- name: Install dependencies
  run: |
    uv venv
    uv pip install -e ".[dev]"

- name: Run tests
  run: uv run pytest
```

### Local Development
```bash
# .envrc (for direnv)
layout python-uv
```

## Project Structure Impact

### Files to Create
- `pyproject.toml` - Project metadata and dependencies (PEP 621)
- `uv.lock` - Lock file for reproducible builds
- `.python-version` - Python version specification

### Files to Remove
- `requirements.txt` - Replaced by pyproject.toml
- `requirements-dev.txt` - Replaced by pyproject.toml extras
- `setup.py` - Not needed with modern pyproject.toml

### Files to Update
- `README.md` - Update setup instructions
- `.gitignore` - Add `.venv/` (if not already present)
- CI/CD configs - Use uv installation steps

## Performance Expectations

Based on benchmarks:
- **Dependency Resolution**: 10-100x faster than pip
- **Installation**: 5-50x faster than pip (with cache)
- **Cold Start**: 2-10x faster than poetry
- **Lock File Generation**: Nearly instantaneous

## Future Considerations

### Potential Issues
- If uv development stalls, migration path back to pip is straightforward (pyproject.toml is standard)
- Lock file format may evolve (currently pip-compatible)

### Monitoring
- Track uv releases and community adoption
- Evaluate if performance gains continue with project scale
- Consider poetry if collaboration requires it

## References

- [uv Documentation](https://github.com/astral-sh/uv)
- [PEP 621 - Project Metadata](https://peps.python.org/pep-0621/)
- [Astral Blog - uv Announcement](https://astral.sh/blog/uv)
- [uv vs pip Benchmarks](https://github.com/astral-sh/uv#benchmarks)
