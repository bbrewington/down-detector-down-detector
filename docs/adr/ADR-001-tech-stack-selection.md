# ADR-001: Tech Stack Selection

**Status**: Accepted

**Date**: 2025-11-18

## Context

Need to select a modern Python web framework for building a monitoring service that:
- Provides HTTP endpoints for status data
- Serves static frontend assets
- Runs background monitoring tasks
- Is maintainable, transparent, and lightweight
- Uses state-of-the-art Python tooling

## Decision

Use **FastAPI** as the primary web framework with supporting tools:
- **FastAPI**: Modern async web framework with automatic OpenAPI docs
- **Uvicorn**: ASGI server for production deployment
- **Httpx**: Modern async HTTP client for monitoring checks
- **Pydantic**: Data validation and settings management
- **Pytest**: Testing framework with async support

## Consequences

### Positive
- FastAPI provides automatic API documentation (OpenAPI/Swagger)
- Native async/await support for efficient concurrent monitoring
- Type hints throughout improve maintainability
- Excellent performance characteristics
- Strong ecosystem and active development
- Built-in dependency injection simplifies testing

### Negative
- Slightly more complex than Flask for simple use cases
- Requires understanding of async/await patterns
- ASGI deployment differs from traditional WSGI

## Alternatives Considered

### Flask
- **Pros**: Simpler, more established, larger ecosystem
- **Cons**: No native async support, less modern tooling
- **Rejected**: Async capabilities critical for efficient monitoring

### Django
- **Pros**: Batteries-included, excellent admin interface
- **Cons**: Heavy for this use case, ORM overkill for DuckDB
- **Rejected**: Too much overhead for a focused monitoring service

### Starlette (FastAPI's foundation)
- **Pros**: More lightweight, same async capabilities
- **Cons**: Less documentation, no automatic OpenAPI generation
- **Rejected**: FastAPI's conveniences worth minimal overhead
