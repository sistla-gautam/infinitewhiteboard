# Context

## Glossary

- canvas: the only primary user-facing object in the product. A canvas is private by default and can be shared by email with anyone — whether or not they have an account yet (see `pending share`) — and this works identically in every deployment mode. Shared users can edit it; there is no view-only tier.
- owner: the person who created a canvas. The only person who can revoke another person's access to that canvas (pending or already-active). Ownership carries no special editing authority — any collaborator can edit any object, consistent with the no-locks model — it is solely the gate on who can be removed from the canvas. Ownership transfer is not yet supported.
- pending share: an email-based invite to a canvas that has not yet resolved into access. Created the moment a canvas is shared with an email, regardless of whether that email already has an account on the deployment. Resolves into real access the moment the invited email, once authenticated, loads that canvas's URL. If never resolved within `CANVAS_SHARE_INVITE_EXPIRY_HOURS`, it is discarded outright rather than marked expired — the owner's only path forward is to reshare.
  _Avoid_: invite, invitation (ambiguous with Clerk's own invitation primitives, which this project does not use for canvas sharing)
- template: a prebuilt arrangement of the allowed canvas primitives. Templates are fully editable after creation.
- AI assistant: the canvas-scoped chat agent. It can answer questions about the current canvas and apply requested changes using only the allowed primitives.
- deployment mode: whether a given deployment is `org-gated` or `public`. Chosen once per deployment at build/deploy time, never mixed within one running instance. The two modes differ only in sign-up/access gating (see `organization`) — every other product concept (canvas, template, AI assistant, sharing) behaves identically in both.
- organization: the authentication and authorization boundary in `org-gated` deployments only. Users are admitted by matching a single approved email domain and are auto-provisioned on first sign-in. Not a concept in `public` deployments — a public deployment has no organization boundary at all, and any authenticated user can sign up.
- workspace: not a user-facing product concept in this product.
- slide: not a separate product object. Any slide-like area is just a rectangle with content on the canvas.

## Allowed canvas primitives

- text
- shapes
- connectors

