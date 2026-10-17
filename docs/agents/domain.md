# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

ADR storage: repository files. `grill-with-docs`, `triage` and `improve-codebase-architecture`
read this repo's `docs/agents/domain.md` before selecting `domain-modeling` with the configured
glossary and ADR layout. Work-tracker choice does not override ADR storage.

## Before exploring, read these

- **`GLOSSARY.md`** at the repo root, or
- **`GLOSSARY-MAP.md`** at the repo root if it exists: it points at one `GLOSSARY.md` per context. Read each one relevant to the topic.
- **`docs/adr/INDEX.md`**: maps every ADR to its path and its topic or concern. Use it to find the ADRs that touch the area you're about to work in, then read those.

If `GLOSSARY.md` doesn't exist yet, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The `/domain-modeling` skill (reached via `/grill-with-docs`, `/triage` and `/improve-codebase-architecture`) creates them lazily when terms or decisions actually get resolved.

## File structure

Single-context repo (most repos):

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── INDEX.md
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

Multi-context repo (presence of `GLOSSARY-MAP.md` at the root):

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── GLOSSARY.md
        └── docs/adr/
```

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `GLOSSARY.md`. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal: either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/domain-modeling`).

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (event-sourced orders), but worth reopening because…_

## Keep the ADR index current

Every ADR added, superseded or retitled under `docs/adr/` gets its row in `docs/adr/INDEX.md` updated in the same change.

## Public repository

This repo is public. Never write private tracker references (Linear or Plane identifiers, employer tickets, internal URLs) into tracked files, issues, PRs or commit messages.
