# Clerk auth modes: can one Clerk product serve both org-gated and public sign-up?

**Bottom line:** No — a single Clerk *application instance* cannot be toggled between "org-domain-gated auto-provision" and "fully-open public sign-up" by a runtime/per-request flag, because Clerk's sign-up mode and domain restrictions are **instance-wide Dashboard/CLI configuration**, not conditional logic. The practical implementation is **two separate Clerk applications** (each with its own publishable/secret key pair, one pre-configured Restricted+allowlist/verified-domain, one Public), with `AUTH_MODE` selecting which key pair to load at build/deploy time. This is supported on Clerk's Free ("Hobby") tier for the base case. Clerk's Invitation primitives (app-level `Invitation`, org-level `OrganizationInvitation`) only grant *app access* or *org membership* — never per-resource access — so per-canvas "pending share, resolve on sign-up" logic must be built by this app on top of Clerk's `user.created` webhook.

## Context

This answers issue [#19](https://github.com/sistla-gautam/infinitewhiteboard/issues/19), a child of the auth-modes wayfinder issue #18, which asks whether Clerk can serve both the org-domain-gated/auto-provisioning flow already decided in issue #6 and a new fully-open public sign-up flow from one product via a single `AUTH_MODE` env var, and whether Clerk's invitation system can satisfy per-canvas share-by-email-before-signup requirements. All findings below were pulled live from clerk.com's own docs/pricing pages during this research session (2026-08-08).

## 1. Can Clerk serve both modes from one product via a single env var?

**Answer: Not as a live runtime toggle within one instance — the restriction rules are instance-level configuration, not per-request conditionals.**

Clerk's sign-up behavior is governed by a **Sign-up mode** setting with exactly two states per instance:

> "In **Public** mode, the sign-up process is open to anyone. This mode is the **default**" and "In **Restricted** mode, user access is controlled by the application admin(s)."
— https://clerk.com/docs/authentication/configuration/restrictions

This is configured "In the Clerk Dashboard, navigate to the **Restrictions** page" (same URL), and is confirmed to be a per-instance setting: allowlist/blocklist/domain rules apply "per-instance to sign-ups" and can also be set via `npx clerk@latest config patch` against a specific instance (same URL). There is no documented mechanism for a single Clerk instance to apply Restricted-mode-for-domain-X and Public-mode-for-everyone-else simultaneously and switch between them based on an application-supplied flag per request — the mode is a property of the instance itself, changed by editing Dashboard/CLI configuration, not passed at auth-check time by the calling app.

Because each of this project's deployments is a distinct, standalone environment (per the issue's framing — "one customer's self-hosted or dedicated instance"), the natural fit is: each deployment points at its own pre-configured Clerk application. A single env var can then select *which* Clerk application's keys to load, which achieves "one build/deploy-time setting selects the mode" — but it is selecting between two pre-existing Clerk instances, not flipping a switch inside one instance.

## 2. Does this require separate Clerk "instances" with their own keys, or one instance with a runtime-toggleable rule?

**Answer: Separate instances/applications with their own keys, in the concrete sense that matters for a per-deployment env var.**

Clerk's own docs establish that "instance" (Development or Production) is the unit that carries Restrictions settings, and that credentials are issued per instance:

> "Each Clerk instance—development or production—has its own set of keys, located under Configure > API Keys, with production keys prefixed with pk_live and sk_live for the publishable and secret key respectively."
— https://clerk.com/docs/guides/development/managing-environments

> "When creating a new application within Clerk, you are provided with two instances: `Development` and `Production`."
— https://clerk.com/docs/guides/development/managing-environments

There is no Clerk primitive for "one instance, conditional restriction rule chosen by the calling application at runtime." The Restrictions/sign-up-mode setting lives on the instance; to have two live, simultaneously-different rule sets (org-gated for customer A, public for customer B) you need two instances (in practice, two separate Clerk *applications*, each contributing its own Development+Production instance pair). Clerk explicitly supports creating multiple applications under one account — the pricing page states "Unlimited applications" on every plan including Free (see §4), and Clerk's own docs/CLI reference an `apps create` workflow for provisioning additional applications for scenarios like staging.

## 3. Concretely, what would `AUTH_MODE` need to select?

**Answer: which Clerk application's publishable/secret key pair to load at build/deploy time — not a runtime restriction flag within one instance.**

Given (1) and (2), the concrete shape:

- `AUTH_MODE=org_gated` → load Clerk App A's `pk_live_...` / `sk_live_...`. App A's Dashboard is configured with:
  - **Restrictions → Sign-up mode: Restricted**, plus either a domain entry in the **Allowlist** (`https://clerk.com/docs/authentication/configuration/restrictions` — "if you add `clerk.dev` as an allowed email domain, any user with a `@clerk.dev` email address can sign up for your application"), or Organizations' **Verified Domains** with **Automatic Invitation** mode, which auto-enrolls matching-domain sign-ups with no admin click: "Clerk automatically invites users to join the Organization when they sign up and they can join anytime" (https://clerk.com/docs/organizations/verified-domains). This is the mechanism that satisfies issue #6's "auto-provisioned... no manual invite step."
  - This matches issue #6's existing decision (org-domain-gated + auto-provision).
- `AUTH_MODE=public` → load Clerk App B's `pk_live_...` / `sk_live_...`. App B's Dashboard is configured with **Restrictions → Sign-up mode: Public** (Clerk's documented default — "open to anyone," same restrictions URL), with the Allowlist/Blocklist left off (or Blocklist only used for abuse prevention, not domain gating).

Other Clerk features referenced in the issue map as follows: `organizationSettings` and the **Restrictions** dashboard page are the org-gating controls (§1 above); the `sign_up` lifecycle is observed via the `user.created` webhook (see §5), not a dedicated "sign_up webhook" — Clerk's webhook catalog names events `user.created`, `user.updated`, `organizationInvitation.created`, `organizationInvitation.accepted`, `organizationInvitation.revoked`, etc., discoverable via "the Webhooks page in the Clerk Dashboard... Event Catalog tab" (https://clerk.com/docs/webhooks/overview).

## 4. Pricing-tier implications of running two Clerk applications

**Answer: Running two separate Clerk applications is supported on the Free tier by itself; the org-gated app's specific *Verified Domains + Automatic Invitations* feature is listed under a $100/mo add-on on the pricing page even though Clerk's feature docs don't state a hard gate — this is a genuine ambiguity flagged below.**

From Clerk's pricing page (https://clerk.com/pricing, fetched live):

- **Hobby (Free)**: "Unlimited applications" and "50,000 MRU (monthly retained user) limit per app" — each application gets its own 50k free-user allowance, and there is no stated cap on the number of free applications an account may run.
- Clerk bills by **MRU (Monthly Retained User)**, not raw MAU: "A user only counts as retained if they return to your app at least 24 hours after signing up" (per fetched page content) — relevant if estimating cost, since two low-traffic deployments would each get their own 50k-MRU free allowance.
- **Organizations / B2B**: "100 MROs (monthly retained organizations) included per app" on the base plans, with an **Enhanced B2B Authentication add-on** at "$100/mo ($85/mo billed annually)" that unlocks "Unlimited members per Organization," "Link Enterprise Connections... with Organizations," **"Verified Domains & Automatic Invitations,"** and "Custom Roles & Rolesets" (https://clerk.com/pricing). Absent the add-on, an Organization is capped at "a maximum of 20 members" (https://clerk.com/docs/guides/organizations/configure).

**Open ambiguity (flagging per instructions rather than guessing):** The pricing page's bullet list attaches "Verified Domains & Automatic Invitations" specifically to the paid Enhanced add-on. However, the feature documentation pages themselves (https://clerk.com/docs/organizations/verified-domains, https://clerk.com/docs/guides/organizations/add-members/verified-domains, https://clerk.com/docs/guides/organizations/configure) describe how to configure Verified Domains and Automatic Invitation without any inline callout stating the feature is plan-gated — the only explicit plan-gated limit found in the docs text is the 20-member org cap. It is not possible to determine from the fetched pages alone whether a small single-company org (well under 20 members) can use Verified Domains + Automatic Invitation on the Free/Hobby tier, or whether the feature is hard-gated behind the $100/mo add-on regardless of org size. **This needs a Clerk support ticket or a live signup-flow spike to confirm before committing to the free tier for the org-gated deployment.** If the allowlist-domain approach (Restrictions → Allowlist, not Organizations) is used instead of Verified Domains/Organizations, that path has no plan-gating mentioned anywhere in the fetched docs and appears to be free-tier compatible — this may be the safer choice to sidestep the ambiguity, and it still satisfies "auto-provision on first sign-in" without needing Organizations at all.

Bottom line for cost: two Clerk applications is not itself a forced upgrade ("Unlimited applications" is on every tier, including Free). The one cost risk is specifically the Verified-Domains/Auto-Invite Organizations feature for the org-gated deployment, which the pricing page associates with the $100/mo add-on.

## 5. Does Clerk's invitation flow satisfy per-canvas share-by-email-before-signup?

**Answer: No — Clerk's invitation primitives are scoped to app access or org membership, never to an arbitrary app-defined resource like a canvas. This project must build its own "pending share" layer on top of Clerk's `user.created` webhook.**

**What Clerk provides, and its scope:**

- **App-level `Invitation`** (`clerkClient.invitations.createInvitation`): "Invitations are only used to invite users to your application. The application will still be available to everyone even without an invitation" (https://clerk.com/docs/users/invitations). It sends "an email to the invited user with a unique invitation link," and on completion "their email address will be automatically verified." It supports a `publicMetadata` param that "will end up in the user's public metadata" once they sign up — useful for carrying *app-level* onboarding context, but this is metadata on the resulting `User` object, not a link to a specific canvas/resource record.
- **Org-level `OrganizationInvitation`** (`createOrganizationInvitation()`): scoped strictly to organization membership — required params are `emailAddress`, `organizationId`, and `role` (https://clerk.com/docs/reference/backend/organization/create-organization-invitation). On acceptance, "Clerk stores the **invitation** metadata (`OrganizationInvitation.publicMetadata`) in the Organization **membership's** metadata (`OrganizationMembership.publicMetadata`)" (https://clerk.com/docs/guides/organizations/add-members/invitations) — again, the grant is "membership in this Organization," not "access to this specific resource inside the app."

Neither primitive has any concept of an app-defined resource (a canvas). Both grant either "can this person use the app" (app Invitation) or "is this person a member of this Organization" (OrganizationInvitation) — confirmed by the required-parameter shape (`emailAddress` + `organizationId` + `role`, no resource identifier field) and the metadata being deposited on the `User`/`OrganizationMembership` object, not on any app-specific entity.

**Why the app must build its own layer:**

Because canvas-level sharing requires the grant to survive from "invite by email" (before the invitee has an account) to "resolve to a concrete canvas-access row" (after they sign up), and Clerk has no primitive for "access to resource X," this project needs to:

1. Store a `pending_share` record in the app's own database, keyed by invitee email (`canvas_id`, `email`, `granted_by`), created at share time regardless of whether the invitee has a Clerk account yet.
2. Listen for Clerk's **`user.created`** webhook. The event payload includes the new user's email addresses with verification status — e.g. `"email_addresses": [{ ..., "verification": {"status": "verified", "strategy": "ticket"} }]` (https://clerk.com/docs/webhooks/overview) — confirming Clerk already verified the address at sign-up.
3. On receipt, look up `pending_share` rows matching the newly-verified email (or check `primaryEmailAddress` on session/sign-up completion as an alternative synchronous path) and materialize them into real per-canvas access-grant rows in the app's own database.

Clerk's own webhook guidance supports this general "sync into your own DB on `user.created`" pattern generically (for syncing user records, not specifically for resource-share resolution): "The most notable use case for syncing Clerk data is if your app has social features where users can see content posted by other users... listen to the `user.created` event to perform an initial database insertion" (https://clerk.com/docs/webhooks/sync-data). Clerk's docs do **not** describe using this pattern to resolve pending resource-shares — that logic is not a Clerk feature and was not found in any fetched primary source; it is this app's responsibility to implement. Note also Clerk's own reliability caveat: "Webhook deliveries are not guaranteed and may occasionally fail due to problems like network issues" (same URL) — the pending-share resolver needs to be idempotent/retry-safe (e.g. also resolved lazily on next sign-in) rather than relying solely on webhook delivery.

## Implications for this project

**Recommendation:** `AUTH_MODE` should select **which of two separate Clerk applications' key pairs to load** at build/deploy time (e.g. `CLERK_PUBLISHABLE_KEY` / `CLERK_SECRET_KEY` resolved from `AUTH_MODE=org_gated` → App A's keys, `AUTH_MODE=public` → App B's keys), not a runtime restriction flag within a single instance — Clerk has no per-request/conditional restriction primitive; Restrictions/sign-up-mode is Dashboard/CLI-level instance configuration (§§1–3).

- **App A (org-gated)**: Restrictions → Sign-up mode: Restricted, with either an Allowlist domain entry (simplest, no clear plan-gating found) or Organizations Verified Domains + Automatic Invitation (richer, but pricing-gating is ambiguous per §4 — confirm with Clerk before relying on Free tier for this).
- **App B (public)**: Restrictions → Sign-up mode: Public (Clerk's documented default), no allowlist/domain restriction.
- Both applications can live on Clerk's Free/Hobby tier from a "number of applications" standpoint ("Unlimited applications" per plan) — cost pressure, if any, comes specifically from the Organizations/Verified-Domains add-on for App A, not from running two apps per se.

**What Clerk provides vs. what this app must build, for per-canvas sharing:**

| Layer | Clerk provides | This app must build |
|---|---|---|
| "Can this person sign in to the product at all" | Yes — app `Invitation` / org gating / Restrictions | — |
| "Is this person a member of an org" | Yes — `OrganizationInvitation`, Verified Domains auto-join | — |
| "Does this person have access to canvas #1234" | No primitive for this | Yes: `pending_share(canvas_id, email, granted_by)` table + resolver triggered on Clerk's `user.created` webhook (matching verified email → materialize real canvas-access grants), with idempotent/retry-safe resolution given Clerk's undelivered-webhook caveat |

**Open item to resolve before implementation:** file a Clerk support question (or run a live signup spike on a fresh free-tier Clerk app) to confirm whether Verified Domains + Automatic Invitation is usable on the Free tier for a single small organization, or whether it is hard-gated behind the $100/mo Enhanced B2B add-on — the fetched pricing page and fetched feature docs disagree on this point (§4).
