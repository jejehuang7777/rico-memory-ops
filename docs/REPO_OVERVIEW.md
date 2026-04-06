# rico-memory-ops Overview

Portable memory ops patterns for agent continuity, handoff, and recall.

## What this is

`rico-memory-ops` is a starter kit for teams building long-running agent workflows that need:

- stable handoffs across sessions
- lightweight memory schemas
- recall triggers that wake the right context
- clear boundaries between hot working state and durable memory

This repo publishes the tool layer only.

It does not include any private journals, user profiles, or personal history.

If a pattern cannot survive redaction, it stays private.

## Why this exists

Many agent workflows can store notes, but still fail at one of these steps:

- knowing what memory object should exist
- knowing when a memory should wake up
- knowing what should stay hot vs be demoted
- handing work across sessions without reloading the whole system

This repo focuses on those operating problems.

## Current scope

- `RICO`
  - `Roadmap + Intent + Cue + Origin`
- continuity and handoff templates
- public/private boundary guidance
- synthetic examples

## What is not in scope

- private memory corpora
- user-specific life logs
- personality packaging
- large graph infrastructure

## Starter contents

- `docs/PUBLIC_PRIVATE_BOUNDARY.md`
- `docs/APPLICATION_CHECKLIST.md`
- `docs/CODEX_FOR_OSS_DRAFT.md`
- `examples/rico-example.yaml`
- `templates/session-bootstrap.md`
- `templates/task-handoff.md`

## Who this may help

- maintainers of multi-session AI workflows
- teams building agent memory layers
- people who need cleaner handoff between humans and agents

## Roadmap

### v0

- publish the minimum RICO shape
- publish one synthetic example
- publish boundary rules for safe open sourcing

### v1

- add more examples from continuity, recall, and handoff
- add a cue registry draft
- add starter templates for session bootstrap and task handoff

## Open-source boundary

This project is intentionally derived from a private operating system, but only the reusable method layer is published here.

If a pattern requires exposing personal history to make sense, it does not belong in this repo.

## License

MIT
