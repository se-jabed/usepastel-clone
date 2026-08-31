## Context

See proposal.md - Why. This is a greenfield build (practice clone of usepastel.com); there is no existing code or architecture to integrate with. Stack chosen for the whole project: Next.js (App Router) + TypeScript, Tailwind CSS, Supabase (Postgres + Auth + Realtime), deployed on Vercel.

## Goals / Non-Goals

**Goals:**
- Render an arbitrary third-party, primarily server-rendered marketing site (Webflow/Shopify-style) through our own domain so an overlay script can attach to it.
- Anchor pins in a way that survives different viewports/scroll positions.
- Sync comment/pin state live across concurrent viewers of the same project.

**Non-Goals:**
- Supporting arbitrary heavy client-side-rendered SPAs as proxy targets (their JS-constructed URLs and client routing are not reliably rewritable; explicitly out of scope for this milestone).
- Responsive breakpoint switching UI, copy-edit mode, PDF/image annotation, PM/AI integrations, pause/deadline controls - all deferred to later changes.
- Authenticating or rate-limiting third-party target sites beyond basic timeout/error handling.

## Decisions

**Proxy at the Next.js route layer (Node runtime), not the Edge runtime.**
Rewriting HTML (parsing tags, rewriting `src`/`href`/`srcset`/inline `style="url(...)"`) needs a full DOM/HTML parser; Edge runtime's `HTMLRewriter`-style streaming transforms are more limited for this than a Node-based parser (e.g. `parse5` or `cheerio`). Node runtime route handlers give us that flexibility at the cost of not being edge-distributed. Acceptable trade-off for a practice build where latency isn't a hard requirement.

**Rewrite URLs to absolute, not proxy every sub-resource through our domain.**
Two options considered:
1. Rewrite relative URLs to absolute URLs pointing at the *original* origin (images/CSS/JS load directly from the target site).
2. Rewrite every URL to route back through our proxy (`/p/[projectId]/asset?url=...`), so all traffic passes through us.
Option 1 is chosen for this milestone: far simpler, avoids reimplementing a general-purpose asset proxy, and is sufficient because assets (unlike the top-level document) aren't normally blocked by `X-Frame-Options`/CSP framing rules - those only govern top-level framing. Trade-off: if the target's assets are protected by CORS or referrer checks tied to their own domain, some may fail to load; acceptable known limitation, called out in Risks below.

**Strip `X-Frame-Options` and CSP framing directives from the proxied response; pass through everything else (including third-party analytics/tracking scripts) unmodified.**
Alternatives considered: stripping all third-party `<script>` tags (cleaner, but risks breaking sites whose own functionality depends on that JS, and adds a maintenance burden of detecting "which scripts are trackers"). For a practice clone, correctness of rendering matters more than tracker hygiene, so we pass scripts through as-is and only touch the specific headers that would otherwise block rendering.

**Pin anchoring: CSS selector + relative offset within the matched element, computed client-side by the overlay script.**
Raw `(x, y)` viewport coordinates were rejected (per proposal) because they break under different viewports/scroll/responsive layouts - already reflected in the `annotations` spec. The overlay computes a stable selector (e.g. nearest ancestor with an `id`, else a structural path) plus the click's offset percentage within that element's bounding box, so playback can re-locate the same visual spot across viewport sizes.

**Guest identity: ephemeral display name only, no anonymous auth session.**
Guests provide a display name stored with each comment; no Supabase anonymous-auth session or cookie-based identity is created for this milestone. Simpler, matches the "no login, no friction" positioning, and sidesteps session-management complexity. Trade-off: no way to edit/delete a guest's own prior comment after the fact in this milestone (acceptable, not a stated requirement).

**Realtime via a Supabase Realtime channel keyed per project.**
Chosen over Pusher/Socket.io because it's already part of the chosen stack (one fewer service to run/pay for), and Postgres row-level changes on `pins`/`comments` can be broadcast directly via Supabase's Postgres Changes feature without a separate pub/sub layer.

## Risks / Trade-offs

- **[Risk]** Client-rendered SPA targets will partially or fully fail to proxy correctly (JS-constructed navigation, service workers, dynamically fetched data from the original origin blocked by CORS). → **Mitigation**: explicitly scoped as a non-goal; document supported target types as "server-rendered/static sites" in user-facing copy.
- **[Risk]** Some target-site assets may fail to load if they enforce strict CORS/referrer checks tied to their own origin, since we rewrite to absolute URLs on the original origin rather than proxying assets ourselves. → **Mitigation**: accept as a known limitation for this milestone; revisit with a full asset-proxy approach if it proves common in practice.
- **[Risk]** We become an intermediary for the target site's own cookies/third-party scripts, which could behave unexpectedly when served under our origin (e.g. same-site cookie scoping, mixed-content warnings). → **Mitigation**: out of scope to fully solve now; flagged for revisit once real target sites are tested against.
- **[Risk]** CSS-selector-based anchoring can still fail if the target site's markup changes between visits (elements removed/reordered). → **Mitigation**: accepted as inherent to any DOM-anchoring approach; fall back to displaying an "anchor not found" indicator near the last known position rather than crashing.

## Migration Plan

Not applicable - greenfield build, no existing users or data to migrate.
