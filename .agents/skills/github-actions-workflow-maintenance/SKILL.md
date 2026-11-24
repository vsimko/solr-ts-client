---
name: github-actions-workflow-maintenance
description: Design, audit, and troubleshoot GitHub Actions workflows, with dependency caching as a required default.
---

# GitHub Actions workflow maintenance

Use this skill before creating, modifying, reviewing, or debugging files under `.github/workflows/`.

## Required context

1. Read `AGENTS.md` and follow repository-wide rules.
2. Inspect all related workflow files, reusable workflows, scripts, package manifests, lockfiles, and validation configuration.
3. Inspect recent workflow runs for the affected commit and compare successful and failed runs.
4. Preserve the repository's existing action pinning, permissions, concurrency, and release policies.

## Dependency caching is required

Every workflow that installs or downloads reusable build/test dependencies MUST use a sensible cache unless the dependency is tiny, ephemeral, already supplied by the runner, or caching is demonstrably unsafe. Treat caching as a default design requirement, not an optional later optimization.

1. Inventory dependency download/install steps: Node packages, Python packages, OS packages, Rust crates/toolchains, browsers, SDKs, emulator tooling, and build outputs.
2. Prefer established repository patterns and official setup-action caching where appropriate. Pin third-party actions to full commit SHAs.
3. Cache package-manager download stores, not installed system directories:
   - Node: npm/yarn/pnpm cache; prefer setup-node's built-in cache when lockfile detection is reliable.
   - Python: pip download/wheel cache (normally `~/.cache/pip`).
   - APT: archive cache including `partial`, never the mutable installed package database; continue running `apt-get update` and install normally.
   - Rust: Cargo registry/git and build output only with compatible keys and restore behavior.
   - Other ecosystems: official package-manager cache directories.
4. Include OS/runner image, relevant runtime/toolchain version, and dependency lockfiles/manifests in keys. For APT or workflows without lockfiles, include an explicit cache schema version and hash the workflow/package list that defines installed dependencies.
5. Use restore prefixes only for compatible caches. Include architecture, runtime, or matrix dimensions when needed to prevent incompatible reuse.
6. Keep paths narrow and deterministic. Never cache credentials, tokens, secrets, private/licensed assets, emulator disk images, generated test results, or mutable machine state.
7. A cold cache, cache miss, or eviction must not affect correctness. Installs must still fetch and validate everything required.
8. Avoid pointless caches (one-off downloads or runner-preinstalled tools); document justified exceptions. Never cache an entire home or system directory.

## Workflow audit

- Verify event triggers and path filters include every relevant source, script, manifest, and workflow.
- Check permissions follow least privilege; do not expose secrets to untrusted fork pull requests.
- Pin third-party actions to full commit SHAs.
- Use explicit timeouts and concurrency groups where duplicate runs can race or waste resources.
- Upload useful failure diagnostics with `if: always()`, while excluding secrets and licensed payloads.
- Keep workflow summaries precise about skipped jobs, missing credentials, unavailable assets, and unverified behavior.
- For reusable and matrix workflows, check that cache keys do not cause incompatible jobs to share state.

## Run diagnosis

1. Resolve the exact run, commit SHA, attempt, job, and failed step.
2. Read the complete job log and inspect artifacts from that same run.
3. Separate infrastructure/setup, harness, test-program, and assertion failures.
4. Compare against the latest successful run for the same workflow and target.
5. Make the smallest safe correction and rerun the affected workflow.
6. Do not report a queued or running workflow as validated.

## Validation

- Run the repository's workflow linter (for example, actionlint) against every workflow file.
- Run shell syntax checks for affected scripts and relevant repository validators.
- Validate that cache keys change when dependencies/toolchains change and that the workflow works with an empty cache.
- Inspect a completed CI run after pushing; report exact commands and run status.
- Mention cache coverage and any justified exceptions in the completion report.
