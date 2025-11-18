# Architecture Overview

**Down Detector Down Detector** - A self-hosted monitoring service for tracking downdetector.com availability.

## Quick Links

- [Architecture Decision Records](adr/)
- [C4 Diagrams](architecture/)
- [System Context](architecture/c4-context.md)
- [Container Diagram](architecture/c4-container.md)

## System Purpose

Monitor the availability and performance of downdetector.com with:
- Real-time status tracking
- Historical uptime analytics
- Response time measurements
- Composable architecture for future multi-site expansion

## Design Principles

1. **Composability**: Architecture supports adding new monitoring targets without major refactoring
2. **Simplicity**: Minimal dependencies, transparent operation, easy maintenance
3. **Data-Driven**: Analytics-first design with DuckDB for efficient time-series queries
4. **Self-Hosted**: Local deployment, no cloud dependencies, full data ownership

## Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Web Framework** | FastAPI | Modern async framework, automatic API docs, type safety |
| **Database** | DuckDB | Embedded OLAP database optimized for analytics queries |
| **HTTP Client** | httpx | Async HTTP with timeout/retry support |
| **Scheduler** | APScheduler | Reliable periodic task execution |
| **Frontend** | Vanilla JS + Chart.js | Lightweight, no build step, fast loading |
| **Testing** | Pytest | Async support, extensive mocking capabilities |
| **Package Manager** | uv | 10-100x faster than pip, modern PEP 621 support |

See [ADR-001](adr/ADR-001-tech-stack-selection.md) for detailed technology selection rationale.

## Architecture Patterns

### Composable Monitors

```python
# Abstract base for extensibility
class SiteMonitor(ABC):
    @abstractmethod
    async def check_health(self) -> HealthCheckResult

    @abstractmethod
    def get_site_config(self) -> SiteConfig

# Concrete implementations
class DownDetectorMonitor(SiteMonitor):
    """HTTP-based monitoring for downdetector.com"""

# Future: class IsItDownRightNowMonitor(SiteMonitor)
# Future: class StatusIOMonitor(SiteMonitor)
```

See [ADR-004](adr/ADR-004-composable-architecture.md) for composability design.

### Data Model

```sql
-- Sites (configuration)
CREATE TABLE sites (
    id VARCHAR PRIMARY KEY,
    name VARCHAR NOT NULL,
    url VARCHAR NOT NULL,
    monitor_type VARCHAR NOT NULL,
    check_interval INTEGER NOT NULL,
    enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Health checks (time-series data)
CREATE TABLE checks (
    id INTEGER PRIMARY KEY,
    site_id VARCHAR NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    status VARCHAR NOT NULL,  -- 'up', 'down', 'degraded'
    response_time_ms INTEGER,
    http_code INTEGER,
    error_message VARCHAR,
    FOREIGN KEY (site_id) REFERENCES sites(id)
);

-- Indexes for performance
CREATE INDEX idx_checks_site_timestamp ON checks(site_id, timestamp DESC);
CREATE INDEX idx_checks_timestamp ON checks(timestamp DESC);
```

See [ADR-002](adr/ADR-002-database-choice.md) for database rationale.

## System Components

### 1. Monitoring Scheduler
- **Purpose**: Orchestrate periodic health checks
- **Technology**: APScheduler with AsyncIO
- **Schedule**: Every 5 minutes (configurable)
- **Deployment**: Background process or systemd service

### 2. HTTP Monitor
- **Purpose**: Execute HTTP health checks with resilience
- **Technology**: httpx async client
- **Features**:
  - 30-second timeout
  - Exponential backoff (2x multiplier)
  - Retry-After header respect
  - Connection pooling

### 3. Web API
- **Purpose**: Serve status data via REST API
- **Technology**: FastAPI + Uvicorn
- **Endpoints**:
  ```
  GET /api/status              # Current status
  GET /api/uptime              # Uptime statistics
  GET /api/checks?hours=24     # Check history
  GET /api/response-times      # Time-series data
  GET /docs                    # OpenAPI documentation
  ```

### 4. Frontend SPA
- **Purpose**: User interface for viewing metrics
- **Technology**: Vanilla JavaScript + Chart.js
- **Features**:
  - Live status indicator
  - Historical uptime percentage
  - Response time charts (24h/7d/30d)
  - Recent checks table
  - Auto-refresh (30s interval)

### 5. DuckDB Storage
- **Purpose**: Persistent data storage and analytics
- **Technology**: DuckDB embedded database
- **Location**: `data/monitoring.duckdb`
- **Access Pattern**:
  - Scheduler: Read/Write
  - API: Read-only

See [C4 Container Diagram](architecture/c4-container.md) for detailed component interactions.

## Data Flow

### Monitoring Cycle (Every 5 Minutes)
```
Scheduler
  ↓
Monitor.check_health()
  ↓
HTTP GET → downdetector.com
  ↓
Measure response time & status
  ↓
Write to DuckDB (checks table)
  ↓
Log result
```

### User Request Cycle
```
User Browser
  ↓
Frontend SPA (GET /api/status)
  ↓
FastAPI Handler
  ↓
Query DuckDB
  ↓
Return JSON
  ↓
Render UI with Chart.js
```

## Deployment Architecture

### Local Development
Single process mode runs both scheduler and API:
- Monitoring scheduler (background thread)
- Web API (foreground)
- Both sharing same DuckDB file

Command: `uv run python -m src.main --dev`

### Production Deployment
Separate processes for reliability:
- Terminal 1: Background monitoring (`src.scheduler`)
- Terminal 2: Web server (`src.api`)

See [UV_GUIDE.md](UV_GUIDE.md) for complete deployment commands.

### Systemd Service (Recommended)
Example service file for auto-restart and daemon management. Use `.venv/bin/python -m src.scheduler` as ExecStart.

See [ADR-005](adr/ADR-005-deployment-model.md) for complete systemd configuration.

## Configuration

### Site Configuration (`config/sites.yaml`)
```yaml
sites:
  - id: downdetector
    name: "Down Detector"
    url: "https://downdetector.com"
    monitor_type: http
    check_interval: 300  # seconds
    enabled: true
```

### Application Configuration (`config/app.yaml`)
```yaml
scheduler:
  check_interval: 300
  max_retries: 3
  backoff_multiplier: 2
  request_timeout: 30

api:
  host: "0.0.0.0"
  port: 8000
  cors_origins: ["*"]

database:
  path: "data/monitoring.duckdb"
  read_only_api: true

logging:
  level: INFO
  path: "data/logs/"
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
```

## Testing Strategy

### Unit Tests
```python
# Test monitor with mock HTTP responses
@pytest.mark.asyncio
async def test_monitor_success(mock_httpx):
    mock_httpx.get.return_value = MockResponse(200, 150)
    monitor = DownDetectorMonitor()
    result = await monitor.check_health()
    assert result.status == "up"
    assert result.response_time_ms == 150
```

### Integration Tests
```python
# Test with in-memory DuckDB
def test_storage_integration():
    db = DuckDBStorage(":memory:")
    db.save_check_result(check_result)
    uptime = db.get_uptime_percentage(hours=24)
    assert uptime >= 0.0
```

### E2E Tests
```python
# Test API endpoints
def test_api_status_endpoint(test_client):
    response = test_client.get("/api/status")
    assert response.status_code == 200
    assert "status" in response.json()
```

## Monitoring & Observability

### Logs
- **Location**: `data/logs/dddd.log`
- **Format**: Structured logging with timestamps
- **Content**:
  - Check results (success/failure)
  - Response times
  - Errors and retries
  - Scheduler operations

### Metrics (Future)
Consider adding:
- Prometheus metrics endpoint
- Grafana dashboard
- Alerting (if needed)

## Security Considerations

### Current Scope (Personal Use)
- No authentication (trusted network)
- Read-only API access to database
- No external API exposure

### Future Enhancements (If Public)
- Add authentication (API keys, OAuth)
- Rate limiting on API endpoints
- HTTPS enforcement
- Input validation and sanitization

## Performance Characteristics

### Expected Load
- **Check Frequency**: Every 5 minutes = 12 checks/hour = 288 checks/day
- **Database Growth**: ~100 KB/day, ~35 MB/year
- **API Requests**: Low traffic (personal use)

### Optimization Strategies
- DuckDB columnar storage for fast aggregations
- Indexed timestamp queries
- Connection pooling for HTTP checks
- Frontend caching with Cache-Control headers

## Future Enhancements

### Phase 2: Multi-Site Support
- Add more status-checking sites
- Comparative uptime dashboard
- Site-specific configurations

### Phase 3: Advanced Monitoring
- Multi-region checks (different geographic locations)
- Deep health checks (specific page elements)
- SSL certificate expiration monitoring
- DNS resolution time tracking

### Phase 4: Notifications (Optional)
- Email alerts on outages
- Webhook integrations
- Discord/Slack notifications

## References

- [Architecture Decision Records](adr/)
- [C4 Model Diagrams](architecture/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [DuckDB Documentation](https://duckdb.org/)
- [APScheduler Documentation](https://apscheduler.readthedocs.io/)
