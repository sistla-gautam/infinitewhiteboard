## Destination

A product spec for a single-org, authenticated, canvas-first application where internal teams create and collaborate live on infinite canvases, with AI assistance for generating and editing canvas content.

## Notes

- Use the glossary in `CONTEXT.md`.
- The product has one user-facing primary object: the canvas.
- The app is authenticated; access is restricted to users whose email matches the approved organization domain.
- Canvas sharing is live-edit only for now; there is no separate view-only mode.
- The AI chat is scoped to the current canvas.
- The only allowed canvas primitives in v1 are text, shapes, and connectors.
- Slides are not a separate object; slide-like content is represented as a rectangle with content on the canvas.

## Decisions so far

- Canvas creation screen (issues/02-canvas-creation-flow.md): a single surface (modal/dropdown) presents Blank / Template / AI-generated together, each a one-click or inline-input target with no intermediate mode-selection step. Naming is never forced — canvases start "Untitled" + timestamp, are always inline-renamable, and get a one-time AI auto-rename (typing-animation style) the moment content exists and no manual name has been set. AI generation happens fully in the background on the creation surface itself (no live-build, no separate transitional screen) with cancel and inline-retry-on-failure, and the user is navigated into the canvas only once generation completes.
- Live collaboration surface (issues/01-live-collaboration-surface.md): always-visible presence cluster + on-demand access-list dropdown, plain colored live cursors, a sticky laser-pointer tool, informational live selection outlines, free concurrent editing (no locks), a single shared AI chat per canvas with the AI itself appearing as a visible presence/cursor actor, silent join/leave, grace-period disconnect handling with a reconnecting indicator, no persisted attribution/history in v1, and a "jump to collaborator" action instead of ambient off-screen indicators.
- AI action boundaries (issues/03-ai-action-boundaries.md): the AI always reasons over the whole canvas, treating any current selection as a contextual reference rather than an exclusive scope. Every AI response is a batch of primitive-level create/update/delete ops (never wholesale region regeneration) executed progressively via the AI's presence/cursor actor, with full read/write authority over any object (no ownership protection). The AI confirms first with a one-line text plan whenever a request is ambiguous or would touch existing objects; purely additive, unambiguous requests execute immediately. Any collaborator (not just the requester) can approve a pending confirmation or interrupt an in-progress batch; interrupting fully rolls back that changeset via the same inverse-op mechanism used for regular multiplayer undo.
- Template provenance (issues/04-template-provenance.md): a template is not a distinct asset type — it's canvas-shaped data (same primitives, fully editable) stored in a centralized template store separate from users' personal canvas collections. Starting from a template copies that stored data into a new canvas owned by the requesting user, the same copy-on-use mechanic as the rest of the creation flow. In v1 only the product owner populates the template store, and it is deployment/seed-managed with no in-app authoring or admin UI. The store is deliberately shaped so a future "save my canvas as a template" feature is just the same copy mechanism run in the opposite direction (canvas → store) rather than a new asset type or schema change; templates carry no version link back to a source canvas, no extra metadata, and no owner attribution requirement in v1.
- Post-login navigation (issues/05-post-login-navigation.md): login lands on a flat "My Canvases" list (owned + shared-with-you unified, inline owner indicator, no tabs), sorted by most-recently-opened-by-you, with an always-visible title-only search box. A Cmd/Ctrl-K command palette is available on the homepage and inside a canvas with context-sensitive commands (home: new/template/AI-generate/search; canvas: insert primitives, open AI chat, go home), sharing its search logic with the visible search box rather than duplicating it. Canvases are returned to via both a persistent header home affordance and a palette command. A dedicated empty state greets zero-canvas users. No pinning, folders, or tags in v1 — the list stays deliberately flat.

## Not yet specified
(none)

## Out of scope

- Multi-organization support.
- View-only sharing mode.
- Global AI chat outside a canvas.
- Image generation or image placement by the AI in v1.

