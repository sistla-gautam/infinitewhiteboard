# Pending canvas shares resolve on canvas load, not on a Clerk webhook

## Context

Both `docs/research/clerk-auth-modes.md` and `docs/research/2026-08-08-clerk-auth-modes.md` — and ADR 0001, which cites them — assumed pending canvas shares (see `pending share` in `CONTEXT.md`) resolve via Clerk's `user.created` webhook: store a `pending_share` row keyed by invited email, then materialize access when that email's account gets created, with lazy re-resolution on sign-in as a fallback for missed webhook deliveries.

Grilling issue #21 surfaced a gap in that design: sharing deliberately does not check whether the invited email already has a deployment account (checking would add branching cost for the common case, and the resolved sharing flow is meant to be uniform). But `user.created` only fires once, at account creation — it will never fire again for an email that was already registered before the share happened. A webhook-only resolver would leave every share-to-an-existing-user permanently unresolved.

## Decision

Pending shares resolve when an authenticated user loads the specific canvas they were shared on: the app checks for a `pending_share` on that canvas matching the requester's verified email and, if found, resolves it into real access at that point. The invite email sent to the invitee links directly to the canvas (not to a generic sign-up page), so following it is what drives resolution. There is no Clerk webhook involved in this path at all.

This works uniformly for brand-new and already-registered invitees, since resolution depends only on "an authenticated email loaded this canvas," not on when the account was created.

## Consequences

- An invitee who never visits that specific canvas URL never resolves their pending share — there is no fallback resolution path (e.g. checking pending shares on the general "My Canvases" list). This is acceptable because pending shares also expire (`CANVAS_SHARE_INVITE_EXPIRY_HOURS`), so an unresolved share is self-cleaning either way.
- ADR 0001's line describing the pending-share mechanism as a `user.created`-webhook-based design is superseded by this ADR. ADR 0001's actual decision (one Clerk app for both deployment modes) is unaffected.
- No Clerk webhook infrastructure is needed for canvas sharing. The app does still need its own transactional email sending to deliver the invite link (new infra, not yet built).
