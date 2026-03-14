**Date:** 2026-03-14
**Type:** finding
**Status:** active
**Superseded-by:** n/a
**Scope:** global
**Overrides-convention:** no
**Related-to:** n/a

## Summary

Comprehensive challenge of a proposed modernization plan to remove Materialize CSS, replace the filter modal with a sidebar, add visual polish, switch fonts, and adopt aspect-ratio card layout. Ten concerns raised across framework removal scope, accessibility, performance, and sequencing.

## Context

Proposal includes six simultaneous changes: (1) drop Materialize CSS entirely, (2) font swap Roboto to Inter, (3) CSS Grid auto-fill with aspect-ratio cards, (4) filter sidebar instead of modal, (5) card visual refresh with protocol-colored glow, (6) layout/interaction polish. The app is a single-file architecture (inline JS in index.html, one style.css) with ~180 cards and heavy jQuery usage.

## Concerns Raised

### Tooltip replacement strategy is undefined -- Unresolved
Materialize tooltips are the most pervasive framework dependency (~540 instances per render across card sections). The proposal does not address how these will be replaced. CSS title attributes, a custom system, or removal are very different in scope.

### Sidebar modal vs non-modal ambiguity -- Unresolved
Native `<dialog>` provides free accessibility (Escape, focus trap, backdrop) but blocks page interaction, defeating sidebar UX. A non-modal sidebar requires manual accessibility implementation. The answer determines ~40% of implementation effort.

### Aspect-ratio may worsen existing content overflow -- Unresolved
Cards are fixed at 580px height with overflow:hidden. Aspect-ratio makes height dependent on width, shrinking text space at narrow viewports. Unknown whether current card text already overflows at 580px.

### No search debounce + per-card animations = jank -- Unresolved
Search input fires full DOM rebuild on every keystroke (no debounce). Adding fadeInUp animations to 180 cards per rebuild will cause visible performance issues on mid-range devices.

### Collapsible CSS-only replacement is harder than expected -- Unresolved
`<details>/<summary>` does not animate height transitions smoothly. If animated expand/collapse matters, pure CSS requires interpolate-size or explicit height workarounds.

### jQuery dependency not addressed -- Unresolved
Entire app is jQuery. Removing Materialize while keeping jQuery is valid but should be an explicit decision. The inline JS (lines 133-765) is wall-to-wall jQuery calls.

### Protocol hover glow is already easy -- Addressed
Existing --shadow-color CSS custom properties on each protocol class (lines 52-81 of style.css) make this trivial with no JS needed: `box-shadow: 0 0 Npx var(--shadow-color)` on hover. This concern from the risk list is a non-issue.

### formSelect() appears to be dead code -- Unresolved
Line 587 calls `$('select').formSelect()` but no `<select>` elements exist in the current HTML. Likely vestige of previous filter UI. Should be cleaned up during modernization.

### Scope is too large for a single PR -- Unresolved
Six simultaneous changes make regression isolation impossible. Materialize removal alone is a meaningful, testable unit of work that would eliminate 130 !important overrides.

### Collapsible re-initialization may already be broken -- Unresolved
`.collapsible()` is only called once on load (line 184) but displayData() rebuilds the collapsible DOM entirely on each filter. New collapsible elements may not have event listeners. This is a potential latent bug in the current code that should be understood before replacement.

## Implications

- Tooltip strategy must be decided before implementation begins; it is the largest hidden cost
- Sidebar vs modal decision cascades into accessibility, layout, and animation approach
- Sequencing as multiple PRs (Materialize removal first, then sidebar, then visual polish) significantly reduces risk
- The existing --shadow-color infrastructure makes protocol-colored glow straightforward
- A search debounce should be added regardless of whether animations are included
