# Contributing to InovaCC
<!-- rev:001 (RFC 3339) 2026-10-17T22:40:00Z -->

These rules apply to every repository of the `inovacc-core` organization and to every contributor: people and AI
agents alike. A repository's own `CONTRIBUTING.md` adds its build and test commands and may make a rule stricter, never
looser. Read [docs/DOCUMENTATION-STANDARD.md](docs/DOCUMENTATION-STANDARD.md) before you add or change a unit.

## 1. How a change reaches `main`

**Nothing is pushed to `main` directly.** Every change goes through a pull request and is **squash-merged**.

1. **Branch** from the current `main`, one branch per change, short-lived:
   `<type>/<scope>-<short-description>`, for example `fix/data-bound-credentials`, `feat/ai-openapi`,
   `docs/identity-runbooks`. Types are the conventional-commit types below. No long-lived personal or agent branches,
   no extra worktrees.
2. **Commit** as you like on the branch; the history of the branch is squashed on merge.
3. **Open a pull request** with the template filled in completely (section 3). A draft PR is fine while you work.
4. **CI must pass** on the PR: the repository's gates (section 4). A red or skipped required check blocks the merge.
5. **Review:** at least one approval from a code owner (`CODEOWNERS`). The author does not approve their own PR. An AI
   agent's PR is reviewed by the operator or by an independent reviewer; the agent never approves.
6. **Squash merge**, with a conventional-commit title (section 2) and a body that keeps the PR's summary and evidence.
   No merge commits, no rebase merges.
7. **Delete the branch** on merge.

A release is a separate act: a published GitHub Release tagged `cf-<worker>/vX.Y.Z` (or the repository's own tag
pattern), created only on the operator's go for that batch. Merging is never releasing.

## 2. Commits and PR titles

- **Conventional commits** with a scope: `feat(data): ...`, `fix(iam): ...`, `docs(ai): ...`, `refactor(forge): ...`,
  `test(...)`, `build(...)`, `ci(...)`, `chore(...)`, `perf(...)`, `revert(...)`. A breaking change says so with `!`
  (`feat(api)!:`) and a `BREAKING CHANGE:` footer.
- English, imperative, specific: what changes and why, not "update files".
- **No AI attribution** of any kind (no `Co-Authored-By` for a model, no "Generated with" lines).
- Never `--no-verify`, never skip or forge signing, never rewrite `main`'s history.

## 3. What a pull request must contain

The template asks for each item; a PR missing one is not reviewed.

- **What and why**, in a few sentences, and the issue, bug or decision it answers.
- **The developer surface (rule 50):** if the change touches anything a developer sees or calls, the chain:
  `CONTRACT.md` and `openapi*.yaml` updated together and versioned, SDKs, developer portal, the
  `SURFACE-CHANGELOG.md` row (see `inovacc-core/developer/docs/DEVELOPER-SURFACE.md`). Otherwise "not developer-facing".
- **Documentation:** the unit's docs updated in the same PR (behaviour and docs never in separate PRs): README, CHANGELOG
  entry, ROADMAP/BACKLOG/BUGS rows, an ADR for a decision.
- **Evidence:** every command run with its exit code (build, tests, lint, format, coverage). A machine result is a fact,
  not a verdict; only the operator declares a change done.
- **Tests:** new or changed behaviour has a test; **line coverage of every touched file is at least 90%**; the new test
  was **seen failing** without the change (break it, watch it fail, restore it) — say how.
- **Risk and rollback:** what could break, and how to undo it (a migration is forward-only: say how the previous version
  keeps working).

## 4. Gates every repository enforces

- Format and lint with warnings as errors (`cargo fmt --check`, `cargo clippy -D warnings` on host and `wasm32` for
  Workers; the repository's JS/TS linters).
- All tests; the repository's coverage report for the touched files.
- `--locked` builds against the committed lock files.
- Secret scanning (`gitleaks`); never commit a credential, a real token or a credential-shaped literal, not even in a
  test (build such strings at runtime).
- The OpenAPI gates (lint with the pinned linter, examples validate, version equals the contract) when a contract changes.
- A Worker's `wrangler deploy --dry-run` builds.

## 5. Code and design rules

- Each repository's `AGENTS.md` and `docs/` hold its domain rules; read them first.
- No new raw colour in UI code (the design system's tokens only).
- No new dependency without a reason in the PR; Rust follows the platform's Rust engineering rules (no `unwrap` in
  production paths, unsafe only in designated crates, no locks across `.await`).
- Breaking changes: never in place. Add the new beside the old, mark the old deprecated with a removal date at least 30
  days away, announce it (`Deprecation`/`Sunset` on HTTP), remove it in its own later PR.
- Personal data: collect the minimum, document retention, never log it.

## 6. Where to ask, and how to report a vulnerability

Questions and proposals: open an issue in the repository. Security vulnerabilities: never in a public issue; follow
[SECURITY.md](SECURITY.md).
