# Remote-code remediation plan — Blogger theme

**Status:** analysis baseline; implementation blocked pending current source export and feature inventory  
**Target branch:** `audit/site-security-review-2026-11-06`  
**Production:** unchanged

## Objective

Remove the remote executable-code channel from the Blogger theme without breaking required widgets, comments, lazy loading, redirect previews, consent, or advertising behavior. Keep the original archived XML immutable.

## Confirmed dependency chain from available snapshot

1. The theme requests Blogger page ID `137042360492087078` using callback `sp_db`.
2. The archived feed snapshot identifies the page as “Database V9 Scripts” on blog ID `5637748011392113640`, with alternate URL `https://seoplusdatabase8.blogspot.com/p/database-v9-scripts.html`.
3. The theme extracts a script from the Blogger page HTML, stores its text under `sessionStorage` key `storg`, and evaluates it.
4. Lazy-loading paths evaluate the stored script text again.
5. The captured remote script includes three additional eval paths: Blogger widget-output extraction, a second widget script fragment, and script extraction from fetched redirect HTML.

The evidence comes from a captured PDF snapshot with Atom update metadata of 2025-11-30 and author “Unknown”. It does not prove present endpoint content, page ownership, or runtime behavior.

## Why blind removal is prohibited

Removing only the outer `eval`, or replacing it with a dynamically injected script element, would continue to execute remote content. Deleting the loader without a feature inventory could break widgets, comments, redirect previews, and lazy loading. No production patch should be attempted until the current page source and functional dependencies are confirmed.

## Target architecture

- Preserve the original XML as the canonical immutable archive.
- Migrate only required, reviewed functions into readable, versioned code controlled by the repository or an explicitly controlled static host.
- Replace remotely supplied executable HTML with a strict JSON configuration document. Validate schema, types, field lengths, and allowed values; never evaluate strings.
- Remove the `storg` executable-code cache and every eval replay path.
- Load vetted dependencies through conventional script URLs; do not extract script text from Blogger widgets or arbitrary fetched HTML.
- Render redirect previews as sanitized, inert content.
- Parse redirect destinations using `URL` and enforce an explicit protocol and hostname allow-list. Reject credentials, `javascript:`, `data:`, protocol-relative, malformed, and unapproved destinations.
- Keep consent and advertising initialization separate; preserve the current archived theme until the candidate behavior has been tested.

## Required sequence

1. Obtain a fresh raw export of the current Database V9 Scripts page or its exact current Blogger JSON feed response.
2. Compare it with the archived PDF snapshot and record changed script blocks, external hosts, and ownership evidence.
3. Produce a function inventory and dependency map. Trace which functions are invoked by the theme and which are dead code.
4. Reimplement only required features without `eval`, Function constructors, script-text extraction, or HTML-string execution.
5. Store candidate work under a separate path such as `sites/abdulqawiqaid.org/candidates/`; never overwrite the original theme.
6. Run XML parsing, static security scans, external-host inventory, redirect tests, and exact source-diff review.
7. Import only into a private Blogger test blog.
8. Test homepage, posts, static pages, labels, search, comments, pagination, lazy loading, redirects, Consent Mode, AdSense placeholders, mobile/desktop rendering, keyboard accessibility, console errors, and network/storage behavior.
9. Keep the pull request draft until the test matrix and evidence are attached.
10. Request separate explicit approval before production upload, Blogger page edits, merge, or deployment.

## Exit criteria

- No executable JavaScript is fetched from Blogger page/post HTML and evaluated.
- No script extracted from widget output or redirect destination HTML is executed.
- Query parameters and local storage never act as trusted redirect destinations without validation.
- Every retained external dependency has an identified owner, purpose, and review status.
- Required features have documented regression tests.
- The candidate passes the private Blogger compatibility, privacy, accessibility, SEO, and performance checks.
- The original XML checksum and Git blob identity remain unchanged.

## Rollback

Keep the archived original XML untouched and record its existing manifest. Any candidate test must be performed in a private test blog. Rollback in the test environment means restoring the original theme export; production changes remain out of scope until separately approved.
