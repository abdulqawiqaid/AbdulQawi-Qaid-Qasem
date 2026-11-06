# Static security and technical audit — Blogger theme

**Review date:** 2026-11-06  
**Repository:** `abdulqawiqaid/abdulqawiqaid.org`  
**Review branch:** `audit/site-security-review-2026-11-06`  
**Source:** canonical Blogger theme XML archived at repository root  
**Source integrity:** SHA-256 `e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4`; 391,267 bytes  
**Method:** static inspection of the supplied theme and archived Blogger feed snapshot; no script execution and no production changes.

## Executive summary

The theme and the archived Blogger feed response reveal a high-risk remote-code execution design. The theme retrieves a Blogger page titled “Database V9 Scripts”, extracts JavaScript from the returned page content, stores the script in `sessionStorage`, and executes it using `eval`. The captured page script contains further execution paths that extract and evaluate scripts from Blogger widget output and fetched redirect HTML.

This is a serious design risk because executable code supplied through editable remote page content can run in the visitor's blog origin. It is not, by itself, proof that the site has been compromised. The feed PDF is a historical snapshot and does not establish the current endpoint content, ownership, or live runtime behavior.

No code remediation, theme upload, Blogger page edit, domain/DNS change, AdSense change, Analytics change, consent change, or production deployment was performed.

## 1. Canonical source integrity

The repository manifest records the original theme as 391,267 bytes with SHA-256 `e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4` and Git blob SHA-1 `6826a09292ac3882efde09f63de0d06d1257f0c6`. GitHub Actions run #4 successfully checked byte length, SHA-256, Git blob identity, and XML well-formedness. The original root-level XML must remain the single canonical archive and must not be edited as part of remediation.

## 2. Prioritized findings

### F-01 — Remote Blogger page content is executed as JavaScript (Critical / P0)

The theme callback `sp_db` requests the Blogger JSON-in-script feed for page ID `137042360492087078` on blog ID `5637748011392113640`. The captured response identifies the page as “Database V9 Scripts” and includes an alternate URL on `seoplusdatabase8.blogspot.com`. The archived Atom metadata reports an update timestamp of 2025-11-30 and author “Unknown”; it is historical evidence, not proof of current ownership or current content.

The theme parses the feed's HTML content with DOMParser, selects a script element, reads its script text, writes it to `sessionStorage` under key `storg`, and evaluates it. Lazy-loading paths evaluate the stored source again.

**Impact:** code supplied by the remote Blogger page can run with the privileges of the visitor's blog page. A remote page edit or compromise of its delivery path could therefore alter code executed for site visitors.

**Required remediation:** eliminate remote HTML-to-script extraction, `eval`, and executable `sessionStorage` replay. Migrate only verified required behavior into readable, versioned, locally controlled code. Use a strict JSON configuration schema for data; never execute configuration strings. Merely replacing `eval` with dynamically created script elements is not a safe fix.

### F-02 — Nested dynamic execution in the captured remote script (Critical / P0)

The archived Database V9 Scripts snapshot contains three additional `eval` call sites:
- `bloggerJS()` extracts script fragments from Blogger widget-related output and evaluates them.
- Another script fragment is evaluated after loading Blogger `widgets.js`.
- Redirect-preview logic fetches HTML, extracts script elements, and evaluates the collection as `rScripts`.

The exact current runtime and controllability of the redirect destination have not been established from the snapshot alone.

**Required remediation:** remove regex extraction and execution of scripts from widget output and fetched pages. Load only vetted dependencies using normal script URLs. Render redirect previews as sanitized inert content. Do not execute scripts from fetched destinations.

### F-03 — Dynamic HTML construction (High / P1 review)

The static source scan recorded 11 `.innerHTML =` assignments, 2 `.outerHTML =` assignments, and 4 `insertAdjacentHTML` call sites. Counts are review leads, not proof that every occurrence is exploitable. Each source-to-sink flow must be traced.

**Remediation:** prefer `textContent` for text and DOM node construction for markup. Where authored HTML is required, use a maintained allow-list sanitizer. Validate URL schemes and destinations before assigning link attributes.

### F-04 — Obfuscated framework and dynamic-code patterns (High / P1 review)

The theme contains obfuscated framework code and string-decoder logic. A raw scan reported 54 Function-related constructor/call patterns, many likely associated with obfuscation; this is not a vulnerability count. The remote page contains the three nested `eval` paths described above.

**Remediation:** analyze readable upstream source or deobfuscate in a non-executing environment. Inventory dynamic code paths and external script sources. Do not execute unknown code in a logged-in browser or production origin.

### F-05 — Redirect destination trust (High / P1 review)

The redirect component reads a destination from query parameters and/or local storage and may navigate to it. The visible source does not demonstrate a complete explicit hostname allow-list.

**Remediation:** parse destinations with `URL`; allow only approved protocols and explicit hostnames; reject credentials, protocol-relative destinations, malformed values, and unapproved hosts. Show the destination before navigation and test encoded and nested values.

### F-06 — Inline handlers and `javascript:` URLs (Medium / P2)

The source scan found 11 inline event-handler attributes and 2 `javascript:` URL occurrences. These complicate restrictive Content Security Policy deployment and code review.

**Remediation:** use unobtrusive event listeners and semantic buttons/links. Do not add permissive `unsafe-inline` CSP solely to preserve legacy inline handlers.

### F-07 — Consent and advertising initialization (High / P1 verification)

The source declares Consent Mode defaults as denied for ad storage, analytics storage, ad user data, and ad personalization, with a 500 ms wait. The AdSense library is included in the document head and accept/decline handlers update the consent signals and store a single choice in local storage.

Source order alone does not establish that no cookies or storage are written before consent, that every relevant tag honors consent, that integrations are not duplicated, or that the implementation meets applicable legal requirements.

**Remediation:** test a clean browser profile across no-choice, accept, decline, reload, and withdrawal. Inspect network requests and browser storage. Verify consent initialization precedes all relevant Google tags, check regional requirements, and provide a clear withdrawal mechanism. Do not infer compliance from the presence of a banner.

### F-08 — Analytics identity and duplicate tags (Medium / P2 verification)

The theme includes Blogger's `google-analytics` include, but the static pattern scan did not identify a literal GA4 measurement ID. Blogger settings or other runtime injection may still configure Analytics.

**Remediation:** inspect the rendered test blog and Blogger settings; confirm one intended GA4 property and no duplicate Google tags.

## 3. Functional, accessibility, and SEO validation still required

- **Redirect and canonical URLs:** verify preview/staging behavior, custom-domain routing, query preservation, hostname parsing, and redirect loops.
- **Structured data:** source includes BlogPosting, MedicalWebPage, Physician, and WebSite JSON-LD. Validate rendered output on home, post, and static-page templates; check escaping, accuracy, and duplication with Blogger/plugin output.
- **Robots and indexing:** inspect rendered robots meta and canonical URLs on homepage, posts, pages, labels, archive, search, and error views. Avoid changing indexing policy based only on source inspection.
- **Performance:** inventory real runtime requests, fonts, third-party domains, scripts, and images; measure mobile/desktop performance on representative templates.
- **Accessibility:** test keyboard navigation, focus visibility, heading hierarchy, labels, contrast, image alternatives, and screen-reader behavior.
- **Blogger compatibility:** test widgets, comments, pagination, search, lazy loading, printing, redirects, and responsive layouts in a private test blog.

## 4. Repository governance and CI

- The canonical theme archive validation workflow is present and its latest relevant run #4 succeeded.
- CI validates archival integrity and XML well-formedness; it does not test browser runtime behavior or prove Blogger compatibility.
- The current `main` tree contains the archive, manifest, archive README, and validation workflow. It does not yet contain this audit report.
- Branch protection has not been independently confirmed as enabled. Require a pull request for `main`, block force pushes/deletion, and require observed successful status checks when supported. Avoid a one-approval rule if no independent reviewer is available.
- Keep GitHub Actions permissions least-privileged; use `contents: read` for validation-only jobs and pin third-party actions to reviewed immutable commit SHAs as a follow-up hardening task.
- Add non-mutating secret scanning, dependency/security checks where applicable, and workflow linting. Do not add deployment automation to Blogger without separate authorization.

## 5. Evidence limits and safe next steps

This is static source analysis based on the archived XML and a captured Blogger feed snapshot. It is not a browser runtime test, penetration test, malware verdict, legal opinion, or confirmation of the current live Blogger, AdSense, Analytics, or consent configuration. The captured feed snapshot may differ from the current endpoint and does not prove present ownership.

Recommended sequence:
1. Obtain a current raw export of the Database V9 Scripts page or the exact current Blogger feed response.
2. Compare the current content with the archived PDF snapshot; identify script changes, owners, external URLs, and all invoked functions.
3. Build a readable function inventory and dependency map. Confirm which features are still required.
4. Reimplement only required behavior without `eval`, Function constructors, HTML-string execution, or script extraction from fetched pages.
5. Store any candidate theme under a separate candidate path; never overwrite the immutable original.
6. Run strict XML checks, source scans, diff review, and archive-integrity checks.
7. Validate the candidate in a private Blogger test blog using a documented functional, privacy, accessibility, SEO, and performance matrix.
8. Keep the PR in draft until evidence is attached. Production upload, Blogger page edits, merge to production, and deployment require separate explicit approval.

**Production status:** unchanged. This report authorizes no production action.
