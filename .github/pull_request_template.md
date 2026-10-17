## What and why

<!-- A few sentences: what changes, why, and the issue / bug / decision it answers. -->

## Developer surface (rule 50)

<!-- Not developer-facing? Write "Not developer-facing" and delete the list. Otherwise tick each step or say why it does not apply. -->

- [ ] `CONTRACT.md` updated, contract version and rev bumped
- [ ] `openapi*.yaml` updated in this PR, `info.version` = contract version, lint + examples checked
- [ ] SDKs: updated / issue opened / not affected (say which)
- [ ] Developer portal: updated / follow-up PR linked (rule 49)
- [ ] `SURFACE-CHANGELOG.md` row added (`inovacc-core/developer/docs/SURFACE-CHANGELOG.md`)

## Documentation

- [ ] The unit's docs are updated in this PR (README, CHANGELOG entry, ROADMAP / BACKLOG / BUGS rows)
- [ ] A decision has its ADR (`docs/adr/NNNN-title.md`)

## Evidence

<!-- Every command you ran and its exit code. Machine results are facts, not verdicts. -->

| Command | Exit code / result |
|---|---|
| | |

- Line coverage of the touched files (each must be at least 90%):
- The new test was seen failing without the change: <!-- how you broke it and restored it -->

## Risk and rollback

<!-- What could break; how to undo it. For a migration: why the previous version keeps working. -->
