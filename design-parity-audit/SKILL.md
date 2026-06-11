---
name: design-parity-audit
description: Use this skill when the user provides one or more development URLs and Figma MCP/design links and wants a design parity audit for an entire feature module. The audit compares implemented pages, dialogs, drawers, tables, forms, and states against Figma using both visual screenshot comparison and CSS/Figma parameter comparison, then outputs a visual report with restoration percentages, page-level scores, module-level scores, annotated screenshot differences, and 1280px desktop adaptation findings.
metadata:
  short-description: Audit frontend implementation against Figma designs
---

# Design Parity Audit

This skill audits how closely a development implementation matches Figma designs for a complete feature module, not just a single screen.

Use it for requests like:

- Compare this dev module with my Figma design.
- Do a design walkthrough/audit and report the differences.
- Check CSS vs Figma parameters and visual parity.
- Output restoration percentage and discrepancy report.
- Audit multiple pages in a feature flow, including dialogs, drawers, and states.

## Default Scope

Audit at the feature-module level. A module may include:

- Multiple pages or routes
- Tables, forms, detail pages, dashboards, tabs, and settings pages
- Modals, drawers, popovers, dropdowns, and empty/error/loading states
- Repeated components whose consistency affects the module

If the user gives only one URL or one Figma node, audit that item as a single-screen module.

## Inputs

Collect or infer these inputs:

- Development URL or local preview URL
- Figma design URL, file key, or MCP node IDs
- Module name
- Page/frame mapping between dev pages and Figma frames
- Required states to inspect, if any
- Authentication requirements, if any

When the page/frame mapping is incomplete, first inspect available Figma metadata and the development page structure. Ask the user only when the mapping remains ambiguous or multiple reasonable module scopes exist.

## Viewports

Use these viewport rules unless the user overrides them:

- Primary design parity viewport: `1440 * 900`
- Desktop adaptation viewport: `1280 * 800`

The `1440 * 900` viewport is the scoring baseline for design restoration. The `1280 * 800` viewport is a separate desktop adaptation check and does not affect the main restoration percentage.

Do not add tablet or mobile checks unless the user explicitly asks.

## Required Tooling Behavior

For Figma:

- Use Figma MCP tools to obtain screenshots, design context, metadata, variables, and relevant node information.
- Prefer node-specific Figma URLs. If the user provides only a file URL, inspect pages/frames first and identify likely candidates.
- For large modules, audit frame by frame rather than relying on one giant screenshot.

For the development implementation:

- Open the dev URL in a browser-capable workflow.
- If login/session state is required, use the user's browser profile or authenticated browser tooling when available.
- Capture screenshots at `1440 * 900` and `1280 * 800`.
- Extract computed CSS for key visible elements, including layout boxes, typography, color, spacing, border, radius, shadows, and icon/image sizing.

Do not rely only on visual judgment. Always combine screenshot review with measurable CSS/Figma parameter comparison when both sources are available.

## Audit Workflow

1. Establish real page access:
   - Prefer the user's real Chrome session through the Chrome extension when the development URL is authenticated or internal.
   - Open or claim a Chrome tab for the development URL.
   - Verify that the page is the real business surface before scoring. Look for module-specific text, routes, tabs, tables, and controls.
   - If the URL redirects to login, keep the tab open for user login handoff. After the user logs in, reclaim the same tab and continue.
   - Do not score a login page against a business Figma screen.

2. Build the module map:
   - List each audited page, route, frame, modal, drawer, and state.
   - Pair every development surface with its matching Figma node.
   - Mark any missing or inferred mappings.
   - When the Figma module contains child frames or the development page contains tabs, status filters, detail links, create/edit flows, drawers, or dialogs, click through those subpages/states in the development browser and audit them as separate surfaces when they match a Figma frame.
   - Keep a stable surface map using this shape: `<surface name> -> <dev action/path> -> <Figma node id> -> <screenshot pair> -> <issues>`.

3. Capture Figma baselines:
   - Render or fetch Figma screenshots at the design frame size.
   - Collect design parameters: width, height, x/y position, layout, padding, gap, typography, fill, stroke, radius, shadows, opacity, and component states.
   - Record the design frame size. If it differs from `1440 * 900`, preserve the original size and explain how comparison was normalized.

4. Capture development implementation:
   - Screenshot each page/state at `1440 * 900`.
   - Screenshot each page/state at `1280 * 800`.
   - Collect computed CSS and bounding boxes for major elements.
   - Include hover, active, selected, disabled, empty, loading, error, modal, drawer, dropdown, and tab states when they exist in the provided design or are core to the module.
   - For multi-surface modules, navigate by clicking the real development UI whenever possible, not by guessing URLs only. Examples: table action menus, add buttons, status tabs, detail pages, drawers, modal triggers, and step forms.
   - Capture and compare each clicked surface immediately against its mapped Figma frame before moving to the next surface, so browser state and screenshot state stay synchronized.
   - If exact viewport control is unavailable in a real authenticated Chrome tab, capture the actual viewport, label all scores as estimated, and continue only if the business page itself is correct.

5. Compare visually:
   - Check overall layout, alignment, element presence, visual hierarchy, density, crop, scroll position, first-screen composition, and section proportions.
   - Identify missing, extra, shifted, stretched, compressed, or visually inconsistent elements.
   - Pay special attention to tables, filters, form fields, buttons, tabs, sidebars, topbars, dialogs, drawers, and empty states.

6. Compare CSS/Figma parameters:
   - Compare dimensions, x/y coordinates, padding, margin, gap, font family, font size, font weight, line height, color, background, border, radius, shadow, opacity, and icon/image size.
   - Prioritize visible user-facing differences. Avoid flooding the report with insignificant sub-pixel noise.
   - Treat differences under 2px as low priority unless repeated across the module or visibly harmful.

7. Check `1280 * 800` desktop adaptation:
   - Verify that layout remains stable and intentional.
   - Check whether the core layout still fits a smaller desktop, including horizontal overflow, table usability, filter wrapping, sidebar/topbar alignment, modal/drawer placement, and dropdown positioning.
   - Report this as a health grade, not as part of the main restoration percentage.

8. Calculate scores and write the report.

## Scoring

Module restoration:

```text
Module restoration = average primary page restoration * 80% + module consistency * 20%
```

Single page restoration:

```text
Page restoration = visual restoration * 55% + CSS/Figma parameter restoration * 35% + state coverage * 10%
```

Parameter restoration should consider:

```text
Layout and size: 30%
Spacing: 20%
Typography: 20%
Color: 15%
Border, radius, shadow, opacity: 10%
Images and icons: 5%
```

Desktop adaptation is separate:

```text
Desktop adaptation health: A / B / C / D
```

Suggested grade meanings:

- A: Stable, intentional, no material layout issues.
- B: Minor looseness or spacing issues, still usable and visually coherent.
- C: Noticeable layout degradation, stretching, misalignment, or awkward density.
- D: Broken layout, overlap, severe overflow, or unusable desktop adaptation behavior.

When exact numeric comparison is not possible, label the score as estimated and explain why.

## Severity

Use three severity levels:

- High: Clearly visible issue that affects design fidelity, hierarchy, usability, or core workflow.
- Medium: Noticeable mismatch that should be fixed but does not block usage.
- Low: Minor polish difference, small repeated drift, or low-impact inconsistency.

## Report Format

Prefer a visual HTML report when screenshots are available. Markdown is acceptable only when the user asks for text output or when image generation is not possible.

Match the user's language for the report. If the user asks in Chinese, write the audit report in Chinese.

The default deliverable should combine:

- A simple visual summary with audited page count, module restoration percentage, desktop adaptation health grade, and high-priority issue count.
- A fixed switchable comparison-draft tab bar below the summary, one tab per audited page/state. Always render this tab bar even when there is only one audited surface, so the report framework stays consistent for multi-page modules.
- Each comparison draft should show Figma and development screenshots side by side.
- Mark issues directly on the development screenshot with numbered red boxes.
- Provide the issue explanation next to the comparison image, matching the numbers on the development screenshot.
- Support image zoom controls in the visual report: zoom in, zoom out, fit to screen, 100% view, and mouse-wheel zoom inside the image area. The image area must allow horizontal/vertical scrolling and hand-style drag panning so small screens can inspect details.
- Put Figma MCP parameters and development computed CSS parameters inside each numbered issue explanation, not in a detached parameter table unless the user asks for a separate table.
- Prioritize layout structure, spacing, font size/weight/line-height, and colors.
- Every numbered issue must include a concrete fix recommendation. Prefer actionable values or implementation directions, such as target width/height, gap, padding, row height, font size/weight, color token, max-width behavior, alignment rule, or component/state replacement. Avoid vague recommendations like “optimize style” or “adjust spacing”.
- Explicitly list ignored visual noise: demo data values, avatars/names, watermarks, placeholder illustration/fill effects, browser extension overlays, and Figma-only sample content.
- Do not score ignored visual noise as a design parity issue unless the user specifically asks to audit that content.
- Do not flag width differences caused only by reasonable responsive adaptation. Flag them only when they reveal a structural problem, such as missing containers, wrong max-width behavior, uncontrolled table columns, broken density, or overflow that changes usability.
- Do not flag differences between Figma sample text/data and real development data text/counts. Audit the component structure and styling instead.
- Keep text concise. The annotated comparison image is the primary report surface.

Use this structure:

```markdown
# Design Parity Audit Report

## Module Summary

- Module: <module name>
- Audited surfaces: <count>
- Primary design viewport: 1440 * 900
- Desktop adaptation viewport: 1280 * 800
- Module restoration: <percent>
- Desktop adaptation health: <A/B/C/D>
- Overall conclusion: <short conclusion>

## Page Scores

| Surface | Restoration | Visual | CSS/Figma parameters | State coverage | Desktop adaptation |
|---|---:|---:|---:|---:|---|
| <page/state> | <percent> | <percent> | <percent> | <percent> | <grade> |

## High Priority Differences

| Surface | Area | Difference | Figma | Development | Recommendation |
|---|---|---|---|---|---|
| <page> | <module area> | <issue> | <figma value> | <dev value> | <fix direction> |

## All Differences

| Severity | Surface | Area | Difference | Figma | Development | Recommendation |
|---|---|---|---|---|---|---|
| High/Medium/Low | <page> | <area> | <issue> | <figma value> | <dev value> | <fix direction> |

## Desktop Adaptation

| Size | Surface | Health | Issue | Recommendation |
|---|---|---|---|---|
| 1280 * 800 | <page> | <A/B/C/D> | <issue> | <fix direction> |

## Missing Or Unverified Items

- <Anything that could not be checked, including inaccessible states, missing Figma nodes, login blockers, or unavailable CSS data.>
```

## Reporting Rules

- Lead with the restoration percentage and highest-impact issues.
- Be specific: name the page, component area, Figma value, development value, and recommended correction.
- Separate confirmed differences from inferred differences.
- If a Figma frame or dev state is unavailable, mark it as unverified instead of silently skipping it.
- Do not include tablet or mobile findings unless the user requested them.
- Do not automatically modify code unless the user explicitly asks for fixes after the audit.
