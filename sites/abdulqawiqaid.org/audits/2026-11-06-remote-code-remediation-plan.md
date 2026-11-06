# Remote-code remediation plan — Blogger theme

**Status:** analysis complete; code migration blocked pending a current raw export and feature inventory  
**Branch:** feature/blogger-site-integration  
**Production:** unchanged; no Blogger page or theme was edited

## Confirmed dependency chain

1. The archived theme loads the Blogger page endpoint for page ID 137042360492087078 using callback sp_db.
2. The captured feed identifies the page as Database V9 Scripts on blog ID 5637748011392113640, with alternate URL https://seoplusdatabase8.blogspot.com/p/database-v9-scripts.html.
3. The theme extracts the page's first script element, writes its source to sessionStorage key storg, and evaluates it.
4. Lazy-loading code evaluates the stored source again.
5. The captured page script contains three more eval sites:
   - bloggerJS extracts and evaluates script fragments from Blogger widget-related output.
   - a second fragment is evaluated after loading Blogger widgets.js.
   - redirect-preview code fetches HTML, extracts script elements, and evaluates them as rScripts.

The evidence is a PDF snapshot captured in the prior review workflow. Its Atom entry metadata reports an update timestamp of 2025-11-30 and author Unknown. It does not prove the live endpoint's current content, current page ownership, or the current runtime behavior.

## Why no blind patch was applied

Removing the outer eval or replacing it with script-element injection would still execute the same remote source. Removing the loader outright may disable code supplied by Database V9 Scripts, including Blogger widget integration, comments, redirect previews, and other theme behaviors. The source is obfuscated and its full functional contract has not yet been validated in a browser. A blind change would risk replacing a security issue with a broken production theme.

## Target architecture

- Keep the uploaded original XML immutable.
- Migrate only required, reviewed functionality into a readable, versioned JavaScript asset controlled by the repository or an explicitly controlled static host.
- Replace remote executable HTML with a strict JSON configuration document. Validate schema, types, lengths, and allowed values; do not evaluate strings.
- Remove the storg sessionStorage executable-code cache and all eval replay paths.
- Use normal script src loading for vetted dependencies; do not regex-extract executable source from Blogger widgets.js or arbitrary fetched HTML.
- Render redirect previews as sanitized, inert content. Do not execute scripts from fetched destination pages.
- Validate redirect destinations with URL parsing and an explicit hostname/protocol allow-list. Reject credentials, javascript:, data:, protocol-relative, malformed, and unapproved destinations.
- Keep consent and AdSense initialization separate from the migrated application code; preserve the existing consent defaults until behavior is tested.

## Required migration and validation sequence

1. Obtain a fresh raw export of the current Database V9 Scripts page or its exact current Blogger JSON feed response.
2. Compare that export with the archived PDF snapshot and identify all changed scripts and external URLs.
3. Produce a readable function inventory and dependency map; identify which functions are actually invoked by the theme.
4. Reimplement only required features without eval, Function constructors, HTML-string execution, or script extraction from fetched pages.
5. Create a candidate theme file under a separate candidate path; never overwrite the original archive.
6. Run strict XML parsing, checksum verification of the immutable original, static security scans, and diff review.
7. Import the candidate only into a private Blogger test blog. Test home, posts, static pages, labels, search, comments, pagination, lazy loading, redirects, consent, AdSense placeholders, mobile layout, console errors, and network requests.
8. Keep the PR draft until test evidence is attached. Production upload, page edits, merge, and deployment require separate explicit approval.

## Exit criteria

- No executable JavaScript is fetched from Blogger post/page HTML and evaluated.
- No script extracted from redirect destination HTML is executed.
- No arbitrary URL from query parameters or local storage is trusted as a redirect destination.
- Required functionality has a documented owner, source, and test.
- The candidate passes the private Blogger compatibility matrix.
- Original XML hash remains unchanged.
