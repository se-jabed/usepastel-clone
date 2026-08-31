## Purpose

Fetches a project's target URL on the server, rewrites the response so it can be safely and correctly served from our own domain, and injects the annotation overlay script so viewers can interact with it.

## ADDED Requirements

### Requirement: Proxied response renders the target page
When a viewer opens a project's share link, the system SHALL fetch the project's target URL server-side and return a rendering of that page, served from our own domain, rather than redirecting the browser to the original URL.

#### Scenario: Valid project link renders the target page
- **WHEN** a viewer opens a share link for a project whose target URL is reachable
- **THEN** the viewer's browser renders the target page's content while staying on our domain

### Requirement: Asset and link URLs are rewritten to resolve correctly
The system SHALL rewrite relative and absolute asset references (images, stylesheets, scripts, links) in the proxied HTML so that they continue to resolve to the correct resources when served from our domain.

#### Scenario: Relative assets load correctly
- **WHEN** the target page references an image or stylesheet via a relative URL
- **THEN** the proxied page loads that asset successfully

#### Scenario: Absolute assets load correctly
- **WHEN** the target page references an asset via an absolute URL on its own origin
- **THEN** the proxied page loads that asset successfully

### Requirement: Framing-blocking response headers are not propagated
The system SHALL NOT forward the target site's `X-Frame-Options` or restrictive `Content-Security-Policy` framing directives to the viewer's browser, so the proxied page is not blocked from rendering.

#### Scenario: Target site sets framing-blocking headers
- **WHEN** the target URL's response includes `X-Frame-Options` or a `frame-ancestors`/`default-src` CSP directive that would block embedding
- **THEN** the proxied response served to the viewer omits those restrictions and the page still renders fully

### Requirement: Overlay script is injected into every proxied page
The system SHALL inject the annotation overlay script into every successfully proxied page before it is served, so that annotation functionality is available without any action from the viewer.

#### Scenario: Overlay is present and interactive
- **WHEN** a viewer's browser finishes loading a proxied project page
- **THEN** the annotation overlay script has loaded and is ready to accept pin placement

### Requirement: Proxy fails gracefully on unreachable or blocking targets
When the target URL cannot be fetched (network error, timeout, or non-success response), the system SHALL show the viewer a clear error state instead of a broken or partially-rendered page.

#### Scenario: Target URL is unreachable
- **WHEN** a viewer opens a share link whose target URL times out or returns a network error
- **THEN** the system displays an error message to the viewer and does not attempt to render a broken page
