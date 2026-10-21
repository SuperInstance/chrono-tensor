# chrono-tensor — Roadmap

Build order, taken from the ideation documents (`docs/ideation/`) and re-sequenced where the source left the order open. Each step names the repo it lands in, per the component map in README.md: this repo holds the format; tools and runtimes live in their own repos.

## Step 1 — Event serialization (this repo)

Implement the SPEC.md format: the E1 event, the R3 segment header, producer declarations, and a reference encoder/decoder pair — one human-readable encoding and one bit-packed encoding, with the R4 round-trip property tested between them.

*Done when:* a stream can be written, read back, and pass every item on the SPEC compliance checklist, checked by a validator that ships with this step.

**Start here.** The ideation documents left the first build open — serialization, event bus, or lattice engine. Serialization is the answer because every other piece consumes the event, and none of them can be tested until "an event" is a concrete, checkable thing rather than a description.

## Step 2 — Event bus (this repo's transport; runtime behaviour in `continuum-runtime`)

A minimal append-and-subscribe bus that moves serialized events between producers and the ring buffer: producers append (P1 declarations enforced), readers subscribe by track and tick range, scrubbing is served as ranged reads. No routing intelligence, no cross-node synchronisation claims — transport of compliant streams, nothing more.

*Done when:* two producers on separate processes write one merged, tick-ordered, provenance-complete stream that a third process can scrub without either producer sharing memory with it.

## Step 3 — Friction and rigidity analysis (this repo's analysis library)

Offline analysis over recorded streams:

- **Friction reports:** per-seam and per-track aggregates from Tracks 4 and 7 — where disagreement concentrated, how often the deadband was hit, what it cost (from E1 `cost`). This is counting over the record; it needs no new theory.
- **Rigidity checks:** for a recorded decision's dependency graph (rebuilt from `provenance`), count constraints against degrees of freedom to flag under-determined conclusions. **Rigidity here is a heuristic for spotting under-specified reasoning — it is not a test of truth.** A fully constrained graph can be fully wrong; the check reports "this conclusion had fewer inputs than its shape requires," never "this conclusion is false."

*Done when:* the analyser, run over the Step 2 demo stream, reproduces the friction the run actually exhibited and flags a deliberately under-constrained decision planted in it.

## Step 4 — Piano-roll recorder UI (the `chrono-flight` repo)

The flight recorder: a piano-roll view of a stream — sixteen tracks, a movable playhead, mute/solo per track, stems visibly distinguished from full-fidelity ranges (SPEC D4), and a post-mortem mode that aligns several nodes' streams by provenance. This is `chrono-flight`, a separate tool with its own repo; it is a *reader* of this format and must never become a place where format decisions are made.

Explicitly out of scope for the first version: re-simulation from a scrub point. Replaying a live swarm from a recorded state requires deterministic replay, which is unproven for LLM-driven nodes (see ARCHITECTURE.md, Confidence). The recorder shows what happened; claiming it can re-run what happened is a later, separately argued feature.

*Done when:* an operator can take a recorded multi-node run, scrub to a failure, and see — in one view — the velocity, friction, and dissonance tracks that led into it.

## Not scheduled

- **Lattice routing** — no mechanism exists yet (ARCHITECTURE.md, Confidence). It returns to this roadmap only when someone specifies addressing, forwarding, and failure behaviour concretely.
- **A runtime** — belongs to `continuum-runtime`, sequenced there, not here.
- **Any use of the parked physics vocabulary** (Kuramoto, Hodge, Laman-as-truth, conservation) as an implementation description. The reductions in ARCHITECTURE.md §7 are the most these terms will ever mean in this project unless a real mechanism is proposed and specified.
