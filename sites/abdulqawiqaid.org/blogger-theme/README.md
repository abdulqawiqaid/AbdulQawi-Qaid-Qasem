# Blogger theme archive

This directory documents the **single canonical Blogger theme XML** stored at the repository root. It does not contain a second editable copy.

## Canonical source

- [Root-level XML archive](../../../theme-3351920114810663100%20%282%29%20%2815%29.xml)
- Repository path: `theme-3351920114810663100 (2) (15).xml`
- Original file size: **391,267 bytes**
- SHA-256: `e7d8034ad7af3bbdfced737ba75b3bd3ad2efa5f457326fa05d9e861455b4da4`
- Git blob SHA-1: `6826a09292ac3882efde09f63de0d06d1257f0c6`
- Machine-readable reference: [manifest.json](manifest.json)

Do not overwrite the archive with a newly exported, reformatted or downloaded Blogger theme without a separately reviewed change and a deliberate manifest update.

## Verification

The [GitHub Actions workflow](../../../.github/workflows/validate-blogger-theme.yml) reads `manifest.json` and verifies:

1. Exact byte length.
2. SHA-256 digest.
3. Git blob SHA-1.
4. XML well-formedness.

The workflow runs on pull requests targeting `main` when the XML, this archive directory or the workflow changes; manual dispatch is also supported. A passing check only verifies static archive properties, **not** runtime compatibility or live-site safety.

## Testing and recovery

1. Back up the **currently deployed** Blogger theme separately before any changes.
2. Review candidate modifications in a feature branch and pull request.
3. Test an authorized candidate in a **non-production** Blogger blog. Check rendering, mobile behavior, navigation, JavaScript, redirects, SEO metadata, structured data, accessibility, analytics and consent.
4. If the test fails, restore that test blog using its own known-good backup; do not assume this archive equals the live theme.
5. Do not merge or publish to the production blog without explicit authorization.

## Change tracking

Use Git history and pull-request discussions for a record of archive changes. Update `manifest.json` only when an intentionally approved change alters the canonical XML bytes. A documentation-only update does not change the canonical theme identity.

See the [repository overview](../../../README.md) and [dated security audit](../audits/2026-11-06-static-audit.md) for wider context.
