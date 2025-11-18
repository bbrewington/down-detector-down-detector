# Project Summary: Down Detector Down Detector

**Status**: Requirements Complete, Ready for Implementation
**Date**: 2025-11-18
**Type**: Self-Hosted Monitoring Service

## Overview

A self-hosted monitoring service that tracks the availability and performance of downdetector.com. The ironic meta-monitoring service ("who watches the watchers?") serves both as a functional tool and a learning project for modern Python web development.

## Project Genesis

Created in response to the 2025-11-18 Cloudflare outage that caused downdetector.com itself to go offline, highlighting the meta-irony of status-checking services being unavailable.

## Core Requirements

### Functional Requirements
1. **Monitoring**:
   - HTTP health checks every 5 minutes
   - Response time measurement
   - Status validation (up/down/degraded)
   - Exponential backoff and rate limit respect

2. **Data Storage**:
   - Historical check results in DuckDB
   - Site configuration management
   - Time-series data optimized for analytics

3. **User Interface**:
   - Live status indicator
   - Historical uptime percentage
   - Response time charts (24h/7d/30d)
   - Recent checks timeline
   - Minimalist design (mcbroken.com style)

4. **API**:
   - REST endpoints for status data
   - OpenAPI documentation
   - JSON response format

### Non-Functional Requirements
1. **Architecture**:
   - Composable design supporting future multi-site expansion
   - Clear separation of concerns
   - Abstract base classes for extensibility

2. **Technology**:
   - Python with state-of-the-art tooling
   - FastAPI web framework
   - DuckDB embedded database
   - Vanilla JavaScript frontend (no build step)

3. **Deployment**:
   - Self-hosted, local deployment
   - No cloud dependencies
   - Systemd service support

4. **Testing**:
   - Pytest with async support
   - Mock HTTP responses for testing
   - In-memory DuckDB for integration tests

5. **Documentation**:
   - Architecture Decision Records (ADRs)
   - C4 diagrams in MermaidJS
   - Comprehensive architecture documentation

## Key Design Decisions

### Technology Stack
- **Web**: FastAPI (async, type-safe, automatic docs)
- **Database**: DuckDB (OLAP optimized, embedded, analytics-friendly)
- **HTTP Client**: httpx (async, timeout/retry support)
- **Scheduler**: APScheduler (reliable periodic tasks)
- **Frontend**: Vanilla JS + Chart.js (lightweight, no build)

Rationale documented in [ADR-001](adr/ADR-001-tech-stack-selection.md).

### Database Choice
DuckDB selected over SQLite and PostgreSQL for:
- Columnar storage optimized for analytics queries
- Parquet export for long-term archival
- Embedded deployment (no server)
- Efficient time-series aggregations

Rationale documented in [ADR-002](adr/ADR-002-database-choice.md).

### Monitoring Strategy
HTTP health checks chosen over:
- Unofficial downdetector APIs (fragile, Cloudflare-blocked)
- Browser-based monitoring (resource-heavy)

Simple GET requests with:
- 30-second timeout
- Exponential backoff
- Retry-After header respect

Rationale documented in [ADR-003](adr/ADR-003-monitoring-strategy.md).

### Composable Architecture
Service-oriented design with:
- Abstract `SiteMonitor` base class
- Site-specific implementations
- Configuration-driven site management
- Shared database schema with `site_id` partitioning

Supports future expansion to monitor other status sites (isitdownrightnow.com, status.io, etc.).

Rationale documented in [ADR-004](adr/ADR-004-composable-architecture.md).

### Deployment Model
Local process-based deployment:
- Separate scheduler and web processes
- Shared DuckDB file
- Systemd service support
- No serverless/cloud complexity

Rationale documented in [ADR-005](adr/ADR-005-deployment-model.md).

### Design Aesthetic
Minimalist, data-focused UI inspired by mcbroken.com:
- Clean typography, ample whitespace
- Large status indicator
- Response time charts (Chart.js)
- No frontend framework (Vanilla JS)
- System fonts, minimal CSS

Rationale documented in [ADR-006](adr/ADR-006-design-aesthetic.md).

## Architecture

### System Context
```
User → Down Detector Down Detector → downdetector.com
       (views status)                (monitors)
```

### Containers
1. **Web Application** (FastAPI) - REST API and static file serving
2. **Monitoring Scheduler** (APScheduler) - Periodic health checks
3. **HTTP Monitor** (httpx) - Health check execution
4. **Database** (DuckDB) - Data persistence
5. **Frontend SPA** (Vanilla JS) - User interface

See [C4 Diagrams](architecture/) for detailed architecture views.

## Project Goals

1. **Functional**: Create a working monitoring service for downdetector.com
2. **Educational**: Learn modern Python web development patterns
3. **Transparent**: Document all decisions with ADRs and diagrams
4. **Maintainable**: Clean code, comprehensive tests, clear documentation
5. **Extensible**: Composable architecture for future expansion

## Success Criteria

- [ ] Successfully monitors downdetector.com availability
- [ ] Displays real-time status and historical uptime
- [ ] Stores check results in DuckDB
- [ ] Provides REST API for status data
- [ ] Clean, minimalist UI inspired by mcbroken.com
- [ ] Comprehensive test coverage with mocks
- [ ] Complete documentation (ADRs, C4 diagrams, README)
- [ ] Runs locally with simple deployment

## Future Enhancements

### Phase 2: Multi-Site Support
- Monitor additional status-checking sites
- Comparative uptime dashboard
- Site-specific configurations

### Phase 3: Advanced Monitoring
- Multi-region checks
- Deep health checks (page elements)
- SSL certificate monitoring
- DNS resolution tracking

### Phase 4: Notifications (Optional)
- Email alerts
- Webhook integrations
- Discord/Slack notifications

## Repository Structure

```
down-detector-down-detector/
├── docs/
│   ├── ARCHITECTURE.md           # Comprehensive architecture doc
│   ├── PROJECT_SUMMARY.md        # This file
│   ├── adr/                      # Architecture Decision Records
│   │   ├── README.md
│   │   ├── ADR-001-tech-stack-selection.md
│   │   ├── ADR-002-database-choice.md
│   │   ├── ADR-003-monitoring-strategy.md
│   │   ├── ADR-004-composable-architecture.md
│   │   ├── ADR-005-deployment-model.md
│   │   └── ADR-006-design-aesthetic.md
│   └── architecture/             # C4 diagrams
│       ├── README.md
│       ├── c4-context.md         # System Context diagram
│       └── c4-container.md       # Container diagram
├── src/
│   ├── core/                     # Shared abstractions
│   ├── monitors/                 # Site-specific monitors
│   ├── scheduler/                # Task scheduling
│   ├── api/                      # FastAPI routes
│   └── config/                   # Configuration
├── static/                       # Frontend assets
├── tests/                        # Test suite
├── data/                         # Database and logs
├── config/                       # Configuration files
├── README.md                     # Project README
├── requirements.txt              # Python dependencies
└── requirements-dev.txt          # Development dependencies
```

## Next Steps: Implementation

With requirements complete and documented, ready to proceed to implementation phase:

1. **Setup**: Create project structure, configure tooling
2. **Core**: Implement database schema, base abstractions
3. **Monitor**: Build HTTP monitor with timeout/retry
4. **Scheduler**: Implement periodic task execution
5. **API**: Create FastAPI endpoints
6. **Frontend**: Build minimalist UI with Chart.js
7. **Testing**: Comprehensive test suite with mocks
8. **Documentation**: API docs, deployment guide

## Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [DuckDB Documentation](https://duckdb.org/)
- [APScheduler Documentation](https://apscheduler.readthedocs.io/)
- [Chart.js Documentation](https://www.chartjs.org/)
- [mcbroken.com](https://mcbroken.com) (design inspiration)

---

**Ready for Implementation**: All requirements defined, decisions documented, architecture specified.
