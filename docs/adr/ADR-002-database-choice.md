# ADR-002: Database Choice

**Status**: Accepted

**Date**: 2025-11-18

## Context

Need a database for storing:
- Monitoring check results (timestamp, status, response_time)
- Historical uptime data for analytics
- Configuration data for monitored sites

Requirements:
- Efficient analytics queries (aggregations, time-series analysis)
- Local file-based storage (no server required)
- Support for data engineer/analyst workflows
- Easy backup and portability

## Decision

Use **DuckDB** as the primary database with:
- Schema design optimized for time-series analytics
- Parquet export capability for long-term archival
- SQL interface for ad-hoc analysis
- Embedded deployment (no separate server process)

## Consequences

### Positive
- Excellent analytical query performance (columnar storage)
- Native Parquet support for efficient data archival
- Zero-configuration embedded database
- SQL interface familiar to data professionals
- Efficient aggregations for uptime calculations
- Can export data for external analysis tools
- Built-in support for time-series operations

### Negative
- Less common than SQLite in Python ecosystem
- Write throughput lower than row-oriented databases (not an issue for 5-min intervals)
- Smaller community compared to PostgreSQL/SQLite
- May require education for contributors unfamiliar with OLAP databases

## Alternatives Considered

### SQLite
- **Pros**: Ubiquitous, well-understood, simple
- **Cons**: Row-oriented storage less efficient for analytics queries
- **Rejected**: DuckDB better suited for time-series aggregations

### PostgreSQL with TimescaleDB
- **Pros**: Excellent time-series support, mature ecosystem
- **Cons**: Requires separate server process, overkill for local deployment
- **Rejected**: Embedded deployment requirement eliminates server-based solutions

### InfluxDB
- **Pros**: Purpose-built for time-series data
- **Cons**: Separate server, InfluxQL learning curve, heavier deployment
- **Rejected**: Embedded requirement, SQL interface preference

## Schema Design Principles

- Normalize site configurations (support future multi-site expansion)
- Partition checks by site_id for composability
- Index on timestamp for efficient time-range queries
- Store response_time in milliseconds for precision
