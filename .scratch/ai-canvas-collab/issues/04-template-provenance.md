Type: grilling
Status: resolved

## Question

Should templates remain system-provided only in v1, or should the model allow templates to later become user-generated assets (e.g. saving a canvas as a reusable template)?

## Answer

1. **Not a distinct asset type**: a template is canvas-shaped data — same primitives, fully editable after instantiation — not a separate schema or object type.
2. **Storage**: templates live in a centralized template store, separate from users' personal canvas collections, rather than as a flag on a user's own canvas.
3. **Instantiation**: starting from a template copies the stored data into a new canvas owned by the requesting user — the same copy-on-use mechanic as the rest of the creation flow.
4. **v1 authorship**: only the product owner populates the template store. It's deployment/seed-managed — no in-app authoring or admin UI.
5. **Future-proofing**: the store is deliberately shaped so a future "save my canvas as a template" feature is just the same copy mechanism run in reverse (canvas → store), not a new asset type or schema change.
6. **Scope**: no version link back to a source canvas, no extra metadata, and no owner attribution requirement in v1.
