Status: ready-for-agent

## Problem Statement

Internal teams need a single secure application where they can collaborate live on an infinite canvas to produce presentation-like layouts and architecture diagrams with AI help. The product should not force users into a separate slide system, workspace hierarchy, or multi-product mental model. It needs to stay simple at the surface, but still support real-time co-editing, AI-assisted generation and editing, and organization-scoped access.

## Solution

Build an authenticated, canvas-first application with one primary user-facing object: the canvas.

Users sign in with an organization email address and are auto-provisioned when their email matches the allowed organization domain. Canvases are private by default and can be shared with other people in the organization. Shared people can edit the canvas; there is no view-only mode in v1.

The canvas is the product. There is no user-facing workspace concept. After login, users land directly on a flat "My Canvases" list — owned and shared-with-you canvases together, sorted by your own most-recent-open, with title search and a command palette for quick navigation and actions.

Users can start a canvas in one of three ways, all presented together on a single creation surface:

- create a new blank canvas
- start from a template
- generate content with AI

Templates are canvas-shaped data (same primitives, fully editable after instantiation) held in a centralized, system-curated store; starting from one copies that data into a new canvas you own. AI-generated canvases are produced from a prompt entered directly on the creation surface, generating fully in the background before you're navigated in.

The canvas supports live multiplayer collaboration from day one: presence, live cursors, a laser pointer, and free concurrent editing with no locks. The AI assistant is a first-class collaborator on this same footing — it reasons over the whole canvas, applies changes as a batch of primitive create/update/delete operations executed progressively through its own visible cursor/presence actor, confirms first when a request is ambiguous or would touch existing objects, and can be interrupted (with full rollback) by any collaborator. It only works with the allowed canvas primitives in v1: text, shapes, and connectors. It cannot generate or place images in v1.

## User Stories

1. As a user in the organization, I want to sign in with my company email, so that I can access the application without extra account setup.
2. As a first-time user with an approved company email, I want to be admitted automatically, so that I do not need a manual invite to start using the app.
3. As a user without an approved company email, I want to be blocked from access, so that the app stays restricted to the organization.
4. As a user, I want the application to open directly into my canvas experience after login, so that I do not have to learn workspace navigation.
5. As a user, I want to see only canvases that I have access to, so that the app stays focused and private by default.
6. As a user, I want to create a new blank canvas, so that I can start from scratch when I already know what I need.
7. As a user, I want to start from a template, so that I can reuse a prebuilt layout instead of building everything manually.
8. As a user, I want to generate a canvas from AI, so that I can move from a rough idea to a first draft quickly.
9. As a user, I want to switch a canvas from blank to template-like structure or AI-generated structure, so that I can change direction as my work evolves.
10. As a user, I want to clear the contents of a canvas and rebuild it, so that I can restart without creating a new canvas.
11. As a user, I want to collaborate live with other people on the same canvas, so that we can work together in real time.
12. As a user, I want my collaborators' edits to appear as they happen, so that I can follow the conversation and the work without waiting for refreshes.
13. As a user, I want the canvas to remain consistent while multiple people edit it, so that shared work does not corrupt itself.
14. As a user, I want to share a private canvas with another person in the organization, so that we can work on it together.
15. As a user who has been shared a canvas, I want to edit it immediately, so that shared access is useful rather than read-only.
16. As a user, I want sharing to happen inside the authenticated app, so that access is controlled by organization membership.
17. As a user, I want to open a canvas from its URL, so that I can return to work quickly and pass the link to collaborators.
18. As a user, I want the AI assistant to answer questions about the current canvas, so that I can understand and inspect the work in context.
19. As a user, I want to ask the AI assistant to make changes to the current canvas, so that I can delegate repetitive edits.
20. As a user, I want the AI assistant to work only on the current canvas, so that it does not interfere with unrelated work.
21. As a user, I want the AI assistant to use only the allowed primitives, so that generated content stays consistent with the product model.
22. As a user, I want to create text, shapes, and connectors on the canvas, so that I can build both presentation-style layouts and architecture diagrams.
23. As a user, I want all slide-like content to be made from normal canvas objects, so that I can keep editing it like the rest of the board.
24. As a user, I want templates to be editable after creation, so that I can adapt them to the current task.
25. As a user, I want to use the same primitives in templates and AI-generated content, so that the output feels native to the canvas.
26. As a user, I want to work without a separate slide editor, so that I stay in one continuous canvas experience.
27. As a user, I want the application to feel appropriate for office collaboration, so that it supports live, shared working sessions.
28. As a user, I want the application to remain simple at the product level, so that I only need to understand canvases, not extra workspace abstractions.
29. As a user, I want AI-assisted generation to help me get started faster, so that I can spend my time refining rather than laying out every element manually.
30. As a user, I want the product to support future expansion of permissions and ownership later, so that the model can grow if the organization needs stricter controls.
31. As a user, I want to see who's currently active on a canvas, so that I know who I'm collaborating with in real time.
32. As a user, I want to see everyone who has access to a canvas (not just who's currently active), so that I understand who else can reach it.
33. As a user, I want to see collaborators' live cursors with their names, so that I can follow what they're doing.
34. As a user, I want a laser pointer tool, so that I can gesture at something on the canvas without leaving a permanent mark.
35. As a user, I want to see what element a collaborator has selected, so that I know what they're currently focused on.
36. As a user, I want to edit any element freely without waiting on locks, so that collaboration stays fluid.
37. As a user, I want the AI to participate in a single shared chat visible to all collaborators, so that everyone sees the same conversation and result.
38. As a user, I want the AI's actions to appear as a visible actor on the canvas with its own cursor and presence, so I can watch what it's doing like a teammate.
39. As a user, I want collaborators to join or leave without disruptive notifications, so that the experience stays calm.
40. As a user, I want a brief grace period before a disconnected collaborator disappears, so that flickering presence doesn't distract me.
41. As a user, I want to see a "reconnecting" indicator when my own connection drops, so that I know my edits might not be syncing.
42. As a user, I want to jump directly to a collaborator's (or the AI's) current view, so that I can quickly see what they're looking at even if it's off-screen.
43. As a user, I want the canvas creation surface to present blank/template/AI options together in one place, so that I can choose a starting point without extra navigation.
44. As a user, I want a new blank canvas to open immediately without naming it, so that I can start working right away.
45. As a user, I want to preview template cards with a thumbnail and description before choosing one, so that I know what I'm about to start from.
46. As a user, I want to type an AI prompt directly on the creation surface, so that I can generate a canvas without an extra mode-selection step.
47. As a user, I want to cancel AI generation while it's running, so that I'm not stuck waiting on something I no longer want.
48. As a user, I want to retry AI generation inline after a failure, so that I don't have to restart the whole creation flow.
49. As a user, I want my canvas to get a one-time AI-suggested name based on its content, so that I don't have to name it manually.
50. As a user, I want to rename a canvas at any time, so that the name stays meaningful to me.
51. As a user, I want the AI to reason over my entire canvas rather than only my current selection, so that it doesn't miss relevant context.
52. As a user, I want the AI to treat my selection as a hint rather than a strict boundary, so that it can still make holistically sensible edits.
53. As a user, I want the AI's changes to apply as individual primitive operations, so that partial results still make sense if I stop it partway.
54. As a user, I want to see the AI's edits build progressively on the canvas, so that I can follow along like watching a collaborator work.
55. As a user, I want the AI to ask for confirmation before an ambiguous or destructive change, so that I don't lose work unexpectedly.
56. As a user, I want any collaborator, not just the requester, to be able to approve or interrupt an AI action, so that the canvas stays a shared responsibility.
57. As a user, I want to interrupt an in-progress AI batch and have it fully roll back, so that a bad request doesn't leave the canvas half-changed.
58. As a user, I want to start a canvas from a curated set of templates, so that I don't have to build common layouts from scratch.
59. As a user, I want templates to behave exactly like any other canvas once created, so that I can edit them without special constraints.
60. As a user, I want the product owner to be able to add new templates over time, so that the template set can grow and improve.
61. As a user, I want to land on a list of my canvases immediately after logging in, so that I get straight to my work.
62. As a user, I want owned and shared-with-me canvases to appear together in one list, so that I don't have to check multiple places.
63. As a user, I want to tell at a glance which canvases I own versus which were shared with me, so that I understand my relationship to each one.
64. As a user, I want my canvases sorted by when I last opened them, so that my recent work is easy to find.
65. As a user, I want to search my canvases by title, so that I can quickly find one without scrolling.
66. As a user, I want a command palette (Cmd/Ctrl-K) available from the homepage and inside a canvas, so that I can navigate and act quickly from the keyboard.
67. As a user, I want the command palette's search to match the visible search box, so that I get consistent results either way.
68. As a user, I want a persistent way to get back to my canvas list from inside any canvas, so that I'm never stuck without a way home.
69. As a new user with no canvases yet, I want a clear empty state with the same creation options, so that I know how to get started.

## Implementation Decisions

**Access and identity**

- The application will be organization-restricted at the authentication boundary using approved company email addresses.
- First-time users with an approved email will be auto-provisioned.
- Canvases are private by default and can be shared with other users in the organization; shared users can edit, with no view-only sharing mode in v1.

**Product model**

- The canvas is the only primary user-facing container in the product; there is no user-facing workspace concept in v1.
- The only allowed canvas primitives in v1 are text, shapes, and connectors. Any slide-like content is represented as a normal rectangle with content on the canvas, not as a separate slide object.

**Canvas creation flow**

- A single unified surface (modal or dropdown) presents Blank / Template / AI-generated together as one-click or inline-input targets, with no intermediate mode-selection step.
- Blank: a plain click target that creates and opens the canvas immediately.
- Template: a visual grid of cards (title, description, thumbnail preview); clicking instantiates and opens immediately, no separate preview/confirm step.
- AI-generated: a card with an inline prompt input built in from the start.
- Naming is never forced. Canvases start "Untitled" + timestamp, are always inline-renamable, and get a one-time AI auto-rename (typing-animation style) the moment content exists and no manual name has been set; it never re-fires once a name has been set by the user or by this one-time pass.
- AI generation happens fully in the background on the creation surface itself — no live-build, no separate transitional screen — with a "Cancel" action that aborts without creating a canvas, and an inline "try again" action on failure (no silent fallback to blank). The user is navigated into the canvas only once generation completes.

**Live collaboration surface**

- Always-visible presence cluster in the header (who's currently active) plus an on-demand access-list dropdown (everyone with access, not just who's live).
- Plain colored live cursors with name labels, no click-trail.
- A dedicated, sticky (not hold-to-activate) laser-pointer toolbar tool.
- Live selection outlines (matching cursor color) are purely informational — free concurrent editing, no soft locks.
- A single shared AI chat per canvas, visible to all live collaborators.
- The AI is a visible presence/cursor/selection actor while applying changes, using the same mechanism as human collaborators, including "jump to" targeting.
- Join/leave is silent (no toast/sound). Disconnection has a grace period before a user's presence disappears for others, and the disconnected user sees a "reconnecting…" indicator.
- No persisted attribution/history in v1 (no "last edited by," no version history).
- "Jump to collaborator" (or to the AI actor) pans the viewport to their current location, instead of an ambient off-screen indicator.

**AI action boundaries**

- The AI always reasons over the whole canvas; any current selection is a contextual reference/weighting, not an exclusive scope.
- Every AI response resolves to a batch of primitive-level create/update/delete operations — never wholesale region regeneration, even for large vague requests.
- Batches execute progressively, object-by-object, via the AI's own visible cursor/presence actor.
- The AI has full read/write authority over any object regardless of who created it — no ownership protection, consistent with the no-locks human editing model.
- Before executing, the AI posts a one-line text plan and pauses whenever the request is ambiguous or would update/delete existing objects; unambiguous, purely additive requests execute immediately with no confirmation step.
- Any collaborator (not only the requester) can approve a pending confirmation, or interrupt an in-progress batch at any point.
- Interrupting triggers a full rollback of that changeset, implemented by tagging every op in one AI turn with a shared changeset id and running the same inverse-op undo mechanism used for regular multiplayer undo.

**Templates and provenance**

- A template is not a distinct asset type: it is canvas-shaped data (same primitives, fully editable after instantiation), held in a centralized template store separate from users' personal canvas collections.
- Starting from a template copies the stored data into a new canvas owned by the requesting user — the same copy-on-use mechanic as the rest of the creation flow.
- In v1, only the product owner populates the template store; it is deployment/seed-managed, with no in-app authoring or admin UI.
- The store is shaped so that a future "save my canvas as a template" feature would be the same copy mechanism run in reverse (canvas → store), not a new asset type or schema change. No version link back to a source canvas, no extra metadata, and no owner-attribution requirement in v1.

**Post-login navigation**

- Login always lands on a "My Canvases" list — the app's home, not a specific canvas.
- Owned and shared-with-you canvases appear in one unified list (no tabs/filters), with a small inline owner indicator on canvases you don't own.
- Default sort is most-recently-opened-by-you (not global recency, not alphabetical).
- An always-visible search box matches canvas title only — no content indexing in v1.
- A Cmd/Ctrl-K command palette is available on the homepage and inside a canvas, with context-sensitive commands (home: new/template/AI-generate/search; canvas: insert primitives, open AI chat, go home), sharing its search logic with the visible search box rather than duplicating it.
- Returning to the list from a canvas is available via both a persistent header home affordance and an equivalent palette command.
- A dedicated empty state (explanatory copy plus the same three creation entry points) greets zero-canvas users.
- Organization is fully flat in v1: no pinning/favoriting, folders, or tags.

**Seam**

- All canvas behavior (creation, editing, sharing, template instantiation, presence, and AI-issued operations) is driven through a single canvas operations boundary — one API/service layer that fronts every canvas mutation and read. This is the single seam the product is built and tested against, rather than scattering logic or test coverage across UI, transport, and persistence layers independently.

## Testing Decisions

- The seam for this feature is the canvas operations boundary described above: tests issue operations (or sequences of them) through this layer and assert on resulting canvas state and broadcast events, not on UI DOM structure or mocked transport internals.
- Multiplayer/collaboration behavior is tested by driving multiple simulated clients against that same seam and asserting they converge on consistent shared canvas state — there is no separate "collaboration seam."
- Core behaviors to test include: sign-in gating, auto-provisioning, canvas creation modes (blank/template/AI), live collaborative editing convergence, sharing and edit access, template instantiation (and that instantiated canvases are fully independent copies), AI canvas actions (scope, batching, confirm-first gating, interrupt/rollback), and primitive constraints (only text/shapes/connectors ever produced).
- AI tests should prove the assistant stays canvas-scoped, resolves to primitive-level batched ops, follows the confirm-first gate on ambiguous/destructive requests, and fully rolls back on interrupt.
- Sharing tests should prove access is restricted to authenticated organization members and that shared canvases become editable (never view-only) for the people they're shared with.
- Template tests should prove templates instantiate as independent copies (editing the new canvas never affects the store, and vice versa) and remain fully editable afterward.
- Navigation tests should prove the canvas list reflects ownership/sharing correctly, sorts by the viewer's own recency, and that title search and the command palette return consistent results.
- There is no existing test suite in this repository yet, so this spec establishes the first behavioral contract for the product.

## Out of Scope

- Multi-organization support.
- View-only sharing mode.
- Global AI chat outside a canvas.
- Image generation or image placement by the AI in v1.
- Workspace concepts as a product surface.
- A separate slide editor or separate slide object model.
- Public unauthenticated access to canvases.
- Comments, exports, version history, and presentation mode unless added later.
- Persisted edit attribution/history (who last edited what) in v1.
- Canvas organization features: pinning/favoriting, folders, or tags.
- In-app template authoring or admin UI — the template store is deployment/seed-managed only in v1.
- User-generated templates (e.g. "save canvas as template") — deliberately deferred, though the template store is shaped so this can be added later without a redesign.

## Further Notes

This spec collapses the decisions recorded across `.scratch/ai-canvas-collab/map.md` and its five resolved decision tickets (`issues/01` through `issues/05`) into a single buildable plan. All product questions that were open in that map are now resolved; none remain outstanding.

The spec intentionally keeps the model small: one authenticated organization, one primary user object (the canvas), one canvas-first collaboration surface, one canvas-scoped AI assistant, and one canvas-operations seam that every behavior above is built and tested against.
