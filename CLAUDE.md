# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Down Detector Down Detector** monitors downdetector.com availability and performance. It's a self-hosted service built with modern Python tooling, designed for composability to support monitoring multiple status-checking sites in the future.

## AI Collaboration in This Project

This project was bootstrapped with AI assistance (Claude Code). All future AI-assisted contributions should follow documented guidelines.

**Key Points**:
- Technology decisions = Human
- Documentation structure = AI-assisted
- All AI contributions must be documented in commits
- See `docs/AI_COLLABORATION.md` for complete guidelines

**When Using AI for Development**:

1. **Document in Commit Messages**:
   ```
   feat: add retry logic to HTTP monitor

   - Implemented exponential backoff
   - Added Retry-After header handling

   AI-assisted: Claude Code helped structure the retry logic
   ```

2. **Quality Requirements**:
   - [ ] All AI-generated code passes tests (pytest)
   - [ ] Type checking passes (mypy strict mode)
   - [ ] Linting passes (ruff)
   - [ ] Human reviewed for correctness
   - [ ] Includes error handling

3. **Attribution Standards**:
   - Note AI tool used (Claude Code, Copilot, etc.)
   - Specify scope (full generation vs. assistance)
   - Human takes responsibility for all merged code

See `docs/AI_COLLABORATION.md` for detailed guidelines and `docs/sessions/` for session logs.

## Package Management: uv (CRITICAL)

This project uses **uv** for all Python package management. **Never use pip directly.**

All dependencies are in `pyproject.toml` (PEP 621 format). See `docs/UV_GUIDE.md` for complete command reference.

## Development Commands

**Quick Reference** (all commands use `uv run`):
```bash
pytest                           # Run tests
pytest tests/test_monitor.py -v  # Specific test
mypy src/                        # Type checking (strict)
ruff check --fix src/            # Lint and auto-fix
```

**Running**:
- Development: `uv run python -m src.main --dev` (single process)
- Production: Run `src.scheduler` and `src.api` separately

See `docs/UV_GUIDE.md` for complete command reference and options.

## Architecture: Composable Monitor Pattern

The system is designed around a **composable architecture** that separates monitoring logic from infrastructure. This is critical for understanding code organization.

### Core Abstraction (src/core/monitor.py)

```python
class SiteMonitor(ABC):
    @abstractmethod
    async def check_health(self) -> HealthCheckResult

    @abstractmethod
    def get_site_config(self) -> SiteConfig
```

All monitors inherit from `SiteMonitor`. This pattern allows adding new sites without touching core infrastructure.

### Directory Structure

```
src/
├── core/           # Shared abstractions (SiteMonitor, models, storage)
├── monitors/       # Site-specific implementations (downdetector.py, ...)
├── scheduler/      # APScheduler orchestration (every 5 minutes)
├── api/            # FastAPI REST endpoints (read-only DB access)
└── config/         # Configuration management
```

**Key Pattern**: `scheduler` → `monitors` → `storage`, while `api` → `storage` (read-only).

### Data Flow

**Monitoring Cycle (every 5 minutes)**:
```
Scheduler (APScheduler)
  → Monitor.check_health() (httpx with timeout/retry)
    → HTTP GET to target site
    → Measure response time
  → Storage.save_check_result() (DuckDB write)
```

**User Request**:
```
User → Frontend (Vanilla JS)
  → API (FastAPI, read-only)
    → Storage (DuckDB query)
```

### Database: DuckDB (OLAP, not OLTP)

**Critical**: DuckDB is an OLAP database optimized for analytics queries, not transaction processing. Design queries for:
- Time-series aggregations
- Columnar storage benefits
- Efficient `GROUP BY` and window functions

**Schema**:
```sql
sites (id, name, url, monitor_type, check_interval, enabled)
checks (id, site_id, timestamp, status, response_time_ms, http_code, error_message)
```

Index on `(site_id, timestamp DESC)` for performance.

## Configuration

### Sites Configuration (config/sites.yaml)
```yaml
sites:
  - id: downdetector
    name: "Down Detector"
    url: "https://downdetector.com"
    monitor_type: http
    check_interval: 300  # seconds
    enabled: true
```

New sites are added here, then a new monitor class is created in `src/monitors/`.

### Application Configuration (config/app.yaml)
```yaml
scheduler:
  check_interval: 300
  max_retries: 3
  backoff_multiplier: 2
  request_timeout: 30

database:
  path: "data/monitoring.duckdb"
  read_only_api: true  # API has read-only access
```

## Monitoring Strategy: HTTP Health Checks

**Important Decision** (ADR-003): Uses simple HTTP GET requests, NOT unofficial downdetector APIs or scrapers.

**Rationale**: Downdetector has no official API, and unofficial scrapers are blocked by Cloudflare. HTTP health checks are reliable, respectful, and testable.

**Implementation Requirements**:
- 30-second timeout
- Exponential backoff on failures (2x multiplier)
- Respect `Retry-After` headers
- Success: HTTP 200-299
- Failure: timeout, 4xx/5xx, connection errors

## Testing with Mocks

All HTTP interactions must be mockable for testing:

```python
@pytest.mark.asyncio
async def test_monitor_success(mock_httpx):
    mock_httpx.get.return_value = MockResponse(200, 150)
    monitor = DownDetectorMonitor()
    result = await monitor.check_health()
    assert result.status == "up"
    assert result.response_time_ms == 150
```

Use `pytest-asyncio` for async tests and `pytest-mock` for mocking.

## Frontend: Vanilla JS (No Build Step)

**Critical Design Decision** (ADR-006): No frontend framework (React, Vue). Uses Vanilla JavaScript + Chart.js.

**Rationale**: Minimalist design inspired by mcbroken.com. No build complexity, fast loading, simple maintenance.

**Technologies**:
- Vanilla JavaScript (ES6+)
- Chart.js for response time graphs
- CSS Grid for layout
- Auto-refresh every 30 seconds

## Type Safety: mypy Strict Mode

Type hints are required for all functions. `mypy` runs in strict mode:

```python
# Good
async def check_health(self) -> HealthCheckResult:
    pass

# Bad (will fail mypy)
async def check_health(self):
    pass
```

Configure type hint ignores in `pyproject.toml` only for external libraries without stubs.

## Documentation Standards

All architectural decisions are documented in ADRs (`docs/adr/`):
- ADR-001: Tech Stack Selection (FastAPI, DuckDB, httpx)
- ADR-002: Database Choice (DuckDB over SQLite/Postgres)
- ADR-003: Monitoring Strategy (HTTP checks, no APIs)
- ADR-004: Composable Architecture (SiteMonitor pattern)
- ADR-005: Deployment Model (local, self-hosted)
- ADR-006: Design Aesthetic (minimalist, mcbroken-inspired)
- ADR-007: Package Management (uv over pip/poetry)

When making significant architectural changes, create a new ADR following the existing format.

## Adding a New Monitoring Target

1. Create `src/monitors/new_site.py` implementing `SiteMonitor`
2. Add site configuration to `config/sites.yaml`
3. Register monitor in factory (once implemented)
4. Add tests in `tests/test_new_site.py`
5. No database schema changes needed (partitioned by `site_id`)

## Performance Characteristics

- **Check Frequency**: Every 5 minutes (configurable)
- **Database Growth**: ~100 KB/day, ~35 MB/year
- **API Performance**: Optimized for read-heavy analytics queries
- **Target**: Personal use, low traffic

## Key External Dependencies

- **FastAPI**: Async web framework, automatic OpenAPI docs at `/docs`
- **httpx**: Async HTTP client (connection pooling, timeout/retry)
- **DuckDB**: Embedded OLAP database (file: `data/monitoring.duckdb`)
- **APScheduler**: Periodic task scheduling (AsyncIO executor)
- **Pydantic v2**: Data validation and settings management

## Files NOT to Edit

- `uv.lock`: Generated by uv, commit to git for reproducible builds
- `.python-version`: Specifies Python 3.11 requirement
- Historical documentation in `docs/`: Reference only, update ADRs for new decisions

## Common Pitfalls

1. **Don't use pip**: Always use `uv` for package management
2. **DuckDB is OLAP**: Design for analytics, not high-frequency writes
3. **API is read-only**: Scheduler writes, API only reads from database
4. **Async everywhere**: All I/O operations must be async (httpx, FastAPI)
5. **Type hints required**: mypy strict mode will fail without complete type hints
6. **No frontend build**: Keep frontend simple, no webpack/vite/bundlers
7. **Document AI usage**: Note AI assistance in commit messages (see `docs/AI_COLLABORATION.md`)

## AI-Assisted Development Guidelines

When using AI tools (Claude Code, Copilot, etc.) for this project:

### Before Merging AI-Generated Code

- **Test**: Run full test suite with coverage
- **Type Check**: `uv run mypy src/` must pass
- **Lint**: `uv run ruff check src/` must pass
- **Review**: Manually review for correctness and edge cases
- **Document**: Note AI assistance in commit message

### Commit Message Format

```
<type>: <brief description>

<detailed description>

<list of changes>

AI-assisted: <tool name> helped with <specific contribution>
```

### Example

```
feat: add exponential backoff to HTTP monitor

- Implemented retry logic with 2x backoff multiplier
- Added Retry-After header parsing
- Tests with mocked HTTP failures

AI-assisted: Claude Code structured the retry state machine
```

### What to Document

- **Full Generation**: "AI-generated: Claude Code created initial implementation"
- **Assistance**: "AI-assisted: GitHub Copilot helped with error handling"
- **Refactoring**: "AI-assisted: Claude Code refactored for DRY principles"

### Architecture Decisions

For significant architectural changes, create an ADR (Architecture Decision Record):
- Follow format in `docs/adr/`
- Document alternatives considered
- Note if AI-assisted in research phase
- Human approves final decision

See `docs/AI_COLLABORATION.md` for complete guidelines and examples.
