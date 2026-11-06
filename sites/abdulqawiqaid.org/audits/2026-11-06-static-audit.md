# Static security and technical audit — Blogger theme

**Date:** 2026-11-06  
**Repository:** `abdulqawiqaid/AbdulQawi-Qaid-Qasem`  
**Branch:** `feature/blogger-site-integration`  
**Source:** `blogger-theme/original/blogger-theme-original.xml`  
**SHA-256:** `e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4`

## Executive result

The original XML is well-formed under strict XML parsing. Static review found a high-risk executable-content path: code is retrieved from a Blogger feed entry, extracted from its HTML, persisted in `sessionStorage`, and passed to `eval`. The same storage value is later evaluated again by the lazy-loading path. This is not proof of compromise, but it creates a remote code execution surface within the visitor's page origin if the designated post content or the Blogger account/editorial pipeline is compromised or can be modified by an untrusted party. This path must be understood and removed or constrained before production approval.

No code remediation was applied in this audit phase. The original theme remains byte-for-byte archived. No merge, deployment, Blogger setting, domain, DNS, AdSense, Analytics, or consent setting was changed.

## 1. Integrity and parser checks

| Check | Result |
|---|---|
| File size | 391,267 bytes |
| SHA-256 | `e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4` |
| Strict XML parse | PASS; `lxml` recovery disabled; original doctype retained |
| XML elements | 2,549 |
| LF newline count | 3,322 |

## 2. Security findings

### F-01 — Blogger feed content reaches eval (Critical / P0)

Static evidence in the obfuscated framework:

1. A Blogger JSON feed callback named `sp_db` receives an entry object.
2. It checks the entry ID against an encoded identifier.
3. It parses the entry's HTML content with `DOMParser`.
4. It selects a script element and reads its text content.
5. It stores that text under the `storg` key in `sessionStorage` and calls `eval` on it.
6. The `LazMin` / lazy-load path later reads the same key and evaluates the stored text again.

**Impact:** executable content supplied through the designated Blogger entry is executed in the blog page's JavaScript context. If the source post, Blogger account, or an editor capable of changing that post is compromised, the attacker may gain script execution in visitors' sessions on the site. The encoded ID check is not a cryptographic trust boundary.

**Recommendation:** remove the executable-script extraction/evaluation feature. If the theme needs remote configuration, return structured JSON with a strict schema and allow-listed values; never interpret returned strings as JavaScript. Clear legacy `storg` values after migration. If the feature is essential, isolate it on a separately controlled origin and require signed, versioned artifacts with a reviewed deployment process; a signature must not be treated as a substitute for access control.

### F-02 — Dynamic HTML construction (High / P1 review)

Counts from source scan: 11 `.innerHTML =` assignments, 2 `.outerHTML =` assignments, and 4 `insertAdjacentHTML` call sites. Some strings are assembled from post content, labels, widget data, or redirect-related values. Not every sink is automatically exploitable; each source-to-sink path requires taint review.

**Recommendation:** use `textContent` for text, DOM node construction for markup, and a maintained allow-list sanitizer only where authored HTML is intentionally supported. Validate URL schemes and destinations before assigning link attributes.

### F-03 — Obfuscated framework and dynamic code constructors (High / P1)

The main framework uses several obfuscation layers and string-decoder functions. A raw scan reports 54 `Function`-related constructor/call patterns, many likely part of obfuscation machinery; this count is not a vulnerability count. Three direct `eval` call sites were identified.

**Recommendation:** obtain a readable upstream source or deobfuscate the framework in a non-executing analysis environment; document every dynamic-code path and external script source. Do not execute the unknown code in a logged-in browser or production origin during analysis.

### F-04 — Redirect destination trust (High / P1 review)

The redirect component reads a destination from a query parameter and/or local storage, then assigns it to a navigation target when automatic redirect is enabled. The visible static code does not establish a complete origin allow-list.

**Recommendation:** parse with `URL`, permit only explicitly approved protocols and hostnames, reject credentials and protocol-relative forms, and show the destination before navigation. Test nested, encoded, malformed, and cross-origin values.

### F-05 — Inline handlers and javascript URLs (Medium / P2)

The source contains 11 inline event-handler attributes and 2 `javascript:` URL occurrences. These complicate a restrictive Content Security Policy and make behavior harder to audit.

**Recommendation:** replace with unobtrusive event listeners and semantic buttons/links. Do not introduce a permissive `unsafe-inline` CSP merely to accommodate the legacy template.

### F-06 — Consent and advertising load order (High / P1 verification)

The template declares Consent Mode defaults as denied for ad storage, analytics storage, ad user data, and ad personalization, with a 500 ms wait. It loads the AdSense library in the document head and includes accept/decline handlers that update all four signals and persist a single choice in local storage.

**What is established:** the default declaration precedes the visible AdSense script include in the source; the library is nevertheless requested before the visitor makes a choice. The handlers implement all-or-nothing accept/decline rather than granular categories.

**What is not established:** whether cookies/storage are actually written before consent, whether all tags honor the consent state, whether a separate Blogger/Google Analytics integration duplicates tags, and whether the implementation meets applicable legal requirements.

**Recommendation:** test with a clean browser profile and network/storage inspection for no-choice, accept, decline, reload, and withdrawal paths. Verify consent initialization precedes every relevant Google tag. Confirm the correct regional requirements and provide a clear withdrawal mechanism. Do not assume a consent banner alone establishes compliance.

### F-07 — Analytics identity and duplicate integration (Medium / P2 verification)

The theme includes Blogger's `google-analytics` include, but the static source scan did not identify a literal GA4 measurement ID. This does not establish whether Analytics is configured in Blogger settings or injected elsewhere.

**Recommendation:** inspect the rendered page and Blogger settings in a test environment; ensure one intended GA4 property and no duplicate Google tags.

## 3. Functional and SEO review items

- Canonical redirect: verify preview/staging behavior, custom-domain routing, query preservation, and loops. The static redirect checks for a `blogspot.com` hostname substring; hostname parsing and an explicit host allow-list would be safer.
- Structured data: source includes BlogPosting, MedicalWebPage, Physician, and WebSite JSON-LD. Validate rendered JSON-LD on home, post, and page templates; confirm values are escaped and not duplicated by Blogger or plugins.
- Robots: conditional directives exist for label, error, and single-item views. Verify rendered robots meta and canonical URL across all page types before changing indexing policy.
- External origins found include Google/Blogger infrastructure, Google Fonts, Google Charts, AdSense, social networks, and third-party theme/social domains. Inventory actual runtime requests and remove unused origins.
- Static inspection cannot confirm the live site is serving this exact archive.

## 4. CI and repository controls

- The feature branch's `.github` directory contains only `pull_request_template.md`; no Actions workflow file was found.
- The reviewed commit has no associated workflow runs and no reported commit statuses. CI is therefore **not configured/evidenced**, not passing.
- Branch-protection settings could not be read: GitHub returned `403 Resource not accessible by integration`. This is an access limitation, not proof that protection is absent.
- The pull request remains open and draft. No merge has occurred.

**Recommended next repository step:** add a CI workflow on the feature branch to validate XML well-formedness, detect accidental changes to the immutable original archive, scan for unsafe sinks and external hosts, and run repository hygiene checks. CI should report findings without auto-rewriting the original theme.

## 5. Scope, limitations, and next actions

This is static source analysis, not a browser runtime test, penetration test, malware verdict, legal opinion, or verification of live Blogger/AdSense/Analytics configuration. Obfuscated behavior remains partially unresolved.

Recommended order:

1. Resolve F-01 and map the full Blogger feed / script-loading chain.
2. Resolve F-04 redirect validation and audit all external destinations.
3. Trace each HTML sink in F-02 to its source and apply minimal safe replacements.
4. Verify consent and advertising behavior in a private test blog with clean browser profiles.
5. Add CI and verify branch protection independently with an authorized GitHub account.
6. Run a functional compatibility matrix on a private Blogger test blog before requesting any production approval.

**Production status:** unchanged. No production action is authorized by this audit report.
