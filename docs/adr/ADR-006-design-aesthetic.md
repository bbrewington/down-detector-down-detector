# ADR-006: Design Aesthetic

**Status**: Accepted

**Date**: 2025-11-18

## Context

Need a visual design for the status page that:
- Takes inspiration from mcbroken.com (minimal, data-focused)
- Plays it straight (no parody elements)
- Clearly displays live status and historical data
- Professional and lightweight
- Respects downdetector.com trademarks (no logo copying)

## Decision

Implement **minimalist data-focused design** inspired by mcbroken.com:

### Design Principles

1. **Minimalism First**
   - Clean typography, ample whitespace
   - Focus on data, not decoration
   - Single-page application
   - Fast loading, no unnecessary assets

2. **Data Visualization Priority**
   - Large, clear status indicator (UP/DOWN)
   - Historical uptime percentage prominently displayed
   - Response time graph (last 24h, 7d, 30d)
   - Incident timeline (recent outages)

3. **Color Scheme**
   - Neutral background (white/light gray)
   - Status colors: Green (up), Red (down), Yellow (degraded)
   - Minimal accent colors for data visualization
   - High contrast for accessibility

4. **Typography**
   - System font stack (no custom fonts)
   - Large headings for key metrics
   - Monospace for timestamps and technical data

### Component Hierarchy

```
┌─────────────────────────────────────┐
│  Down Detector Down Detector        │  ← Branding header
├─────────────────────────────────────┤
│  ● UP                               │  ← Live status (large)
│  99.7% uptime (30 days)             │  ← Key metric
├─────────────────────────────────────┤
│  [Response Time Graph]              │  ← Chart.js visualization
│  [24h] [7d] [30d]                   │  ← Time range selector
├─────────────────────────────────────┤
│  Recent Checks:                     │  ← Recent history table
│  2025-11-18 14:30 ● 234ms          │
│  2025-11-18 14:25 ● 198ms          │
└─────────────────────────────────────┘
```

### Technology Choices

- **No frontend framework**: Vanilla JavaScript
- **Charts**: Chart.js (lightweight, simple)
- **CSS**: Minimal custom CSS, CSS Grid for layout
- **Icons**: Unicode symbols (●, ✓, ✗) - no icon libraries
- **Fonts**: System font stack

### Visual Reference

Inspired by mcbroken.com:
- Single-column layout
- Large status indicator
- Minimal navigation
- Data-first presentation

## Consequences

### Positive
- Fast load times (no heavy frameworks)
- Easy to maintain (simple tech stack)
- Accessible (semantic HTML, high contrast)
- Clear information hierarchy
- Professional appearance
- No trademark/copyright concerns (original design)

### Negative
- Less interactive than modern SPA frameworks
- Manual DOM manipulation vs. reactive frameworks
- May need refactoring if complexity grows

## Implementation Details

### HTML Structure
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Down Detector Down Detector</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>Down Detector Down Detector</h1>
  </header>
  <main>
    <section id="status">
      <div class="status-indicator"></div>
      <div class="uptime-metric"></div>
    </section>
    <section id="charts">
      <canvas id="responseTimeChart"></canvas>
    </section>
    <section id="recent-checks">
      <table></table>
    </section>
  </main>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

### CSS Approach
- CSS Grid for layout
- CSS Custom Properties for theming
- Mobile-first responsive design
- No preprocessors (Sass, Less)

### JavaScript Approach
- Fetch API for data loading
- Chart.js for visualizations
- Auto-refresh every 30 seconds
- No build step required

## Alternatives Considered

### React/Vue SPA
- **Pros**: Modern, reactive, component-based
- **Cons**: Build complexity, unnecessary for simple status page
- **Rejected**: Overkill for data display use case

### Tailwind CSS
- **Pros**: Utility-first, rapid development
- **Cons**: Adds build step, larger CSS payload
- **Rejected**: Custom CSS sufficient for minimal design

### Exact downdetector.com Clone
- **Pros**: Familiar to users
- **Cons**: Trademark concerns, not original work
- **Rejected**: User specified respecting copyright/trademark

## Accessibility Considerations

- Semantic HTML structure
- ARIA labels for status indicators
- Keyboard navigation support
- Color is not the only indicator (icons + text)
- High contrast ratios (WCAG AA minimum)
