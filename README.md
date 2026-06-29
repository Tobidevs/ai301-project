# Contribution [#]: Render each page in a different element

**Contribution Number:** [1 / 2 / 3]  
**Student:** Tobi Akere
**Issue:** [Github URL](https://github.com/vivliostyle/vivliostyle.js/issues/1235)
**Status:** Phase 1

---

## Why I Chose This Issue
I have solid experience in typescript, and seems like an interesting fix that can be done quickly to get back on track with the program.
---

## Understanding the Issue

### Problem Description

the current implementation is each page is renderer in a single element, and the person would like to be able to see the document each page in the bottom of the other, like in a PDF viewer.

### Expected Behavior

See the document each page in the bottom of the other, like in a PDF viewer.

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. Clone the vivliostyle.js repository: `git clone https://github.com/vivliostyle/vivliostyle.js.git && cd vivliostyle.js`
2. Install dependencies: `yarn install`
3. Build the core package: `yarn build`
4. Open the Vivliostyle Viewer in a browser (run `yarn dev` inside `packages/viewer` and navigate to `http://localhost:9000`)
5. Load any multi-page document (e.g., pass a sample EPUB or HTML document via the viewer URL parameter `#src=...`)
6. Attempt to scroll or view pages stacked top-to-bottom as you would in a PDF viewer
7. **Observed:** only one page is visible at a time inside a single shared container element; navigating to the next page re-renders content into that same element, making it impossible to see multiple pages simultaneously

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/Tobidevs/ai301-project/tree/main](https://github.com/Tobidevs/ai301-project/tree/main)
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Vivliostyle currently reuses a single DOM element to render each page, so only one page is visible at a time. The goal is a "continuous scroll" mode where every page gets its own element, stacked vertically so users can scroll through the whole document like a PDF viewer.

**Match:** The existing spread view (two pages side-by-side) already demonstrates that the viewer can coordinate multiple page elements; that layout logic is the closest existing pattern to build from.

**Plan:**
1. Locate the core page-rendering entry point in `packages/core/src` (likely `vivliostyle/page.ts` or the layout manager) and trace how the single container element is created and reused per page
2. Add a new rendering mode flag (e.g., `continuousScroll`) to the viewer's public API / options object
3. When the flag is set, create a fresh `<div>` wrapper for each page instead of recycling the existing one, and append it below the previous page's element
4. Apply CSS to the wrapper container so pages stack vertically with consistent margins (similar to a PDF viewer)
5. Update scroll/navigation logic so "next page" scrolls into view rather than replacing content
6. Add a toggle in `packages/viewer` UI to expose the new mode to end users
7. Write or update unit tests covering the new rendering path and verify existing single-page mode is unaffected

**Implement:** [Link to your branch/commits as you work]

**Review:** Confirm changes follow the project's TypeScript style, pass `yarn test`, and do not regress existing navigation or spread-view behavior.

**Evaluate:** Load a multi-page EPUB in the viewer with the new mode enabled and confirm all pages render simultaneously and are scrollable top-to-bottom.

---

## Testing Strategy

### Validation Performed So Far

- **Type checking:** Ran `tsc --noEmit` on `packages/core` — passes with no
  errors after the changes.
- **Build:** Ran `yarn build` for `@vivliostyle/core` (tsc declarations +
  esbuild bundle) — builds cleanly.
- **Formatting/lint:** Ran Prettier on the changed files; the pre-commit
  hook (lint-staged) passed on commit.

### Planned / Remaining

- [ ] Manual browser test: load a multi-page EPUB in the viewer with
  `pageViewMode: "continuousScroll"` and confirm all pages render stacked
  top-to-bottom and scroll smoothly.
- [ ] Regression check: confirm `singlePage`, `spread`, and `autoSpread`
  modes are unaffected, including switching to/from continuous scroll.
- [ ] Verify zoom / fit-to-screen behavior fits page width in the new mode.
- [ ] Add unit/integration coverage for the new rendering path.

---

## Implementation Notes

### Progress Summary

Implemented a new `continuousScroll` page view mode in the Vivliostyle core
engine. While tracing the rendering path I found that, with the existing
`renderAllPages` option, every page is *already* rendered into its own
container element inside the viewport's spread container — but all pages
except the current one (or current spread) are hidden with `display: none`.
So the fix did not require a new per-page element pipeline; it required a
mode that reveals all of those existing page elements and stacks them
vertically.

The new mode:
- Adds `CONTINUOUS_SCROLL = "continuousScroll"` to the `PageViewMode` enum
  and exposes it through the public `pageViewMode` option.
- Reveals every rendered page (instead of hiding non-current ones) and lays
  them out in a single vertical column.
- Scrolls the current page into view on navigation instead of swapping the
  content of a shared element.
- Sizes the zoom box to fit page width and the summed page heights so the
  whole document scrolls like a PDF viewer.
- Forces `renderAllPages` on, since all pages must exist to be stacked.

**Decisions:** Reused the existing page-container elements and the
`data-vivliostyle-*` attribute pattern (mirroring `spread-view`) rather than
introducing a new DOM structure, keeping the change small and consistent
with the codebase. Interaction (hyperlink) listeners are currently attached
to the current page only, matching existing behavior; extending them to all
visible pages is a noted follow-up. Exposing the mode through the viewer UI
(settings panel toggle + i18n) is also a follow-up — the mode is currently
reachable via the core API option.

### Code Changes

- **Original fix branch:** [`feat/continuous-scroll-page-view-mode`](https://github.com/Tobidevs/vivliostyle.js/tree/feat/continuous-scroll-page-view-mode)
  (fork: `Tobidevs/vivliostyle.js`)
- **Original key commit:** [`0955643`](https://github.com/Tobidevs/vivliostyle.js/commit/0955643c7b795fbc149d0e17806b1276f528b812) —
  *feat(core): add continuousScroll page view mode*
- **PR branch:** [`feature/continuous-page-scroll-view-mode`](https://github.com/Tobidevs/vivliostyle.js/tree/feature/continuous-page-scroll-view-mode)
  (clean cherry-pick onto current upstream `master`)
- **PR commit:** [`22f428f`](https://github.com/Tobidevs/vivliostyle.js/commit/22f428f1) —
  *feat(core): add continuousScroll page view mode*
- **Pull request:** [vivliostyle/vivliostyle.js#2028](https://github.com/vivliostyle/vivliostyle.js/pull/2028)
- **Files modified:**
  - `packages/core/src/vivliostyle/adaptive-viewer.ts` — enum value,
    `data-vivliostyle-continuous-scroll` attribute, `showAllPages`,
    `hideAllPages`, `scrollToPage`, `setContinuousScrollZoom`, integration
    into `showCurrent`, and forcing `renderAllPages`.
  - `packages/core/src/vivliostyle/epub.ts` — `OPFView.forAllPages()` helper
    to iterate every rendered page in spine order.
  - `packages/core/src/vivliostyle/assets.ts` — CSS to stack page containers
    vertically when the mode is active.
  - `packages/core/src/vivliostyle/core-viewer.ts` — documented the new
    `pageViewMode` option.

---

## Pull Request

**PR Link:** [https://github.com/vivliostyle/vivliostyle.js/pull/2028](https://github.com/vivliostyle/vivliostyle.js/pull/2028)

**PR Description:** Adds a new `continuousScroll` value to `PageViewMode` so
multi-page documents can be viewed in a PDF-like vertical scroll mode. The PR
reuses Vivliostyle's existing rendered page containers, reveals all rendered
pages in document order, scrolls the active page into view on navigation, adds
the viewport styling for stacked pages, and documents the new core option.

**What I contributed:**
- Located the existing fix commit
  [`0955643`](https://github.com/Tobidevs/vivliostyle.js/commit/0955643c7b795fbc149d0e17806b1276f528b812)
  on the fork branch
  [`feat/continuous-scroll-page-view-mode`](https://github.com/Tobidevs/vivliostyle.js/tree/feat/continuous-scroll-page-view-mode).
- Found that the original branch produced a noisy diff against upstream
  `master` because it was based on older history.
- Created a clean branch,
  [`feature/continuous-page-scroll-view-mode`](https://github.com/Tobidevs/vivliostyle.js/tree/feature/continuous-page-scroll-view-mode),
  from current upstream `master` and cherry-picked only the continuous-scroll
  fix as commit
  [`22f428f`](https://github.com/Tobidevs/vivliostyle.js/commit/22f428f1).
- Verified the cleaned branch with `yarn workspace @vivliostyle/core build`
  and `yarn workspace @vivliostyle/core test` (600/600 Chrome Headless tests
  passed).

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** PR opened and awaiting maintainer review. Manual browser validation
of the viewer UI remains a follow-up because this contribution currently
exposes the mode through the core API option rather than a viewer settings UI.

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
