# One Clerk application for both deployment modes, not two

## Context

`docs/research/clerk-auth-modes.md` and `docs/research/2026-08-08-clerk-auth-modes.md` researched how to support both an org-gated deployment mode (issue #6) and a fully-open public deployment mode (issue #18) from one product. Both research docs recommend **Path A**: two separate Clerk applications, each with its own key pair, selected by a build/deploy-time `AUTH_MODE` env var — because Clerk's Restrictions/Allowlist (the feature that gates sign-up to an approved domain) is instance-wide configuration, and Restrictions requires a paid Clerk plan for production use.

## Decision

Use **one Clerk application** for both deployment modes instead. Clerk's sign-up mode stays "Public" (Clerk's free default) always, in both `org_gated` and `public` deployments — Clerk's Restrictions/Allowlist feature is never enabled, and Clerk Organizations/Verified Domains is never used. The org-domain gate is instead a plain email-domain string comparison performed in this app's own code, enforced by app-level middleware on every request: Clerk always allows an account to be created; when `AUTH_MODE=org_gated`, the app itself simply refuses to serve the product surface to users whose email domain doesn't match the domain in `ORG_ALLOWED_DOMAIN` (a single domain, not a list).

This deliberately overrides both research docs' Path A recommendation, to avoid the cost of a paid Clerk plan for the org-gated deployment.

## Considered Options

- **Path A (two Clerk applications)** — the research docs' recommendation. Rejected: still requires a paid Clerk plan for the org-gated app's Restrictions/Allowlist feature in production, and doubles the number of Clerk applications to operate.
- **Path B (one Clerk instance, Allowlist toggled between deploys)** — rejected in the original research: creates Clerk-side/env-var state drift, and can't run both modes concurrently.
- **Path C (one Clerk app, domain gate implemented in app code)** — chosen. Trades Clerk's built-in Restrictions feature for app-owned gating logic, in exchange for staying on Clerk's free tier in both modes.

## Consequences

- Enforcement happens via app-level middleware/session checks, not a webhook-triggered account rejection — this avoids the race window created by Clerk's non-guaranteed webhook delivery timing (a blocked-domain user's Clerk account can exist, but the app never serves it the product surface).
- The pending-canvas-share-by-email mechanism is unrelated to this decision and unaffected — it remains shared infrastructure across both modes for resolving canvas access grants. Its resolution mechanism described here (a Clerk `user.created` webhook) is superseded by ADR 0002, which resolves shares on canvas load instead.
