Type: grilling
Status: resolved

## Question

What live collaboration signals and behaviors belong in v1 beyond simultaneous editing on the same canvas?

## Answer

1. **Presence (always visible)**: a live avatar/icon cluster in the header showing users currently active on the canvas.
2. **Access list (on-demand)**: a separate dropdown/action reveals everyone who has access to the canvas (not just who's live) — hidden until the user requests it.
3. **Live cursors**: plain colored cursor + name label per active user, no click-trail.
4. **Laser pointer**: a dedicated toolbar tool (not hold-to-activate), sticky until deselected; keyboard shortcuts for tools (laser included) are a general future enhancement, not v1-specific.
5. **Live selection outlines**: colored outline (matching cursor color) shown on whatever element a user has selected/is editing — purely informational.
6. **Concurrency model**: free concurrent editing — no soft locks; outlines never block others from editing the same element.
7. **AI chat**: single shared conversation per canvas, visible to all live collaborators on that canvas (any future global/outside-canvas AI would stay per-user, but that's already out of scope).
8. **AI as visible actor**: while applying changes, the AI gets its own presence icon + distinct cursor/outline identity, visible to everyone — same mechanism as human presence/cursor/selection.
9. **Join/leave**: silent — presence icon simply appears/disappears, no toast/sound.
10. **Disconnection handling**: grace period before a disconnected user's presence/cursor disappears for others (avoids flicker on brief drops); the disconnected user sees a "reconnecting…" indicator themselves — text banner for now, with opacity-reduction/loading-symbol-on-cursor flagged as an alternative treatment to explore later.
11. **Attribution/history**: none in v1 — no persisted "last edited by," no version history. Git-based periodic checkpointing noted as a candidate mechanism if version history is built later, not a v1 commitment.
12. **Off-screen collaborators**: no ambient edge indicator, but a "jump to" action (from a presence icon) pans your viewport to that collaborator's current location — this applies to the AI actor too, since it shares the same presence mechanism.
