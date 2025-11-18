# ADR-004: Composable Architecture

**Status**: Accepted

**Date**: 2025-11-18

## Context

Initial focus is monitoring downdetector.com, but architecture should support:
- Adding more sites in the future (isitdownrightnow.com, status.io, etc.)
- Different monitoring strategies per site (API, HTTP, browser)
- Shared infrastructure (database, scheduler, API)
- Independent site configurations

Need architecture that balances simplicity now with extensibility later.

## Decision

Implement **service-oriented composable architecture**:

### Core Abstractions

```python
# Abstract base for all monitors
class SiteMonitor(ABC):
    @abstractmethod
    async def check_health(self) -> HealthCheckResult

    @abstractmethod
    def get_site_config(self) -> SiteConfig

# Concrete implementation
class DownDetectorMonitor(SiteMonitor):
    """HTTP-based monitoring for downdetector.com"""
    pass
```

### Directory Structure
```
src/
├── core/           # Shared abstractions and interfaces
│   ├── models.py   # Pydantic models (HealthCheckResult, SiteConfig)
│   ├── monitor.py  # SiteMonitor ABC
│   └── storage.py  # Database interface
├── monitors/       # Site-specific monitor implementations
│   ├── downdetector.py
│   └── [future: isitdownrightnow.py, status_io.py]
├── scheduler/      # Task scheduling
├── api/            # FastAPI routes
└── config/         # Configuration management
```

### Configuration-Driven

Sites defined in `config/sites.yaml`:
```yaml
sites:
  - id: downdetector
    name: "Down Detector"
    url: "https://downdetector.com"
    monitor_type: http
    check_interval: 300
    enabled: true
```

## Consequences

### Positive
- Easy to add new sites (implement SiteMonitor, add config entry)
- Shared database schema supports multi-site queries
- Different monitoring strategies per site
- Test monitors in isolation
- Clear separation of concerns
- Can enable/disable sites via configuration

### Negative
- More abstraction than needed for single site
- Initial overhead for interfaces that might not be reused
- Risk of over-engineering

## Design Principles

### Composability
- Each monitor is independent, interchangeable
- Shared database schema with `site_id` partitioning
- Common interfaces for scheduling and storage

### Progressive Enhancement
- Start with single site, add complexity as needed
- Don't build features until second site requires them
- YAGNI principle for abstractions

### Testing Strategy
- Mock monitors for API testing
- In-memory DuckDB for integration tests
- Test composability by creating dummy second monitor

## Future Site Integration

Adding a new site requires:
1. Implement `SiteMonitor` subclass in `monitors/`
2. Add site configuration to `config/sites.yaml`
3. Register monitor in factory/registry
4. Deploy (no schema changes needed)

## Alternatives Considered

### Monolithic Single-Site Implementation
- **Pros**: Simpler, less abstraction, faster initial development
- **Cons**: Major refactoring needed to add second site
- **Rejected**: User explicitly requested composability

### Plugin Architecture
- **Pros**: Maximum flexibility, runtime loading
- **Cons**: Complex, overkill, harder to test
- **Rejected**: Over-engineering for foreseeable needs

### Microservices (One Service Per Site)
- **Pros**: Maximum isolation
- **Cons**: Deployment complexity, shared-nothing overhead
- **Rejected**: Self-hosted personal use doesn't justify complexity
