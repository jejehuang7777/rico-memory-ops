# rico-memory-ops

For a Chinese introduction, see [README_ZH.md](README_ZH.md).

Most AI memory systems help agents remember more.

`rico-memory-ops` is about something harder:
helping an agent remain coherent over time.

This is a starter kit for teams building long-running AI workflows that need more than note storage or semantic recall.
It focuses on the operating layer behind continuity:

- keeping judgment coherent across sessions
- waking the right context instead of reloading everything
- handing work across rooms, agents, or sessions without losing the thread
- separating hot working state from durable memory

It is designed for builders of persistent agents, continuity systems, and long-running human-AI workflows.

## Why this exists

Many agent systems can store notes and still fail at the harder problem:

they remember facts, but do not stay coherent.

In practice, that shows up as agents that:

- restart their judgment every session
- lose the thread during handoff
- wake too much context, or the wrong context
- blur hot state, archived memory, and identity-level guidance

`rico-memory-ops` is designed to help with those operating problems.

## Example situations

This kind of tooling becomes useful when:

- one agent hands work to another and the reasoning style should survive the handoff
- a long-running assistant should feel like the same mind continuing its work, not a fresh assistant every time
- a system needs to wake the right context for the current task without dumping the whole archive back into the prompt
- teams want continuity to come from repeatable operating rules, not just ad hoc prompt glue

## What makes this different

This repo is not trying to be a giant memory database or a personality wrapper.

It is trying to make three things more reliable:

1. `continuity across sessions`
2. `coherent judgment over time`
3. `memory that preserves identity-level working style, not just facts`

The core idea is simple:

> Memory is not enough.  
> An agent also needs continuity.

## Current scope

- `RICO`
  - `Roadmap + Intent + Cue + Origin`
- lightweight memory semantics for recall and handoff
- continuity and handoff templates
- synthetic examples
- publishing guidance for method-first open source

## What is not in scope

- giant memory corpora
- personality packaging
- large graph infrastructure for its own sake

## Starter contents

- [`docs/PUBLIC_PRIVATE_BOUNDARY.md`](docs/PUBLIC_PRIVATE_BOUNDARY.md)
- [`docs/APPLICATION_CHECKLIST.md`](docs/APPLICATION_CHECKLIST.md)
- [`docs/CODEX_FOR_OSS_DRAFT.md`](docs/CODEX_FOR_OSS_DRAFT.md)
- [`examples/rico-example.yaml`](examples/rico-example.yaml)
- [`templates/session-bootstrap.md`](templates/session-bootstrap.md)
- [`templates/task-handoff.md`](templates/task-handoff.md)

## Who this may help

- maintainers of multi-session AI workflows
- teams building agent memory or continuity layers
- people who want AI systems that feel less like a new assistant every time
- teams that need cleaner handoff between humans and agents

## Why open-source this

The goal is to publish the part other teams can directly reuse:

- schemas
- recall patterns
- handoff templates
- starter examples

See [`docs/PUBLIC_PRIVATE_BOUNDARY.md`](docs/PUBLIC_PRIVATE_BOUNDARY.md) for publishing boundaries and transformation rules.

## Roadmap

### v0

- publish the minimum RICO shape
- publish one synthetic example
- publish safe method-first publishing guidance

### v1

- add more continuity, recall, and handoff examples
- add a cue registry draft
- add starter templates for session bootstrap and task handoff

## License

MIT

