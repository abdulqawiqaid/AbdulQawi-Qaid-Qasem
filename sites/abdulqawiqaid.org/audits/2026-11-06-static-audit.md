# Static security and technical audit — Blogger theme

**Date:** 2026-11-06  
**Repository:** `abdulqawiqaid/AbdulQawi-Qaid-Qasem`  
**Branch:** `feature/blogger-site-integration`  
**Source:** `blogger-theme/original/blogger-theme-original.xml`  
**SHA-256:** `e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4`

## Executive result

The original XML is well-formed under strict XML parsing. Static review plus the user's archived Blogger feed response (blogger.PDF) identifies a high-risk remote-code chain: the theme retrieves the Blogger page titled Database V9 Scripts, extracts its first script element, stores the text in sessionStorage, and executes it with eval; lazy-loading paths execute the stored text again. The captured page content contains three further eval call sites, including code extracted from Blogger widget output and scripts extracted from fetched redirect HTML. This is not proof of compromise, but it creates multiple remote code execution surfaces in the blog page's origin. The PDF is a captured snapshot (page metadata says updated 2025-11-30), not proof of the endpoint's current content.

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

### F-01 — Theme executes a remote Blogger page as JavaScript (Critical / P0)

The theme's callback sp_db loads this Blogger JSON-in-script endpoint:

feeds/5637748011392113640/pages/default/137042360492087078?alt=json-in-script&callback=sp_db

The archived response in blogger.PDF identifies the entry as Database V9 Scripts, page ID 137042360492087078, blog ID 5637748011392113640, with an alternate URL on seoplusdatabase8.blogspot.com. The captured Atom entry reports author Unknown and an update timestamp of 2025-11-30. This snapshot does not establish who currently controls the page or whether the live endpoint still serves identical content.

The theme callback:

1. Checks whether the Blogger entry ID contains a decoded numeric substring corresponding to the expected page identifier. This is an identifier check, not an authenticity or integrity check.
2. Parses entry.content.$t as HTML with DOMParser.
3. Selects the first script element and reads its innerHTML.
4. Stores the script text under the storg key in sessionStorage and immediately evaluates it.
5. The LazMin and lazyfunction paths read and evaluate the same stored value again.

**Impact:** code supplied by that Blogger page runs with the privileges of the visitor's blog page. Anyone able to alter the page or its delivery can potentially alter code executed for site visitors. The use of sessionStorage does not create a trust boundary and extends the lifetime of the code within the tab session.

**Recommendation:** eliminate the remote HTML-to-script-to-eval mechanism. Move only reviewed, required functionality into a readable, versioned local asset or directly into the reviewed theme. Use a strict JSON schema for configuration. Remove the storg storage/replay path. Do not simply replace eval with dynamically created script elements; that still executes the same untrusted source.


### F-02 — Nested dynamic execution inside Database V9 Scripts (Critical / P0)

The captured Database V9 Scripts page source in blogger.PDF contains three direct eval call sites in addition to the theme's three call sites:

- bloggerJS() extracts script text from Blogger widget-related output and evaluates the extracted text; it also evaluates another script fragment after loading the Blogger widgets.js resource.
- The redirect-content path fetches a destination page, parses its HTML, extracts its script elements, and evaluates the extracted script collection (eval(rScripts)).

This means the theme's remote-code page is itself a loader for further executable content. The redirect path deserves special attention because the theme also contains query/local-storage based redirect handling. The captured source alone does not fully prove whether an arbitrary visitor can control the final fetched destination in every runtime configuration; that source-to-sink relationship must be verified before classifying it as an exploitable open redirect or script injection.

**Recommendation:** remove regex extraction and evaluation of scripts from Blogger widget output. Use documented Blogger widget resources through normal script URLs where needed. For redirect previews, render only reviewed/sanitized content and never execute scripts from the fetched page. Validate redirect destinations against an explicit host allow-list, reject non-HTTP(S) schemes and credentials, and do not persist arbitrary destinations as trusted values.

### F-03 — Dynamic HTML construction (High / P1 review)

Counts from source scan: 11 `.innerHTML =` assignments, 2 `.outerHTML =` assignments, and 4 `insertAdjacentHTML` call sites. Some strings are assembled from post content, labels, widget data, or redirect-related values. Not every sink is automatically exploitable; each source-to-sink path requires taint review.

**Recommendation:** use `textContent` for text, DOM node construction for markup, and a maintained allow-list sanitizer only where authored HTML is intentionally supported. Validate URL schemes and destinations before assigning link attributes.

### F-04 — Obfuscated framework and dynamic code constructors (High / P1)

The main framework uses several obfuscation layers and string-decoder functions. A raw scan reports 54 `Function`-related constructor/call patterns, many likely part of obfuscation machinery; this count is not a vulnerability count. Three direct `eval` call sites were identified.

**Recommendation:** obtain a readable upstream source or deobfuscate the framework in a non-executing analysis environment; document every dynamic-code path and external script source. Do not execute the unknown code in a logged-in browser or production origin during analysis.

### F-05 — Redirect destination trust (High / P1 review)

The redirect component reads a destination from a query parameter and/or local storage, then assigns it to a navigation target when automatic redirect is enabled. The visible static code does not establish a complete origin allow-list.

**Recommendation:** parse with `URL`, permit only explicitly approved protocols and hostnames, reject credentials and protocol-relative forms, and show the destination before navigation. Test nested, encoded, malformed, and cross-origin values.

### F-06 — Inline handlers and javascript URLs (Medium / P2)

The source contains 11 inline event-handler attributes and 2 `javascript:` URL occurrences. These complicate a restrictive Content Security Policy and make behavior harder to audit.

**Recommendation:** replace with unobtrusive event listeners and semantic buttons/links. Do not introduce a permissive `unsafe-inline` CSP merely to accommodate the legacy template.

### F-07 — Consent and advertising load order (High / P1 verification)

The template declares Consent Mode defaults as denied for ad storage, analytics storage, ad user data, and ad personalization, with a 500 ms wait. It loads the AdSense library in the document head and includes accept/decline handlers that update all four signals and persist a single choice in local storage.

**What is established:** the default declaration precedes the visible AdSense script include in the source; the library is nevertheless requested before the visitor makes a choice. The handlers implement all-or-nothing accept/decline rather than granular categories.

**What is not established:** whether cookies/storage are actually written before consent, whether all tags honor the consent state, whether a separate Blogger/Google Analytics integration duplicates tags, and whether the implementation meets applicable legal requirements.

**Recommendation:** test with a clean browser profile and network/storage inspection for no-choice, accept, decline, reload, and withdrawal paths. Verify consent initialization precedes every relevant Google tag. Confirm the correct regional requirements and provide a clear withdrawal mechanism. Do not assume a consent banner alone establishes compliance.

### F-08 — Analytics identity and duplicate integration (Medium / P2 verification)

The theme includes Blogger's `google-analytics` include, but the static source scan did not identify a literal GA4 measurement ID. This does not establish whether Analytics is configured in Blogger settings or injected elsewhere.

**Recommendation:** inspect the rendered page and Blogger settings in a test environment; ensure one intended GA4 property and no duplicate Google tags.

## 3. Functional and SEO review items

- Canonical redirect: verify preview/staging behavior, custom-domain routing, query preservation, and loops. The static redirect checks for a `blogspot.com` hostname substring; hostname parsing and an explicit host allow-list would be safer.
- Structured data: source includes BlogPosting, MedicalWebPage, Physician, and WebSite JSON-LD. Validate rendered JSON-LD on home, post, and page templates; confirm values are escaped and not duplicated by Blogger or plugins.
- Robots: conditional directives exist for label, error, and single-item views. Verify rendered robots meta and canonical URL across all page types before changing indexing policy.
- External origins found include Google/Blogger infrastructure, Google Fonts, Google Charts, AdSense, social networks, and third-party theme/social domains. Inventory actual runtime requests and remove unused origins.
- Static inspection cannot confirm the live site is serving this exact archive.

## 4. CI and repository controls

- Added `.github/workflows/validate-blogger-theme.yml` to parse the archived XML and verify its SHA-256 on pull requests and feature-branch pushes.
- GitHub Actions run `42230026224` completed successfully; its XML/checksum validation job concluded `success`.
- The `main` branch metadata reports `protected: false` and protection `enabled: false`. A separate branch-protection read returned `403 Resource not accessible by integration`.
- The available GitHub connector exposes branch-protection reads but no branch-protection mutation action. Therefore, although protection was requested, I could not enable it from this session. This is an outstanding repository safeguard, not a completed action.
- The pull request remains open and draft. No merge has occurred.

**Next repository step:** enable main-branch protection in GitHub repository settings: require pull requests, require at least one approval (or the strongest available review rule), require the successful 'Validate original Blogger XML and checksum' check, dismiss stale approvals, and block force pushes/deletions. The current connector cannot change this setting. Extend CI later with non-mutating security and repository-hygiene scans; never auto-rewrite the original theme.

## 5. Scope, limitations, and next actions

This is static source analysis using the archived theme and a captured Blogger feed response, not a browser runtime test, penetration test, malware verdict, legal opinion, or verification of the current live Blogger/AdSense/Analytics configuration. The captured feed response may differ from the current endpoint. Obfuscated behavior remains partially unresolved.

Recommended order:

1. Obtain the current raw export of the Database V9 Scripts page and compare it with the archived blogger.PDF snapshot.
2. Inventory every function and dependency supplied by that page; rewrite required behavior as reviewed readable code without eval.
3. Remove the theme's remote HTML-to-script-to-eval and sessionStorage replay chain.
4. Remove nested evaluation of Blogger widget output and redirect-page scripts; validate redirect destinations.
5. Trace each HTML sink and apply minimal safe replacements.
6. Verify consent, advertising, comments, widgets, pagination, and lazy loading in a private test blog.
7. Request separate approval before any production theme upload or Blogger page change.

**Production status:** unchanged. No production action is authorized by this audit report.
