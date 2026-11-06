# Blogger theme — original archive and static review

## Source provenance
- Source filename received: `theme-3351920114810663100 (2) (15).xml`
- Archived filename: `blogger-theme-original.xml`
- Size: 391,267 bytes
- Lines: 3,323
- SHA-256: `e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4`
- XML parser check: parsed successfully after removing the XHTML doctype declaration; no recovery errors were reported.
- Status: archived unchanged; not deployed.

## Initial technical findings
These are static observations, not a full browser, Blogger, or penetration test.

| Area | Observation | Initial assessment |
|---|---|---|
| Blogger structure | Blogger template namespaces, widgets, includables, and skin variables are present. | Expected Blogger theme structure. |
| Redirect | A script redirects any hostname containing `blogspot.com` to the custom domain and carries path and query string. | Verify preview, staging, canonical, and exceptional paths before use. |
| AdSense | Loads `adsbygoogle.js` for client `ca-pub-7823117555703776`. | Verify active account and avoid duplicate Blogger-native integration. |
| Consent default | Consent Mode defaults ad/analytics storage and related signals to `denied`. | Test end-to-end; default declaration alone does not prove blocking. |
| Consent update | Accept/decline handlers update consent signals and store a value in `localStorage`. | Review timing, category mapping, and whether tags remain blocked until consent. |
| `eval` | Three `eval(...)` call sites in minified/obfuscated JavaScript. | High-priority review. Trace source and execution path before release. |
| Dynamic HTML | Eleven `.innerHTML =` assignments. | Trace data provenance; use safe DOM construction/sanitization for untrusted data. |
| Obfuscated code | Large obfuscated script in the theme framework. | Deobfuscate and review before approval; behavior not fully assessed. |
| Inline event handlers | Eleven inline event-handler attributes. | Consider CSP compatibility and controlled event listeners. |
| `javascript:` links | Two occurrences, including print and redirect-related anchors. | Replace with buttons or ordinary links and listeners where feasible. |
| Structured data | `BlogPosting`, `MedicalWebPage`, `Physician`, and `WebSite` JSON-LD blocks. | Validate rendered output; avoid conflicting/duplicated page types. |
| Robots directives | Conditional robots declarations for search labels, errors, and single-item pages. | Inspect rendered output; ensure canonical pages are not accidentally noindexed. |
| Analytics | Includes Blogger's `google-analytics` include; no literal GA4 ID found by simple pattern scan. | Confirm Blogger settings and rendered network requests. |
| External dependencies | Google Fonts, AdSense, Blogger image infrastructure, and Google Charts. | Inventory versions, conditions, privacy implications, and availability. |

## Prioritized remediation
1. Preserve this original unchanged; map scripts, widgets, hosts, and dependencies.
2. Deobfuscate the theme framework and explain every `eval`, storage read, dynamic script load, redirect, and network request.
3. Trace all `innerHTML` sources and replace unsafe construction for untrusted input.
4. Test Consent Mode defaults before all Google tags; map choices accurately and verify no duplicate AdSense integration.
5. Test canonical, robots, title, description, Open Graph, schema, and pagination on homepage, post, page, label, archive, search, and error views.
6. Import only into a private test blog; inspect widgets, mobile behavior, rendering, console, and network output.
7. Release only after review and explicit approval. No production theme upload or live configuration change is authorized by this archive.

## What this review does not establish
- Active AdSense status, approval, policy compliance, or ad serving.
- Actual Blogger settings for Analytics, AdSense, consent, or custom-domain redirects.
- Whether every obfuscated function is malicious or benign.
- Whether the live site currently serves this exact uploaded template.
- Rendered schema or indexing behavior.

## Source integrity
```bash
sha256sum original/blogger-theme-original.xml
```
Expected SHA-256:
```text
e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4
```
