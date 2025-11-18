# Down Detector Down Detector

> Monitoring downdetector.com availability — because who watches the watchers?

A self-hosted monitoring service that tracks the availability and performance of downdetector.com. Created in response to the 2025-11-18 Cloudflare outage that took down downdetector.com itself.

## Features

- **Real-Time Status**: Live monitoring with 5-minute check intervals
- **Historical Analytics**: Uptime percentage and response time tracking
- **Clean UI**: Minimalist design inspired by mcbroken.com
- **Composable Architecture**: Extensible design for monitoring multiple sites
- **Data Engineering Ready**: DuckDB storage optimized for analytics queries

## Quick Start

```bash
# Clone the repository
git clone https://github.com/bbrewington/down-detector-down-detector.git
cd down-detector-down-detector

# Install uv (if not installed) - see docs/UV_GUIDE.md for details
curl -LsSf https://astral.sh/uv/install.sh | sh  # macOS/Linux

# Setup and run
uv venv
uv pip install -e ".[dev]"
uv run python -m src.main --dev

# Visit http://localhost:8000
```

For complete uv installation options (Windows, pipx) and all commands, see [UV_GUIDE.md](docs/UV_GUIDE.md).

## Architecture

Built with modern Python tooling:
- **FastAPI**: Async web framework with automatic API documentation
- **DuckDB**: Embedded OLAP database for efficient analytics
- **httpx**: Async HTTP client with timeout and retry support
- **APScheduler**: Reliable periodic task scheduling
- **Chart.js**: Lightweight data visualization

See [Architecture Documentation](docs/ARCHITECTURE.md) for detailed design decisions.

## Documentation

- [Architecture Overview](docs/ARCHITECTURE.md)
- [Architecture Decision Records](docs/adr/)
- [C4 Diagrams](docs/architecture/)
- [System Context](docs/architecture/c4-context.md)
- [Container Diagram](docs/architecture/c4-container.md)

## Development

```bash
# Run tests
uv run pytest

# Type checking and linting
uv run mypy src/
uv run ruff check --fix src/
```

For complete development commands (test options, coverage, etc.), see [UV_GUIDE.md](docs/UV_GUIDE.md).

## Deployment

**Development**: `uv run python -m src.main --dev`

**Production**: Run scheduler and API as separate processes. See [ARCHITECTURE.md](docs/ARCHITECTURE.md#deployment-architecture) for systemd service configuration.

## Project Goals

1. **Learn**: Explore modern Python web tooling and monitoring patterns
2. **Transparency**: Document all decisions with ADRs and architecture diagrams
3. **Composability**: Design for future expansion to monitor multiple status sites
4. **Maintainability**: Simple, well-documented code with comprehensive tests

## Tech Stack Decisions

All technology choices are documented in [Architecture Decision Records](docs/adr/):
- [ADR-001: Tech Stack Selection](docs/adr/ADR-001-tech-stack-selection.md)
- [ADR-002: Database Choice](docs/adr/ADR-002-database-choice.md)
- [ADR-003: Monitoring Strategy](docs/adr/ADR-003-monitoring-strategy.md)
- [ADR-004: Composable Architecture](docs/adr/ADR-004-composable-architecture.md)
- [ADR-005: Deployment Model](docs/adr/ADR-005-deployment-model.md)
- [ADR-006: Design Aesthetic](docs/adr/ADR-006-design-aesthetic.md)
- [ADR-007: Package Management](docs/adr/ADR-007-package-management.md)

## License

MIT License - See [LICENSE](LICENSE) for details.

## AI-Assisted Development

This project was bootstrapped with assistance from **Claude Code (Sonnet 4.5)**. All architectural decisions were human-made; AI helped structure documentation and research best practices.

**Transparency**:
- All AI contributions documented in commit messages
- Architecture Decision Records explain the "why" behind choices
- Session logs available in [docs/sessions/](docs/sessions/)
- See [AI_COLLABORATION.md](docs/AI_COLLABORATION.md) for complete transparency

## Acknowledgments

- Inspired by [mcbroken.com](https://mcbroken.com) for design aesthetic
- Created in response to the 2025-11-18 Cloudflare outage
- Built with AI assistance from Claude Code
