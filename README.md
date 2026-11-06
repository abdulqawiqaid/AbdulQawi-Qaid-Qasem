# abdulqawiqaid.org — Blogger Website Repository

Documentation and a verified archive of the Blogger theme associated with [abdulqawiqaid.org](https://www.abdulqawiqaid.org/).

> **Important:** This repository is not documented as an automatic deployment pipeline. A successful GitHub check does not publish changes to Blogger or prove the live website is secure or functional.

## Repository layout

| Path | Purpose |
| --- | --- |
| [Canonical Blogger XML](theme-3351920114810663100%20%282%29%20%2815%29.xml) | Single authoritative archived theme file at the repository root |
| [Theme archive guide](sites/abdulqawiqaid.org/blogger-theme/README.md) | Identity, verification and recovery guidance |
| [Theme manifest](sites/abdulqawiqaid.org/blogger-theme/manifest.json) | Expected size and cryptographic fingerprints |
| [Validation workflow](.github/workflows/validate-blogger-theme.yml) | Static verification in GitHub Actions |
| [Security audit](sites/abdulqawiqaid.org/audits/2026-11-06-static-audit.md) | Dated static findings, not a live-site security certification |
| [Remediation plan](sites/abdulqawiqaid.org/audits/2026-11-06-remote-code-remediation-plan.md) | Proposed staged treatment of remote script execution |

## Theme integrity and CI

The XML archive is kept **once**, at the repository root. The manifest identifies it by exact byte length (391,267), SHA-256 and Git blob SHA-1. The [validation workflow](.github/workflows/validate-blogger-theme.yml) checks those values and XML well-formedness.

The workflow is triggered by pull requests to `main` that change the theme, theme-archive directory or validation workflow; it can also be started manually with `workflow_dispatch`. **Documentation-only changes to this root README do not automatically trigger this workflow.**

Static validation does **not** test Blogger rendering, JavaScript execution, responsive behavior, accessibility, performance, SEO, AdSense, analytics or consent behavior.

## Security and change control

The dated [static audit](sites/abdulqawiqaid.org/audits/2026-11-06-static-audit.md) documents concerns about dynamically loaded and executed scripts. Treat this as a remediation priority, not evidence of compromise. Review the [remediation plan](sites/abdulqawiqaid.org/audits/2026-11-06-remote-code-remediation-plan.md) before making script-loading changes.

Use a separate branch and pull request for proposed changes. Review diffs, verify checks and test any changed theme in a non-production Blogger site before considering production deployment. Do not upload, merge or publish a modified theme without explicit authorization.

## SEO, advertising and privacy

The presence of structured-data markup, canonical tags, robots directives, AdSense code or consent defaults in source files does not establish their correctness on the live site. Validate rendered pages, indexing directives, advertising eligibility, analytics duplication and consent behavior independently in a browser and the relevant provider dashboards.

Do not commit credentials, access tokens, unpublished personal information or private configuration.

## Safe test and recovery outline

1. Preserve a separate backup of the currently deployed Blogger theme before any experiment.
2. Review the proposed XML diff and verify archive integrity.
3. Import a candidate theme **only into an authorized non-production test blog** and test desktop/mobile views, navigation, scripts, structured data and consent.
4. Record observed results and obtain explicit approval before any production action.
5. If a test fails, restore the test blog from its own known-good backup. The archived XML is a historical reference and is **not guaranteed** to match the currently deployed theme.

For detailed archive identification and verification, see the [theme archive guide](sites/abdulqawiqaid.org/blogger-theme/README.md).

## Scope and licensing

This repository documents a Blogger site and its theme archive; it does not claim that all live Blogger content, settings, integrations or assets are version-controlled here. No license grant is asserted by this README; consult repository licensing information and applicable rights before reuse.
