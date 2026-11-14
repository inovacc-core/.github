# Platform backlog
<!-- rev:002 (RFC 3339) 2026-11-14T13:56:16Z -->

Planned work that spans the whole inovacc-core platform rather than one repository. Work owned by one domain stays in
that repository's own `docs/BACKLOG.md`; a row here moves there once its owner is decided.

| ID | What | Why / evidence | Status |
|---|---|---|---|
| BL-001 | A feature-flag system for the whole platform: one way for every Worker, container, console and client of every inovacc-core repository to turn a capability on or off (per environment, organisation, application or user) without a release | Operator, 2026-10-29: "un sistema de feature flags para toda la plataforma ... va a atravesar toda la plataforma ... después vemos cómo va a ser la implementación ... después hacemos el brainstorm" | Open: design not started; waits on a brainstorm with the operator (owner domain, storage, evaluation, SDK, console, audit) |
| BL-002 | A store for the cloud copy of an application's master key: for one signed-in account and one pair of product and profile, the platform keeps one 119-byte sealed container that it cannot open, replaced in two steps (stage, read back, commit) | Operator, 2026-11-14: "salvar una copia de la credencial en la nube. La nube sería mi plataforma", "ni la plataforma sepa cómo descriptografiarla", and "infórmalo en la organización InnovaCC Core y solamente déjalo documentado para otro agente implementarlo". Specified in `inovacc-shared/specs`, `secrets/docs/spec/KEY-LIFECYCLE-v1.md` section 13 | Open: owner domain not decided; no contract and no code. The client side exists (`inovacc-shared/masterkey`). Detail below |

## BL-002: the cloud copy of a master key

Written for the agent that will implement it. Nothing here is an order: work starts on the operator's go.

**What it is for.** An application keeps a 32-byte master key on the person's machine. A product that declares it
also keeps a copy on the platform, so that the account and a password bring the key back on another machine. The
application seals the master on the machine under a password of at least 12 characters (Argon2id at 2 GiB, then
XChaCha20-Poly1305) and uploads only the result: a 119-byte container. The platform never receives the password or
the master, cannot open the container and cannot reset the password. The decision is
`inovacc-shared/specs`, `secrets/docs/adr/0001-cloud-copy-sealed-by-a-password.md`.

**Where it is specified.** `B:\platform\inovacc-shared\specs\secrets\docs\spec\KEY-LIFECYCLE-v1.md`, section 13: the
container's layout, rule K-91 (what the platform does), the operations table, and rules K-93 to K-96. That text rules
over this entry; read it before designing anything.

**What exists.** The client: `B:\platform\inovacc-shared\masterkey`, `src/cloud/copy.rs`. Its trait `Platform` has the
six operations the client calls (`current`, `stage`, `staged`, `commit`, `discard`, `delete`), and its tests run
against a stand-in in memory. No client speaks HTTP yet: that is written once this entry has a contract.

**What the platform must provide** (operations, not routes; the owning domain gives them paths):

| Operation | What it does |
|---|---|
| `current` | Returns the committed container and the time it was stored, or nothing. |
| `stage` | Stores a container as the pending copy. The committed copy is untouched. A new `stage` replaces the pending copy. |
| `staged` | Returns the pending container. |
| `commit` | The pending copy becomes the committed one, in one step. The copy it replaces is deleted, not kept. |
| `discard` | Drops the pending copy. |
| `delete` | Deletes the committed copy and any pending one. |
| `events` | Lists the uploads, downloads and deletions of this copy with their times, for the account. |

**Rules it must hold** (the ids are the specification's):

- One container for each account, product and profile, served, replaced and deleted only for that signed-in account,
  over TLS (K-91).
- `stage` accepts only a body of exactly 119 bytes that begins with `IKM1` and the byte 3. The platform inspects
  nothing else and stores the bytes as given (K-94).
- Every operation needs a token of the account with a scope nothing else uses. No console, support or operator route
  returns a container (K-95).
- The copy is deleted when the account is deleted (K-91). The container is the account holder's data: retention and
  deletion follow LGPD.
- Proposed, not decided (the operator's D10 in `inovacc-shared/specs/secrets/README.md`): a pending copy expires after
  one hour; `current` and `staged` together are limited to 10 calls a day for one copy; the account is told of every
  download (K-96).
- The platform does no cryptography on the container and has no operation that opens it.

**What the implementer brings to the operator before building:**

1. The owner domain. `identity` owns accounts, tokens, `/v1/me` and device sign-in (`identity/AGENTS.md:8`); `data`
   owns blobs, files and the activity log (`data/AGENTS.md:8-9`). This entry moves to that repository's
   `docs/BACKLOG.md` once decided.
2. The scope's name, the paths, and where the bytes are stored.
3. Whether `events` is the Data API's activity log or a record of its own.
4. D10 above.

**It is developer surface** (rule 50): `CONTRACT.md` and `openapi*.yaml` together, the SDKs, the portal, the
`SURFACE-CHANGELOG.md` row.

**How it is accepted.** An implementation of the client's `Platform` trait over the real routes passes the tests that
today run against the stand-in (`masterkey`, `src/cloud/copy.rs`: an upload then a restore gives the master; a
read-back that differs fails and leaves the previous copy; a platform that does not answer gives `Unreachable`; a
refused account gives `Unauthorized`). On the platform's side, each of these is a test seen failing once: a `stage`
body of 118 bytes is refused; a second account cannot read the first one's copy; a token without the scope is
refused; deleting the account removes the copy; after a `stage` with no `commit`, `current` still returns the
previous copy.
