Type: grilling
Status: resolved

## Question

With no workspace concept, what should the navigation model be after login, beyond giving the user access to their own canvases (e.g. how canvases are listed, organized, or found)?

## Answer

1. **Landing surface**: login always lands on a "My Canvases" list screen — this is the app's home, not a specific canvas.
2. **Ownership display**: a single unified list, not split into "My canvases" / "Shared with me" sections. Owned and shared-with-you canvases appear together, with a small inline owner indicator on canvases you don't own. No separate tabs or filters for this.
3. **Default sort**: most recently opened by you. Not global recency (others' edits don't reorder the list) and not alphabetical.
4. **Search**: an always-visible search box on the list screen, matching canvas title only — no canvas-content indexing in v1.
5. **Command palette**: a Cmd/Ctrl-K palette is available both on the homepage and inside a canvas, with context-sensitive commands.
   - Homepage: new blank canvas, create from template, generate with AI, and search-by-title (shares its underlying search implementation/UI with the visible search box rather than reimplementing it separately).
   - Inside a canvas: insert primitives (rectangle, ellipse, text), open the AI chat, plus navigation commands like "go home."
6. **Returning to the list from a canvas**: both a persistent home affordance in the canvas header (logo/back-arrow, always visible) and an equivalent palette command — not palette-only, since leaving a canvas needs to be discoverable without knowing a shortcut.
7. **Empty state**: a brand-new user with zero canvases sees a dedicated empty state (explanatory copy) alongside the same three creation entry points, not a silently blank list.
8. **Organization**: fully flat in v1 — no pinning/favoriting, folders, or tags. Recency sort plus title search is the entire findability model, consistent with the product's deliberate avoidance of workspace-like hierarchy.
