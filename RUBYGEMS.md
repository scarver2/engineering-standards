# RubyGem Engineering and Release Standard

Scope: public open-source RubyGems primarily maintained under `scarver2/*`. Reuse these practices in other organizations where appropriate. This is a public, secrets-free engineering policy; do not copy private operating instructions, credentials, tenant identities, account IDs, token material, session values, or internal infrastructure details here.

## Principle

Do not rediscover RubyGem packaging, CI, or publication on each project. Start from accepted existing implementations (notably `scarver2/abbu`, `scarver2/printavo-ruby`, and `scarver2/5pm-ruby`), verify current upstream compatibility, and change shared practices through reviewed improvements. Prefer Bundler, RubyGems, and official GitHub Actions over custom frameworks.

## Project creation checklist

1. Check existing gem name/ownership and RubyGems naming conventions. Gem package name, Ruby constant namespace and `require` path may differ; document each.
2. Start with an MIT (or deliberately selected) license, gemspec, version file, README, CHANGELOG, CONTRIBUTING, security policy, and examples.
3. Use a minimum-supported Ruby version justified by dependencies. Maintain a current supported Ruby CI matrix; do not assume a single local version proves compatibility.
4. Use RSpec, SimpleCov, RuboCop, RBS where useful, and package/isolated-install smoke tests. Keep tests deterministic and scrub all fixtures.
5. Standardize `bin/spec`, `bin/lint`, `bin/package`, and `bin/release-check` entrypoints; shared Bash helper may use mise but must work without it. Inspect actual built artifact inventory and runtime load, not just source-tree tests.
6. Use PRs, independent review, exact-head SHA acceptance, green CI, and normal branch protection. No self-approval or bypass. No deployment implied by PR approval.
7. Optional adapters (e.g. MCP) must not burden core SDK consumers. If separate gems are used, verify package naming, dependency versions, and publication order.

## First publication is different from later releases

Before building a release workflow, verify the current official RubyGems trusted-publishing documentation and *action implementation* for the exact first-publication flow. Do not assume an action supports an input just because an API or UI does. In particular, confirm the supported owner-selection mechanism if a pending trusted publisher requires it. Pin third-party actions to verified immutable SHAs and test the declared input schema.

Prefer OIDC trusted publishing with a protected GitHub release environment and no persistent RubyGems API key in repository secrets. A first-publication incompatibility is a bounded blocker: investigate official support; if unavailable, prepare a **maintainer-only local manual first publication** rather than inventing a credential exchanger. A manual path replaces authentication only; it never waives source, artifact, review, or qualification gates.

For subsequent versions, prefer the approved automated OIDC path after verifying publisher binding. Pending publisher registrations can expire; recheck them immediately before release. Never hurry a release solely to beat an expiration.

## Release gates

- Identify the exact accepted commit SHA, version, dated changelog, dependency locks, and reproducible package artifacts with SHA-256 digests.
- Run supported Ruby matrix, tests, lint, type checks if applicable, packaging, isolated installs, and relevant live integration qualification.
- Separate **historical development evidence** from **current release evidence**. Do not turn failed or dirty-worktree historical reports into a PASS by hashing them or changing a checker.
- If the SDK has live write operations, use a verified disposable environment and least-privilege test identity; authorize mutations explicitly, label created records, stop on failure, and independently verify cleanup. Document unsupported or privileged operations honestly. A discovered cleanup failure is a blocker until disposed by the maintainer under an explicit reviewed release policy.
- Review exact-source evidence and publishable artifacts. Never silently weaken gates to meet schedule.
- Tag only the exact accepted immutable candidate after owner approval. Keep tag-writing permissions separate from OIDC publication permissions where feasible.
- Require protected environment human review for publication. Environment variables are execution context, **not** authorization.
- Publish dependency/core gems before dependent adapter gems, verifying each is available. Never overwrite existing published versions or move tags. For partial releases, stop and reconcile published state before retrying.
- No release tags, dispatches, or gem pushes without separate explicit maintainer authorization.

## Manual first-publication handoff

The engineering agent prepares a private owner-facing runbook: exact candidate SHA, artifact names/versions and SHA-256, verified build/install commands, current RubyGems name state, any unresolved gates, and the order of owner-only `gem push` commands. The owner authenticates locally and personally performs the push only after release acceptance. Never request or paste passwords, MFA codes, RubyGems API keys or recovery codes into prompts, tickets, CI logs, source, or pull requests. Do not run `gem push` from an agent or with shared administrator credentials.

## Agent workflow and lessons learned

Before creating a new RubyGem, agents must read this standard and inspect at least one accepted existing RubyGem. Use a common reusable agent skill/checklist; update it when the release process changes. Record reusable lessons here, not only in a project-specific issue. Escalate uncertainty to the maintainer or Deputy; continue other authorized work while blocked. Do not add orchestration/tooling experiments to a release-critical path.

Measure time to first green test, time to accepted release candidate, human interventions, CI rework, and agent cost when available. Optimize for reproducible quality and fewer owner decisions, not just code generation speed.

## Maintenance

Treat this as the normative **public** baseline, not proof that every historical repository already complies. Before any release, verify live provider behavior and the repository's actual policies. Propose shared fixes in a reviewable PR, then apply them to other gems in bounded follow-ups. Avoid publishing private security architecture or secrets in examples or documentation.
