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

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

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
