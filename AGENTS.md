# AGENTS.md

These instructions apply to AI-assisted work in this repository.

## Authority

Read, in order:

1. `GOVERNANCE.md`
2. accepted standards in `standards/`
3. accepted ADRs in `adrs/`
4. applicable `skills/*/SKILL.md`
5. templates in `templates/`

Repository-local policy in an adopting product may add product-specific gates. If this repository and an adopting repository conflict, report `POLICY CONFLICT:` and do not silently choose one.

## Agent behavior

- Do not infer product behavior, release gates, ownership, credentials, signing identities, store policy, pricing, branding, legal authority, or public commitments.
- Report missing product decisions as `AMBIGUITY:`.
- Do not introduce hard-coded user-visible or accessibility-visible strings in adopting products.
- Preserve native platform localization and accessibility mechanisms.
- Treat shared standards as semantic policy, not as a mandate for a single shared runtime, framework, or file format.
- Never commit secrets, customer data, credentials, certificates, private keys, provisioning profiles, store credentials, or proprietary editable marketing masters.

## Local project layout

Use `~/Projects/<owner>/<repo>` for local checkouts and preserve the repository’s existing worktree layout. For repositories owned by `scarver2`, use `~/Projects/scarver2/<repo>` in place of `~/Projects/OSS/<repo>`. For example, use `~/Projects/scarver2/howdy-world` for `scarver2/howdy-world`.
