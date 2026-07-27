Type: grilling
Status: resolved

## Question

How should the canvas-scoped AI chat be allowed to act on the canvas, and what level of granularity should its requested changes support?

## Answer

1. **Scope**: the AI always reasons over the whole canvas. If the user (or the AI's own presence) has something selected, it's treated as a contextual reference/weighting for the request, not an exclusive restriction to that selection.
2. **Granularity**: every AI response resolves to a batch of primitive-level operations — create/update/delete on individual text, shape, or connector objects. There is no wholesale region regeneration; even a large, vague request ("shard this system") decomposes into a (possibly large) batch of primitive ops, never a full replace of the target area.
3. **Execution model**: batches execute progressively, object-by-object, using the AI's existing visible cursor/presence actor (same mechanism as issues/01) — collaborators watch it build the same way they'd watch a human teammate draw it, rather than seeing the result pop in all at once.
4. **Authority**: the AI has full read/write over any object on the canvas, including ones it didn't create — no special ownership protection for AI-authored vs. human-authored objects, consistent with the no-locks/free-concurrent-editing model already decided for humans.
5. **Confirm-first gate**: before executing, the AI pauses and posts a short text plan in the shared chat whenever the request is ambiguous (multiple reasonable interpretations) **or** the resulting batch would update/delete existing objects. Purely additive requests that are unambiguous execute immediately with no confirmation step.
6. **Confirmation format**: the confirmation is a one-line text plan, not a spatial mockup or a separate "pending" object state on the canvas. Once approved, the progressive execution itself (item 3) is the visual preview — the user watches it land for real and can still act on it per item 7.
7. **Interrupt + rollback**: any collaborator on the canvas (not only the requester) can interrupt an in-progress AI batch at any point. Interrupting triggers a full rollback of that changeset back to last known-good state — implemented by tagging every op in one AI turn with a shared changeset id and running the same inverse-op undo mechanism the canvas already needs for regular multiplayer undo, just auto-triggered instead of via a manual undo keystroke.
8. **Approval access**: any collaborator can approve or reject a pending confirmation, not only the person who sent the triggering message — same "no per-action ownership" principle as item 4 and 7.
