# Research: Can Clerk serve both org-gated and public sign-up from one product?

Resolves GitHub issue [#19](https://github.com/sistla-gautam/infinitewhiteboard/issues/19), part of the wayfinder map issue [#18](https://github.com/sistla-gautam/infinitewhiteboard/issues/18) ("Auth modes: org-gated vs. public deployment"). Research-only — no implementation decisions are made here.

Date: 2026-08-08. All claims below are sourced directly from Clerk's own docs pages (clerk.com/docs), fetched live; each claim links to its source.

## 1. Can one Clerk product switch between org-gated and fully-open public sign-up via a single build/deploy-time env var?

Not cleanly as a single toggle inside one Clerk application instance. Clerk's relevant primitives are:

- **Verified Domains** (part of Organizations) — lets an org admin verify an email domain and auto-invite or auto-suggest matching users to *join that organization*. Critically, this **does not restrict sign-up to the app at all**: "Verified Domains do not restrict sign-up to your application... Users outside the domain can still sign up to your Clerk application normally." It's configured application-wide in the Clerk Dashboard's Organizations settings, applies to all organizations in that instance, and cannot be set per-organization. ([Verified domains](https://clerk.com/docs/guides/organizations/verified-domains))
- **Restrictions / Allowlist** — the feature that actually gates sign-up by domain. Enabling the allowlist and adding an allowed domain (e.g. `acme.com`) blocks sign-up for every other domain, application-wide: "only users with those identifiers will be able to sign up to your application, while others will be blocked." This is a per-application-instance Dashboard toggle (Restrictions page → Allowlist tab), not a per-request or per-org rule. There's also a CLI path (`npx clerk@latest config patch` against key `auth_access_control`) for scripting the same instance-level setting. ([Restricting access](https://clerk.com/docs/guides/secure/restricting-access))

So the actual domain-gate Clerk product needs for org mode is **Restrictions/Allowlist**, not Organizations/Verified Domains (Organizations still matters separately for canvas-collaborator grouping if the app wants it, but not for the sign-up gate itself). Allowlist is a single on/off + domain-list setting that lives on the Clerk side (Dashboard or Backend API/CLI), not in the app's own env vars — so an app-side env var cannot, by itself, flip it. Toggling it requires a corresponding write against Clerk's config (Dashboard click or API/CLI call), separate from the app's deploy.

## 2. Does this require separate Clerk application instances, or can one instance apply conditional rules toggled by an env var?

Two workable paths, with different tradeoffs:

**Path A — two separate Clerk applications (recommended for this project's "never mixed at runtime, single env var" requirement).** Create two Clerk applications, each with its own publishable/secret key pair: one with Allowlist enabled and the org's domain(s) added (org-gated), one with Allowlist off (fully open). The deploy-time env var then selects *which key pair to load* — a value the app's own env var fully controls, with no dependency on a second, out-of-band Clerk-side config change at deploy time. This matches the ticket's constraint that the env var alone determines the mode.

**Path B — one Clerk application, Allowlist toggled between deploys.** Technically possible (Allowlist is just a Dashboard/API setting on one instance), but the toggle is Clerk-side state, not app-side. A single app env var cannot itself flip it — you'd need a paired step (manual Dashboard click, or a CLI/API call in the deploy pipeline) kept in sync with the env var, which is exactly the kind of state-drift risk the ticket's "single env var, not runtime-toggleable, not admin-configurable" wording seems to want to avoid. Path B also can't run both modes *concurrently* (e.g., two live production deployments at once), since the Allowlist is a single instance-wide setting — it's inherently one-mode-at-a-time.

**Path C — bypass Clerk's native domain gate entirely.** Use one Clerk instance with public sign-up always enabled, and implement the domain check in app code (e.g., in a `user.created` webhook, checking the new user's email domain against an env-var-supplied allow-list, then rejecting/deleting non-matching accounts or gating downstream app access). This is the only option where a *single app-side env var* is the sole, literal source of truth with no paired Clerk-side state — at the cost of building and maintaining the gating logic yourself instead of using Clerk's built-in Allowlist.

Docs do not describe any built-in way to apply Allowlist conditionally per-request within one instance based on an external flag — it's a static instance-level setting.

## 3. What would the env var concretely need to select?

Depends on which path is chosen:
- **Path A (two instances):** the env var selects *which Clerk instance's publishable/secret keys* the build loads (e.g. `CLERK_MODE=org-gated` → org app's keys, `CLERK_MODE=public` → public app's keys). No Clerk-side runtime flag is involved; each instance is pre-configured once and left alone.
- **Path B (one instance):** the env var alone is insufficient; it would need to be one half of a two-part config where the other half is a manual/scripted Clerk Dashboard-or-API change to the Allowlist toggle, kept in lockstep with the env var.
- **Path C (app-level gating):** the env var supplies the allowed-domain list (or an "org-gated"/"public" mode flag) to the app's own webhook/middleware logic; Clerk instance config itself never changes.

## 4. Pricing-tier implications of running multiple separate Clerk instances/organizations

- **Applications:** all Clerk plans (including free Hobby) include "unlimited applications," so running two separate Clerk applications (Path A) has no per-app cost by itself. ([Pricing](https://clerk.com/pricing))
- **Restrictions/Allowlist in production requires a paid plan.** Clerk's docs state directly: "This feature requires a paid plan for production use" — free only in development/test mode. ([Restricting access](https://clerk.com/docs/guides/secure/restricting-access)) So the org-gated instance (whichever path exposes Allowlist) needs at least the Pro plan ($25/mo, or $20/mo billed annually) once it's live in production; the public instance, having no restriction turned on, can stay on the free Hobby tier.
- **Organizations feature itself (needed if canvas-collaborator grouping uses Clerk Organizations) is included free**: "Free plans include up to 50 MROs [monthly retained organizations] in development and 100 in production," per the Organizations overview page. Only high organization-count usage (100+ orgs/month) triggers the paid "B2B Authentication" add-on ($100/mo). For a single-org deployment this is a non-issue.
- **MRU (monthly retained user) limits:** Hobby/free covers 50,000 MRU per app; Pro includes the same 50,000 with metered overage at $0.02/MRU/month beyond that.

Bottom line: two Clerk applications is fine on cost grounds (unlimited apps), but the org-gated app specifically needs a paid (Pro-tier-or-above) Clerk plan once in production because Allowlist/Restrictions is gated to paid plans for production use — the public app does not need this and can run on Hobby.

## 5. Does Clerk have a built-in invitation flow for "share a canvas by email with someone who has no account yet, auto-granting access on signup"?

Partially — Clerk gives two invitation primitives, neither of which is scoped to an arbitrary in-app resource like a single canvas:

- **Application invitations** (`createInvitation`) — invite an email to the *whole app*. If the invited address has no account, clicking the link takes them to sign-up with their email pre-verified. Supports `publicMetadata` that lands on the user's profile after they accept and sign up. But: "invitations alone don't restrict access... the application will still be available to everyone even without an invitation" — it's an onboarding nicety, not an access-control primitive, and it carries no built-in concept of "this invite grants access to canvas X." ([Invite users to your application](https://clerk.com/docs/guides/users/inviting), [createInvitation](https://clerk.com/docs/references/backend/invitations/create-invitation))
- **Organization invitations** (`createOrganizationInvitation`) — invite an email to join a specific *Organization*. Works for both existing and brand-new users; new users sign up to accept. This is scoped to org membership, not to an individual canvas. ([Organization invitations](https://clerk.com/docs/guides/organizations/invitations))

**Conclusion:** neither primitive natively models "grant access to this specific canvas once this specific email signs up." Clerk's invitations can be reused as the *account-creation/pre-verification* mechanism (skip email confirmation, know the invited address is validated), and `publicMetadata` could theoretically carry a small hint — but the actual canvas-sharing semantics (store a pending share keyed by the invited email, then resolve it against the new user's verified email once their Clerk account exists) is application logic the app must build itself on top of Clerk's user/email data, most likely via a `user.created` (or session) webhook that checks the newly-verified email against a table of pending canvas shares and grants access at that point. This matches the app-level pattern the ticket description already anticipated.

## Sources

- https://clerk.com/docs/guides/organizations/verified-domains
- https://clerk.com/docs/guides/secure/restricting-access
- https://clerk.com/docs/guides/organizations/overview
- https://clerk.com/docs/guides/users/inviting
- https://clerk.com/docs/references/backend/invitations/create-invitation
- https://clerk.com/docs/guides/organizations/invitations
- https://clerk.com/pricing
