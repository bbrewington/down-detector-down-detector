# ADR-005: Deployment Model

**Status**: Accepted

**Date**: 2025-11-18

## Context

Need deployment strategy for:
- Self-hosted personal use
- Runs locally during development
- Background monitoring task (every 5 minutes)
- Web UI serving
- API endpoints

Requirements:
- Simple local deployment
- Minimal infrastructure
- Easy to start/stop
- Suitable for personal machine

## Decision

Implement **local process-based deployment** with:

### Components

1. **Background Monitor** (scheduler process)
   - APScheduler for periodic task execution
   - Runs async monitoring checks every 5 minutes
   - Writes results to DuckDB
   - Can run as systemd service or standalone process

2. **Web API** (FastAPI application)
   - Serves JSON API endpoints
   - Serves static frontend files
   - Read-only access to DuckDB
   - Runs on configurable port (default: 8000)

3. **Database** (DuckDB file)
   - Single file: `data/monitoring.duckdb`
   - Shared between scheduler and API (read-only for API)
   - Automatic creation if missing

### Deployment Modes

```bash
# Development: Run both in single process
python -m src.main --dev

# Production: Separate processes
python -m src.scheduler &  # Background monitoring
python -m src.api          # Web server
```

### Process Management Options
- **Manual**: Terminal sessions
- **systemd**: Linux service files (docs provided)
- **Docker Compose**: Optional containerization (future)

## Consequences

### Positive
- Simple deployment (Python process)
- No cloud dependencies
- Easy debugging and logs
- Low resource usage
- Direct file access to DuckDB for analysis
- Can run on personal machine indefinitely

### Negative
- Requires machine to be running for monitoring
- No automatic restart on failure (unless using systemd)
- Single point of failure
- No geographic distribution

## Implementation Details

### File Structure
```
down-detector-down-detector/
├── data/
│   ├── monitoring.duckdb     # Database file
│   └── logs/                 # Application logs
├── static/                   # Frontend assets
├── src/                      # Application code
└── config/
    └── sites.yaml            # Site configurations
```

### Configuration
```yaml
# config/app.yaml
scheduler:
  check_interval: 300  # seconds

api:
  host: "0.0.0.0"
  port: 8000

database:
  path: "data/monitoring.duckdb"

logging:
  level: INFO
  path: "data/logs/"
```

## Alternatives Considered

### Serverless (AWS Lambda + DynamoDB)
- **Pros**: No server management, auto-scaling
- **Cons**: Complexity, cost, vendor lock-in, less personal control
- **Rejected**: User explicitly requested self-hosted local deployment

### Docker Compose
- **Pros**: Container isolation, easy multi-service orchestration
- **Cons**: Adds complexity, unnecessary for local deployment
- **Deferred**: Can add later if containerization needed

### Separate Monitoring Service
- **Pros**: Clear separation, independent scaling
- **Cons**: Over-engineering for personal use
- **Rejected**: Single process sufficient for 5-minute intervals

## Future Enhancements

Consider adding:
- systemd service files for auto-restart
- Docker Compose for optional containerization
- Health check endpoint for monitoring the monitor
- Graceful shutdown handling
