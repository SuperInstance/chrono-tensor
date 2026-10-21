# chrono-tensor

**chrono-tensor is the temporal event format and recording layer for sovereign agent fleets — every state change is recorded as a 4D event (time/tick, channel/agent, intent, pressure) in one scrubbable multi-track stream.**

It is for builders of multi-agent and edge-fleet systems who need one shared answer to a narrow question: *in what format do we write down what happened, in time order, so that any node — or any later investigator — can replay it?*

Today that record is scattered across per-node text logs, transcripts, and job histories with incompatible timestamps. chrono-tensor replaces that with a single event format: each event carries a tick (`tau`), a track (`kappa`, 0–15), an intent vector, and a pressure/velocity value (`theta`), plus provenance and cost. Sixteen tracks group events by kind — kinetics, semantic intent, spatial, regulation — so a run can be read as one multi-track score instead of sixteen separate logs.

## What it is not

- **Not an ontology.** It makes no claim about what intelligence is or how nodes should think. Earlier drafts wrapped the format in a "continuum" metaphysics; that material stays in `docs/ideation/` as provenance.
- **Not a runtime.** It does not execute agents, schedule work, or route messages. The runtime is a separate repo (`continuum-runtime`).
- **Not a replacement for beam-quilt.** The seam architecture — sovereign patches, transmission envelopes, sender/receiver judges, consent rules — belongs to **beam-quilt**. chrono-tensor is beam-quilt's time and logging layer: it specifies the record that beam-quilt's SPEC already requires (P5 logs, C2 visible seams).

## Component map

| Piece | Lives in |
|---|---|
| The 4D event format, track allocation, downmixing rule | **this repo** (`chrono-tensor`) |
| The recorder / flight-recorder tool (`chrono-flight`) | `chrono-flight` repo |
| The runtime that executes on the stream | `continuum-runtime` repo |
| Seam architecture, patch contract, consent rules | `beam-quilt` repo |

One tool, one repo. This repo holds the format and the rules for writing and compressing the stream — nothing that executes it and nothing that displays it.

## Reading order

1. **README.md** (this file) — what the format is and is not.
2. **ARCHITECTURE.md** — the buildable core: event, tracks, ring buffer, downmixing, and an honest confidence section separating settled design from speculation.
3. **SPEC.md** — the format as testable rules, with a compliance checklist.
4. **ROADMAP.md** — build order and what comes first.
5. **docs/ideation/** — the raw Gemini conversation this was sorted from. Provenance, not instructions. Read it for context and for nuggets the builder docs filtered out; do not build from it directly.

## Status

Specification stage. No code yet. The event format, the scrubbable ring buffer, and downmixing (as lossy summarization) are the settled parts. Claims beyond that — microsecond rewind of a live swarm, lattice routing — are marked speculative in ARCHITECTURE.md and are not part of what this repo promises.
