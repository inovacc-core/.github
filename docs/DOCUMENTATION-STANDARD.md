# Documentation standard
<!-- rev:001 (RFC 3339) 2026-10-17T22:39:55Z -->

Operator, 2026-10-17: *"Todos los proyectos tienen que ser documentados. Cada worker, cada container, cada lógica tiene
que tener su carpeta, docs, documentado, con su ADR, books, findings, roadmap, milestone, todo lo que se necesite."* This
is a hard rule for every repository of the organization. Documentation changes ride in the same pull request as the
behaviour they describe ([CONTRIBUTING.md](../CONTRIBUTING.md) §3).

## 1. Units and their document sets

A **unit** is something that is built, deployed or released on its own.

| Unit | Recognised by | Document set |
|---|---|---|
| **Worker** | a `wrangler.jsonc` | full set (§2) |
| **Container** | a `Dockerfile` that is deployed | full set (§2) |
| **Service crate / library** that other units depend on | a `Cargo.toml` with `[package]`, a `go.mod`, a published `package.json` | `README.md` (§3); a full set when it is released on its own |
| **Repository** | the git root | `README.md`, `AGENTS.md`, `CONTRIBUTING.md`, `CODEOWNERS`, `docs/` for decisions that span its units |

## 2. The full set (Workers and containers)

```text
<unit>/
├── README.md            what it is, why it exists, how to run, test and release it (top-level, short)
├── CONTRACT.md          only if it has an API (public or internal); openapi*.yaml beside it when public (rule 50)
└── docs/
    ├── ARCHITECTURE.md  components, bindings, data, flows (Mermaid), trust boundaries
    ├── ROADMAP.md       milestones (M1, M2, ...) with goal, done-signal and status
    ├── BACKLOG.md       planned work not yet started, each row with an id (BL-NNN)
    ├── BUGS.md          findings: defects in shipped or committed work (BUG-NNN), open and fixed
    ├── ISSUES.md        limitations accepted on purpose (ISS-NNN), each with its reason
    ├── RUNBOOKS.md      operations: release, rollback, secrets, migrations, incidents, health checks
    ├── CHANGELOG.md     every released version (Keep a Changelog format), newest first
    └── adr/
        ├── README.md    index of this unit's ADRs and of the platform ADRs that govern it
        └── NNNN-title.md
```

Every file holds **real content read from the code, the contract, the git history and the release tags**. A file with
nothing to say yet says so in one line ("No accepted limitations recorded as of YYYY-MM-DD.") — never placeholder text,
never invented facts. A fact the repository does not establish is written as UNKNOWN with what would settle it.

### README.md

```markdown
# <name>
<!-- rev:001 (RFC 3339) <timestamp> -->

<One paragraph: what this unit does and for whom.>

| | |
|---|---|
| Kind | Worker / container / library |
| Status | released `<tag>` / built, not released |
| Public host | `<host>` or none (binding only) |
| Contract | [CONTRACT.md](CONTRACT.md) `<version>` / none |
| Owner | <team or person> |

## Run, test, release
<the exact commands; release = a published GitHub Release tagged <pattern>, on the operator's go>

## Documents
ARCHITECTURE · ROADMAP · BACKLOG · BUGS · ISSUES · RUNBOOKS · CHANGELOG · ADRs (links)
```

### ARCHITECTURE.md

Purpose; components (modules/files and what each owns); bindings (D1, KV, R2, Queues, Durable Objects, service
bindings) with the names in `wrangler.jsonc`; callers and callees; data it stores and its retention; one Mermaid flow per
main path; trust boundaries (who may call it, how it authenticates them).

### ROADMAP.md

```markdown
| Milestone | Goal | Done-signal | Status | Evidence |
|---|---|---|---|---|
| M1 | <outcome> | <observable check> | done / in progress / planned | <tag, commit, file:line> |
```

Milestones come from what the unit already did (its release tags) and what is planned (BACKLOG, operator decisions).

### BACKLOG.md, BUGS.md, ISSUES.md

One table each, stable ids, never renumbered: `| ID | What | Why / evidence | Status |`. BUGS holds defects (open and
fixed, with the fixing commit); ISSUES holds accepted limitations; BACKLOG holds planned work. A finding goes into BUGS
**when it is found**, not at the end of a session.

### RUNBOOKS.md

Sections: release; rollback; secrets (names only, how to rotate, never values); migrations (forward-only, order);
health checks (the exact URL or command and the expected answer); incident steps for each known failure (symptom, check,
fix). Commands are exact and copyable.

### CHANGELOG.md

[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format: `## [vX.Y.Z] - YYYY-MM-DD` with `Added`, `Changed`,
`Deprecated`, `Removed`, `Fixed`, `Security`; built from the release tags and the commits between them. Developer-facing
entries also get a row in the platform's `SURFACE-CHANGELOG.md` (rule 50).

### adr/

```markdown
# NNNN. <Decision title>

- Status: proposed / accepted / superseded by NNNN
- Date: YYYY-MM-DD
- Deciders: <who>

## Context
## Decision
## Consequences
## Alternatives considered
## References (file:line, commits, platform ADRs)
```

`adr/README.md` lists this unit's ADRs and the platform-wide ADRs that govern it. A decision already written in prose
(a README "decision", an operator ruling recorded in docs) becomes an ADR that cites where it was recorded; nothing is
decided anew while documenting.

## 3. Library README (crates and packages)

```markdown
# <crate>
<!-- rev:001 (RFC 3339) <timestamp> -->

<What it provides, in two or three sentences, and which units use it.>

| | |
|---|---|
| Kind | library (`cargo` package `<name>`) |
| Used by | <units> |
| Status | internal / published `<version>` |

## API
<the main types and functions, one line each>

## Test
<exact command>
```

## 4. Rules for every document

- **Rev tag** on living documents: `<!-- rev:NNN (RFC 3339) <timestamp> -->` on the line after the H1, bumped by exactly
  one with a fresh UTC timestamp on every edit. ADRs and dated records are point-in-time and carry no rev tag.
- **English.** Exact commands. File references as `path:line`. No secrets, no customer data.
- **One source per fact:** link to the contract or the platform doc instead of copying it.
- **Developer surface (rule 50):** a change a developer can notice also moves `CONTRACT.md`, `openapi*.yaml`, the SDKs, the
  portal and the surface changelog.
