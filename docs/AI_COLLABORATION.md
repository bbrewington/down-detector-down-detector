# AI Collaboration in This Project

This project was bootstrapped and is maintained with assistance from AI tools. This document provides transparency about AI usage and guidance for future contributions.

## Current Status

**Phase**: Requirements & Architecture (Complete) → Implementation (Next)
**Last Updated**: 2025-11-18

## Tools Used

| Tool | Model | Purpose | Date |
|------|-------|---------|------|
| Claude Code | Sonnet 4.5 | Requirements gathering, architecture design, documentation | Nov 2025 |

## What AI Generated

### ✅ Full AI Generation

These were created entirely by AI based on human requirements:

**Architecture Documentation**:
- `docs/adr/` - All 7 Architecture Decision Records
- `docs/architecture/` - C4 diagrams (Context, Container) in MermaidJS
- `docs/ARCHITECTURE.md` - Complete architecture overview
- `docs/PROJECT_SUMMARY.md` - Requirements specification

**Development Setup**:
- `pyproject.toml` - Dependencies and tool configuration
- `.python-version` - Python version specification
- `docs/UV_GUIDE.md` - Complete uv command reference
- `CLAUDE.md` - AI/developer onboarding guide
- `.gitignore` - Project-specific entries

### 🤝 AI-Assisted (Human-Guided)

Human made decisions, AI structured and documented:

**Technology Stack**:
- Human chose: Python, FastAPI, DuckDB, httpx, APScheduler, uv
- AI documented: Rationale in ADRs, alternatives considered, trade-offs

**Architecture**:
- Human required: Composability, self-hosted, simple
- AI designed: SiteMonitor pattern, directory structure, data model

**Documentation Format**:
- Human requested: ADRs, C4 diagrams, transparency
- AI generated: Structured documents following standards

### ❌ Not AI-Generated

100% human decisions:

- Project concept (meta-monitoring downdetector.com)
- All technology choices and preferences
- Architecture requirements (composability, self-hosted)
- Design aesthetic (mcbroken.com-inspired, minimalist)
- Documentation standards (ADRs, C4, transparency)
- Data engineering preferences (DuckDB over SQLite)

## AI Collaboration Methodology

### Phase 1: Requirements Discovery (Brainstorming)

**Pattern**: Socratic dialogue
- AI asks clarifying questions
- Human provides detailed requirements
- Iterative refinement through conversation

**Example**:
```
AI: "Should this monitor only downdetector.com or other sites?"
Human: "Start with downdetector.com, but design for future expansion"
AI: [Creates composable architecture with SiteMonitor pattern]
```

### Phase 2: Architecture Documentation

**Pattern**: Structured documentation generation
- AI researches best practices (e.g., DuckDB vs SQLite)
- AI generates ADRs with alternatives and trade-offs
- Human reviews and approves decisions

**Quality Control**:
- All ADRs include "Alternatives Considered" section
- Rationale documented for transparency
- Human validates technical accuracy

### Phase 3: Development Setup

**Pattern**: Configuration and tooling
- AI generates `pyproject.toml` with appropriate dependencies
- AI creates comprehensive guides (UV_GUIDE.md)
- Human validates package versions and settings

### Phase 4: Code Hygiene

**Pattern**: DRY improvements
- Human identifies duplication
- AI refactors with single source of truth
- Human approves architectural approach

## Session Logs

Detailed conversation records:
- [2025-11-18: Initial Brainstorming](sessions/2025-11-18-initial-brainstorm.md)

## Future AI Usage Guidelines

This project welcomes AI-assisted contributions. When using AI tools:

### 1. Documentation

**In Commit Messages**:
```
feat: add HTTP monitoring with retry logic

- Implemented SiteMonitor abstract base class
- Added DownDetectorMonitor with exponential backoff
- Tests with mocked HTTP responses

AI-assisted: Claude Code helped structure the retry logic
```

**In Pull Requests**:
```markdown
## Summary
[Your changes]

## AI Assistance
- Tool: Claude Code / GitHub Copilot / etc.
- Scope: Code generation / Documentation / Refactoring
- Review: All AI-generated code reviewed and tested
```

### 2. Code Review

All AI-generated code must:
- [ ] Pass all tests (pytest with coverage)
- [ ] Pass type checking (mypy strict mode)
- [ ] Pass linting (ruff)
- [ ] Be reviewed by a human for correctness
- [ ] Include appropriate error handling
- [ ] Have clear documentation

### 3. Testing Requirements

AI-generated code should include:
- Unit tests with mocks
- Type hints (mypy strict compliance)
- Docstrings for public APIs
- Error handling for edge cases

### 4. Attribution

Be transparent about AI usage:
- Note AI tool in commit messages
- Indicate scope (full generation vs. assistance)
- Document any significant AI decisions

## Architecture Decision Records (ADRs)

All major decisions are documented in ADRs, whether human or AI-assisted:

| ADR | Decision | Human/AI Split |
|-----|----------|----------------|
| 001 | Tech Stack (FastAPI) | Human choice, AI documented alternatives |
| 002 | Database (DuckDB) | Human preference, AI research |
| 003 | Monitoring Strategy | Human requirements, AI implementation details |
| 004 | Composable Architecture | Human requirement, AI pattern design |
| 005 | Deployment Model | Human choice (self-hosted), AI documentation |
| 006 | Design Aesthetic | Human preference (mcbroken.com), AI spec |
| 007 | Package Management (uv) | Human choice, AI research and setup |

## AI Limitations Encountered

Important to document what AI couldn't do:

1. **Technology Decisions**: AI provided options, but human made final choices
2. **Business Requirements**: Human had to specify all requirements
3. **Design Preferences**: Aesthetic choices are subjective and human-driven
4. **Implementation Priority**: AI can suggest, but human prioritizes

## Benefits of AI Collaboration

What worked well in this project:

1. **Speed**: Complete architecture documented in 1 session vs. days
2. **Thoroughness**: ADRs include alternatives and trade-offs
3. **Best Practices**: AI researched current best practices (uv, DuckDB)
4. **Consistency**: Documentation follows consistent format
5. **DRY Improvements**: AI identified and fixed duplication patterns

## Anti-Patterns to Avoid

Lessons learned:

1. ❌ **Blindly accepting AI suggestions**: Always validate technical accuracy
2. ❌ **Skipping human review**: AI can miss edge cases
3. ❌ **Over-engineering**: AI may suggest complex solutions; keep it simple
4. ❌ **Unclear requirements**: AI needs clear direction; vague prompts yield poor results
5. ❌ **Ignoring alternatives**: Always ask AI to provide alternatives

## Questions About AI Usage?

- **General Discussion**: Open a GitHub Discussion
- **Specific Concerns**: Create an issue with label `ai-collaboration`
- **Contributing**: See CONTRIBUTING.md (when created)

## Resources

- [Session logs](sessions/) - Detailed conversation records
- [ADRs](adr/) - All architecture decisions
- [CLAUDE.md](../CLAUDE.md) - AI/developer onboarding guide
- [Claude Code Documentation](https://code.claude.com/docs)

---

**Transparency Commitment**: This project maintains full transparency about AI usage. All AI contributions are documented, and human decision-making is clearly delineated.
