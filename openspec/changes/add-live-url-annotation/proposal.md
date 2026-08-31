## Why

We are building a practice clone of usepastel.com. Its defining feature — the thing that differentiates it from a generic comment/ticket tool — is letting anyone open a link and leave pinned comments directly on a live external website, with no login and no cooperation required from the site owner. This proposal scopes the first buildable milestone: owner-created projects, server-side proxying/rendering of the target URL, and click-to-pin threaded commenting for guests and owners.

## What Changes

- Owners (authenticated) can create a **project** by supplying a target URL.
- Visiting a project's share link **proxies and renders the target URL** through our own domain: fetch the page server-side, rewrite asset/link URLs, strip framing-blocking headers, and inject an overlay script before serving it to the browser.
- The injected overlay lets any viewer **click anywhere on the rendered page to drop a pin** and leave a comment, anchored by CSS selector + relative offset (not raw x/y) so it survives viewport/scroll differences.
- Guests reach a project via its share link **with no account/login required**; owners authenticate normally.
- Comments support **threaded replies**, a **resolved/unresolved** state, and free-form **tags**.
- Comment and resolution changes **broadcast live** to everyone currently viewing the same project.
- Out of scope for this change (later milestones): responsive breakpoint switching, copy-edit mode, PDF/image annotation, PM-tool/AI-agent task conversion, pausing commenting, and feedback deadlines.

## Capabilities

### New Capabilities
- `projects`: Owner-facing capability to create and manage a project pointing at a target URL, and to generate its guest share link.
- `url-proxy`: Server-side capability that fetches a target URL, rewrites it to be safely servable from our domain, and injects the annotation overlay script.
- `annotations`: Capability for placing pins, writing threaded comments, tagging, and resolving feedback on a rendered project page, with realtime sync across viewers.

### Modified Capabilities
- None — this is a greenfield build with no existing specs.

## Impact

- New Next.js app: marketing/dashboard routes (owner-facing) and a proxy route (`/p/[projectId]`) that serves rewritten HTML.
- New Supabase schema: `projects`, `pins`, `comments`, `tags` tables; Supabase Auth for owners; Supabase Realtime channel per project; Supabase Storage not yet needed at this stage (no screenshots/PDFs/images in this slice).
- New client-side overlay bundle (`overlay.js`) injected into proxied pages, communicating with our API/Realtime channel.
- No existing code affected (greenfield repo).
