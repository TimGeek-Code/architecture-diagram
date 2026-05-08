---
name: architecture-diagram
description: Create professional, dark-themed architecture diagrams as standalone HTML files with SVG graphics. Use when the user asks for system architecture diagrams, infrastructure diagrams, cloud architecture visualizations, security diagrams, network topology diagrams, or any technical diagram showing system components and their relationships.
license: MIT
metadata:
  version: "1.1"
  author: TimGeek (winnerku.liu@gmail.com)
---

# Architecture Diagram Skill

Create professional technical architecture diagrams as self-contained HTML files with inline SVG graphics and CSS styling.

## Design System

### Color Palette

Use these semantic colors for component types:

| Component Type | Fill (rgba) | Stroke |
|---------------|-------------|--------|
| Frontend | `rgba(8, 51, 68, 0.4)` | `#22d3ee` (cyan-400) |
| Backend | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald-400) |
| Database | `rgba(76, 29, 149, 0.4)` | `#a78bfa` (violet-400) |
| AWS/Cloud | `rgba(120, 53, 15, 0.3)` | `#fbbf24` (amber-400) |
| Security | `rgba(136, 19, 55, 0.4)` | `#fb7185` (rose-400) |
| Message Bus | `rgba(251, 146, 60, 0.3)` | `#fb923c` (orange-400) |
| External/Generic | `rgba(30, 41, 59, 0.5)` | `#94a3b8` (slate-400) |

### Typography

Use JetBrains Mono for all text (monospace, technical aesthetic):
```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
```

Font sizes: 12px for component names, 9px for sublabels, 8px for annotations, 7px for tiny labels.

### Visual Elements

**Background:** `#020617` (slate-950) with subtle grid pattern:
```svg
<pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
  <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#1e293b" stroke-width="0.5"/>
</pattern>
```

**Component boxes:** Rounded rectangles (`rx="6"`) with 1.5px stroke, semi-transparent fills.

**Security groups:** Dashed stroke (`stroke-dasharray="4,4"`), transparent fill, rose color.

**Region boundaries:** Larger dashed stroke (`stroke-dasharray="8,4"`), amber color, `rx="12"`.

**Arrows:** Use SVG marker for arrowheads:
```svg
<marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#64748b" />
</marker>
```

**Arrow z-order:** Draw connection arrows early in the SVG (after the background grid) so they render behind component boxes. SVG elements are painted in document order, so arrows drawn first will appear behind shapes drawn later.

**Masking arrows behind transparent fills:** Since component boxes use semi-transparent fills (`rgba(..., 0.4)`), arrows behind them will show through. To fully mask arrows, draw an opaque background rect (e.g., `fill="#0f172a"`) at the same position before drawing the semi-transparent styled rect on top:
```svg
<!-- Opaque background to mask arrows -->
<rect x="X" y="Y" width="W" height="H" rx="6" fill="#0f172a"/>
<!-- Styled component on top -->
<rect x="X" y="Y" width="W" height="H" rx="6" fill="rgba(76, 29, 149, 0.4)" stroke="#a78bfa" stroke-width="1.5"/>
```

**Auth/security flows:** Dashed lines in rose color (`#fb7185`).

**Message buses / Event buses:** Small connector elements between services. Use orange color (`#fb923c` stroke, `rgba(251, 146, 60, 0.3)` fill):
```svg
<rect x="X" y="Y" width="120" height="20" rx="4" fill="rgba(251, 146, 60, 0.3)" stroke="#fb923c" stroke-width="1"/>
<text x="CENTER_X" y="Y+14" fill="#fb923c" font-size="7" text-anchor="middle">Kafka / RabbitMQ</text>
```

### Spacing Rules

**CRITICAL:** When stacking components vertically, ensure proper spacing to avoid overlaps:

- **Standard component height:** 60px for services, 80-120px for larger components
- **Minimum vertical gap between components:** 40px
- **Inline connectors (message buses):** Place IN the gap between components, not overlapping

**Example vertical layout:**
```
Component A: y=70,  height=60  → ends at y=130
Gap:         y=130 to y=170   → 40px gap, place bus at y=140 (20px tall)
Component B: y=170, height=60  → ends at y=230
```

**Wrong:** Placing a message bus at y=160 when Component B starts at y=170 (causes overlap)
**Right:** Placing a message bus at y=140, centered in the 40px gap (y=130 to y=170)

### Legend Placement

**CRITICAL:** Place legends OUTSIDE all boundary boxes (region boundaries, cluster boundaries, security groups).

- Calculate where all boundaries end (y position + height)
- Place legend at least 20px below the lowest boundary
- Expand SVG viewBox height if needed to accommodate

**Example:**
```
Kubernetes Cluster: y=30, height=460 → ends at y=490
Legend should start at: y=510 or below
SVG viewBox height: at least 560 to fit legend
```

**Wrong:** Legend at y=470 inside a cluster boundary that ends at y=490
**Right:** Legend at y=510, below the cluster boundary, with viewBox height extended

### Layout Structure

1. **Header** - Title with pulsing dot indicator, subtitle
2. **Main SVG diagram** - Contained in rounded border card
3. **Summary cards** - Grid of 3 cards below diagram with key details
4. **Footer** - Minimal metadata line

### SVG Element Grouping (REQUIRED)

**CRITICAL:** All SVG elements must be wrapped in semantic `<g>` groups for interactive editing support.

**Nodes** — each component must be wrapped in `<g class="node" data-id="UNIQUE_ID">`:
```svg
<g class="node" data-id="api-server">
    <rect x="X" y="Y" width="W" height="H" rx="6" fill="#0f172a"/>
    <rect x="X" y="Y" width="W" height="H" rx="6" fill="FILL_COLOR" stroke="STROKE_COLOR" stroke-width="1.5"/>
    <text x="CENTER_X" y="Y+20" fill="white" font-size="11" font-weight="600" text-anchor="middle">LABEL</text>
    <text x="CENTER_X" y="Y+36" fill="#94a3b8" font-size="9" text-anchor="middle">sublabel</text>
</g>
```

**Connections** — each connection must use `<path>` (never `<line>`) and include `data-waypoints` for interactive curve editing. Wrap in `<g class="connection" data-from="SOURCE_ID" data-to="TARGET_ID" data-waypoints='[...]'>`:

For **straight connections** (2 waypoints):
```svg
<g class="connection" data-from="api-server" data-to="database" data-waypoints='[{"x":X1,"y":Y1},{"x":X2,"y":Y2}]'>
    <path d="M X1 Y1 L X2 Y2" fill="none" stroke="#a78bfa" stroke-width="1.5" marker-end="url(#arrowhead)"/>
    <text x="..." y="..." fill="#94a3b8" font-size="9" text-anchor="middle">TLS</text>
</g>
```

For **curved connections** (3+ waypoints — the path passes through all points as a smooth Catmull-Rom curve):
```svg
<g class="connection" data-from="auth" data-to="api" data-waypoints='[{"x":80,"y":140},{"x":80,"y":210},{"x":220,"y":210},{"x":220,"y":278}]'>
    <path d="M 80 140 C ... (generated)" fill="none" stroke="#fb7185" stroke-width="1.5" stroke-dasharray="5,5"/>
    <text x="..." y="..." fill="#fb7185" font-size="8">JWT</text>
</g>
```

Users can add/remove curve control points interactively in edit mode (double-click on curve to add, double-click on control point to remove).

**Boundaries** (regions, security groups) — wrap in `<g class="boundary" data-id="UNIQUE_ID">`:
```svg
<g class="boundary" data-id="aws-region">
    <rect x="..." y="..." width="..." height="..." rx="12" fill="rgba(251, 191, 36, 0.05)" stroke="#fbbf24" stroke-width="1" stroke-dasharray="8,4"/>
    <text x="..." y="..." fill="#fbbf24" font-size="10" font-weight="600">AWS Region: us-west-2</text>
</g>
```

**Legend** — wrap in `<g class="legend">`.

**SVG draw order:** boundaries first (largest area at the very back, smallest in front of larger ones), then connections, then nodes (in front), then legend. The JS `sortBoundariesBySize()` function auto-sorts at runtime, but the static HTML should follow this order too.

The `data-id` values should be short, descriptive kebab-case identifiers (e.g., `api-server`, `user-db`, `auth-provider`). The `data-from`/`data-to` values on connections must match the `data-id` of their source/target nodes.

### Component Box Pattern

```svg
<g class="node" data-id="component-name">
    <rect x="X" y="Y" width="W" height="H" rx="6" fill="#0f172a"/>
    <rect x="X" y="Y" width="W" height="H" rx="6" fill="FILL_COLOR" stroke="STROKE_COLOR" stroke-width="1.5"/>
    <text x="CENTER_X" y="Y+20" fill="white" font-size="11" font-weight="600" text-anchor="middle">LABEL</text>
    <text x="CENTER_X" y="Y+36" fill="#94a3b8" font-size="9" text-anchor="middle">sublabel</text>
</g>
```

### Info Card Pattern

```html
<div class="card">
  <div class="card-header">
    <div class="card-dot COLOR"></div>
    <h3>Title</h3>
  </div>
  <ul>
    <li>• Item one</li>
    <li>• Item two</li>
  </ul>
</div>
```

## Template

Copy and customize the template at `assets/template.html`. Key customization points:

1. Update the `<title>` and header text
2. Modify SVG viewBox dimensions if needed (default: `1000 x 680`)
3. Add/remove/reposition component boxes
4. Draw connection arrows between components
5. Update the three summary cards
6. Update footer metadata

## Output

Always produce a single self-contained `.html` file with:
- Embedded CSS (no external stylesheets except Google Fonts)
- Inline SVG (no external images)
- Embedded JavaScript for interactive editing (drag-and-drop, waypoint curves, PNG export)
- The floating toolbar (hidden behind a ⋮ icon), CSS styles, and JS from `assets/template.html` must be included verbatim
- The toolbar is hidden by default so the page is screenshot-friendly

The file should render correctly when opened directly in any modern browser. Users can click "Edit" to enter edit mode, drag nodes to fix overlaps or adjust layout, drag connection labels freely (snap to curve when close), double-click connections to add/remove waypoints, and click "Save" to download the adjusted version.
