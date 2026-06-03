# Architecture Diagram Guide

Create professional technical architecture diagrams as self-contained HTML files with SVG graphics and CSS styling.

### Typography

Use SF Mono if given as part of input

Font sizes: 14px for component names, 11px for sublabels, 10px for annotations, 9px (Strictly Uppercase)

## Logo usage
Architecture diagram icons are available from the Archicons registry:

https://archicons.oltpdba.workers.dev/registry.json

Use the registry to find icons by `id`, `name`, `provider`, or `category`.
Each icon entry includes a hosted SVG URL in the `url` field.

Do not use hosted icon URLs as SVG `<image href="...">` references in exportable diagrams.

For diagrams that include PNG/PDF export, fetch the icon SVG during generation and paste the SVG markup inline inside the diagram. Keep the icon as plain inline `<svg>`, `<g>`, `<path>`, `<rect>`, `<circle>`, `<polygon>`, and related SVG shape elements. External SVG files referenced through `<image href="...">`, and sometimes `<symbol>/<use>` references, can display correctly in the browser but export as broken-image placeholders in html2canvas captures.

Example registry entry:

{
  "id": "azure.azure-sql",
  "name": "Azure SQL",
  "provider": "Microsoft Azure",
  "category": "azure",
  "path": "icons/azure/azure-sql.svg",
  "url": "https://archicons.oltpdba.workers.dev/icons/azure/azure-sql.svg"
}

Only use an icon if the logo is available in the registry. Do not create custom or mock logos.

Wrong for exportable diagrams:

```html
<image href="https://archicons.oltpdba.workers.dev/icons/aws/aws-glue.svg" x="18" y="16" width="28" height="28" preserveAspectRatio="xMidYMid meet"></image>
```

Right for exportable diagrams:

```html
<svg x="18" y="16" width="28" height="28" viewBox="0 0 80 80" aria-label="AWS Glue">
  <!-- Paste the real registry SVG paths/shapes here. Do not use external image hrefs. -->
</svg>
```

Use `useCORS: true` in html2canvas captures for any remaining CDN-hosted non-SVG assets, but do not rely on it for Archicons SVG export. The Archicons SVG responses may not include browser CORS headers, so external icon references are not export-safe.

Export preflight before final output:
- Search the generated HTML for `<image`, `href="https://archicons`, and `xlink:href="https://archicons`
- If any are present, replace them with inline SVG markup before returning the file
- A generated exportable diagram should have `document.querySelectorAll("svg image").length === 0`

### Workload Inputs Placement

Prefer summarizing workload inputs in the page header/subtitle instead of adding a large top badge inside the SVG.

If workload inputs must appear inside the SVG:
- Place the workload summary outside all platform, region, cluster, and security boundaries
- Prefer a compact footer strip below the lowest boundary and above the legend, or place it below the legend if space is tight
- Leave at least 16px vertical clearance from every boundary line and flow arrow
- Expand the SVG viewBox height if needed
- Skip repeated workload detail when the header already includes volume, cadence, load type, and batch/streaming mix

**Wrong:** Workload badge at `y=24..68` while a platform boundary starts at `y=58`
**Right:** No workload badge, or a compact workload strip below the lowest boundary with clear vertical separation

### Design Notes

Do not add a separate "Design Notes" strip or callout to generated diagrams unless explicitly requested by the user.

Keep architecture decisions visible through the actual component layout, labels, sublabels, and legend.

### Animation

When line animation is enabled, animate only connector or flow lines.

Do not add decorative moving dots, pulsing balls, or animated badges inside component boxes unless explicitly requested by the user.

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

### Export Toolbar (built-in)

Every diagram ships with a single unobtrusive `⋯` toggle in the header. Click it to reveal three buttons — 📋 Copy (high-DPI PNG to clipboard, scale: 2), 🖼️ PNG (high-DPI PNG download), 📄 PDF (PNG embedded in a one-page PDF via jsPDF). The toolbar collapses back to the icon by default so it doesn't clutter the diagram. All three formats use the same html2canvas capture (with the toolbar excluded and 32px padding around the content), so PDF preserves the dark theme without going through the browser's print dialog.

When generating a new diagram, keep these intact in the template:
- The two CDN scripts in `<head>` (pinned versions, with Subresource Integrity hashes and `crossorigin="anonymous"`):
  - `https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js` — `integrity="sha384-ZZ1pncU3bQe8y31yfZdMFdSpttDoPmOZg2wguVK9almUodir1PghgT0eY7Mrty8H"`
  - `https://cdn.jsdelivr.net/npm/jspdf@2.5.2/dist/jspdf.umd.min.js` — `integrity="sha384-en/ztfPSRkGfME4KIm05joYXynqzUgbsG5nMrj/xEFAHXkeZfO3yMK8QQ+mP7p1/"`
  - SRI ensures generated diagrams are tamper-resistant against CDN compromise. Do not modify the hashes; if the version is bumped, the new hash must be computed fresh.
- `id="report-container"` on the outermost `.container` div (this is what gets captured)
- `.toolbar` markup with `.toolbar-actions` (collapsed by default) and `.toolbar-toggle` (the `⋯` button)
- `.toolbar` CSS + `@media print { .toolbar { display: none !important; } }`
- `copyAsImage()`, `downloadPNG()`, and `downloadPDF()` script before `</body>`, all using `getBoundingClientRect()` + `html2canvas(document.body, { x, y, width, height, ignoreElements })` to capture a precise rect with breathing room and no toolbar

Caveats: clipboard API needs a user gesture and a secure context (https/file/localhost). SVG `<foreignObject>` renders inconsistently in html2canvas — stick to plain `<svg>` shapes and `<text>`. Bump `scale: 2` to `3` or `4` for higher-res output.

## Output

Always produce a single self-contained `.html` file with:
- Embedded CSS (no external stylesheets except Google Fonts)
- Inline SVG shapes and inline Archicons SVG markup only. Do not use external SVG `<image href="...">` references for logos/icons in exportable PNG/PDF diagrams.
- No JavaScript required for diagram animations; the export toolbar JavaScript is permitted.

The file should render correctly when opened directly in any modern browser. The export toolbar uses two CDN scripts (html2canvas and jsPDF) — no other JavaScript dependencies.
