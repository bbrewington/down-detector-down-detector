# ADR-003: Monitoring Strategy

**Status**: Accepted

**Date**: 2025-11-18

## Context

Need to monitor downdetector.com availability and performance. Options include:
- Using unofficial downdetector.com API/scraper
- Direct HTTP health checks
- Browser-based monitoring

Requirements:
- Reliable detection of site availability
- Response time measurement
- Respectful of rate limits
- Simple and maintainable
- Testable with mocks

## Decision

Implement **HTTP health check monitoring** with:
- Simple GET request to downdetector.com homepage
- Response time measurement (connection + first byte)
- HTTP status code validation (200-299 = up)
- Exponential backoff on failures
- Respect for Retry-After headers
- 5-minute check interval
- 30-second request timeout

**No API usage**: Downdetector.com has no official public API, and unofficial scrapers are fragile due to Cloudflare protection.

## Consequences

### Positive
- Simple, maintainable implementation
- No dependency on unofficial/fragile scrapers
- Direct measurement of user-facing availability
- Easy to mock for testing
- Respectful of target site resources
- Clear failure modes

### Negative
- Cannot detect partial outages (API down, site up)
- No insight into specific service degradations
- Basic health check only (no deep monitoring)

## Implementation Details

### Request Configuration
```python
timeout = 30  # seconds
check_interval = 300  # 5 minutes
max_retries = 3
backoff_multiplier = 2
```

### Success Criteria
- HTTP status 200-299
- Response received within timeout
- Valid HTTP response structure

### Failure Scenarios
- Connection timeout
- HTTP status >= 400
- SSL/TLS errors
- DNS resolution failures

### Rate Limiting Respect
- Check `Retry-After` header on 429/503 responses
- Implement exponential backoff on failures
- Maximum backoff: 15 minutes

## Alternatives Considered

### Unofficial downdetector-api Package
- **Pros**: Access to detailed outage data
- **Cons**: Fragile (Cloudflare protection), unofficial, maintenance burden
- **Rejected**: Reliability concerns, not our data to scrape

### Browser-Based Monitoring (Playwright)
- **Pros**: Detect JavaScript-dependent failures
- **Cons**: Resource-heavy, complex, slower checks
- **Rejected**: Overkill for basic availability monitoring

### Multi-Endpoint Monitoring
- **Pros**: Detect partial outages
- **Cons**: More complex, higher request volume
- **Rejected**: Start simple, can expand later

## Future Enhancements

Consider adding (if needed):
- DNS resolution time tracking
- SSL certificate expiration monitoring
- Multi-region checks (from different geographic locations)
- Deep health checks (specific page elements)
