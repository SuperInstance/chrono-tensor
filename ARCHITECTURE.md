# chrono-tensor — Architecture

The buildable core only. Anything that could not be stated as a data format, a storage rule, or a testable behaviour was filtered out of the ideation documents; what was filtered, and why, is listed in the Confidence section so a builder never has to guess which parts are load-bearing.

## 1. The 4D event

Every state change in a participating fleet is recorded as one event:

```
T = ( tau, kappa, v, theta )  +  provenance  +  cost
```

- **tau** — the temporal locus: a tick on the recording node's own clock. Time is the primary key of the whole format; an event without a tick is not an event.
- **kappa** — the track index, an integer 0–15. The track says what *kind* of thing changed, so heterogeneous streams stay separable in one recording.
- **v** — the intent vector: a low-dimensional numeric vector describing the change (a projected intent, a telemetry delta, a judge score profile). Content is typed by track; the format does not interpret it.
- **theta** — phase/velocity, recorded here as pressure: how fast or how hard this thread is moving (rate of change, load, urgency). One scalar per event.
- **provenance** — which node, which patch, and which prior event(s) this one answers. A recording that cannot say where an event came from cannot be audited.
- **cost** — what producing and transmitting the event cost (compute ticks, bytes, energy budget unit). Cost is recorded, not assumed zero.

An event is immutable once written. Corrections are later events that reference the corrected one; nothing is edited in place. This is what makes the stream a record rather than a dashboard.

## 2. The 16-track allocation

Tracks are allocated by function, in four groups of four (allocation from the ideation core, `docs/ideation/gemini1.md`):

| Track (kappa) | Group | Contents |
|---|---|---|
| 1 (0) | Kinetic | Local phase / cadence of the node |
| 2 (1) | Kinetic | Execution velocity |
| 3 (2) | Kinetic | Rest intervals, deadband drag |
| 4 (3) | Kinetic | Boundary seam friction (load at the seams) |
| 5 (4) | Semantic | Core intent — the primary reasoning/action vector |
| 6 (5) | Semantic | Abstraction depth of the current work |
| 7 (6) | Semantic | Dissonance — flagged logical contradiction or judge disagreement |
| 8 (7) | Semantic | Associative drift |
| 9 (8) | Spatial | Position / lattice coordinate, axis 1 |
| 10 (9) | Spatial | Position / lattice coordinate, axis 2 |
| 11 (10) | Spatial | Position / lattice coordinate, axis 3 |
| 12 (11) | Spatial | Attention density / focus |
| 13 (12) | Regulation | Environmental entropy (input volatility) |
| 14 (13) | Regulation | Historical weight (downmixed past, see §4) |
| 15 (14) | Regulation | Hardware / node vitals |
| 16 (15) | Regulation | Coordination track — fleet-level tempo and rule-state markers |

Track 7 is the one with an operational job beyond recording: it is where a small, fast judge writes when it detects contradiction between what two tracks (or two nodes) assert. It records the flag and its score; resolving the contradiction is the receiver's business, not the format's.

A node emits only on the tracks it has something for. Sixteen tracks is the address space, not a requirement that every node fill all of them.

## 3. The scrubbable ring buffer

The stream lives in a ring buffer per node:

- Events append in tick order within each track; the buffer is the single store — there is no separate log file the buffer is a view onto.
- **Scrubbing** is reading with a movable playhead: position the playhead at a tick, read any subset of tracks forward or backward, mute tracks, compare two windows. Scrubbing is a read operation on recorded events. It does not re-run anything.
- When the buffer wraps, the oldest segment is not silently dropped: it is downmixed first (§4) and the downmixed stem is retained on Track 14 before the raw segment is released. Losing history must be a visible, recorded act, never a side effect of a full buffer.

What scrubbing gives you, concretely: a post-mortem that shows the *approach* to a failure — velocity rising on Track 2, friction on Track 4, a dissonance flag on Track 7 — in time order, across nodes that shared a tick reference, instead of sixteen unsynchronised text logs.

## 4. Temporal downmixing

History is stored at two fidelities:

- **Recent history: full fidelity.** Every event, as written.
- **Older history: background stems.** A segment past its full-fidelity window is compressed into a per-track summary: event counts, ranges and means of `theta`, and — preserved verbatim — the notable events: dissonance flags, deadband breaches, errors, and any event another event's provenance points back to.

Rules that keep this honest:

- Downmixing is **lossy summarization**. The raw events in a downmixed segment are gone; the stem must say so. A stem records its source tick range, its event count, and the version of the summarizer that produced it.
- Notable events are never summarised away. The summary is of the routine, and the definition of routine is: nothing flagged, nothing referenced, nothing breached.
- The downmixed stem is itself an event stream (Track 14), so an investigator can see *that* history was compressed, *when*, and *by what rule* — the compression is part of the record.

This is the format's answer to unbounded log growth (beam-quilt SPEC P5 requires logs to be garbage-collected at a declared rate): the rate and the rule are declared here, and the collection leaves a stem behind.

## 5. Contributor rules

From CTC-v1 (`docs/ideation/gemini3.md`), kept because they are enforceable:

1. **Time-first serialization is mandatory.** Every event, log entry, and telemetry packet carries its tick and track index natively, at write time. Timestamps reconstructed later from file order or wall-clock metadata do not comply.
2. **Flat unstructured logs are non-compliant.** An append-only text log with no tick, no track, and no provenance is not a chrono-tensor recording, whatever it is named. Producers that can only emit flat logs need an adapter that assigns ticks and tracks at emission, or their output stays outside the stream.
3. **Inductive error handling.** Ambiguous or conflicting input is not an exception to trap and hide. It is recorded — as a Track 7 flag with its scores — and handled by the receiving node's own judgement (in beam-quilt terms, the deadband: accept, refuse, or ask). The format's job is to make sure the ambiguity survives into the record; silently coercing it to a clean value destroys evidence.
4. **Zero shared mutable state.** Events are proposals to the record, appended by their producer. No producer writes into another node's buffer; streams are merged by tick and provenance at read time, not by shared access at write time.

## 6. How this serves beam-quilt

beam-quilt's SPEC makes two demands this format exists to meet:

- **P5 (patches keep their own logs).** A patch's log, in this format, is its track range in the shared stream: tick-ordered, provenance-carrying, downmixed at the declared rate instead of growing forever.
- **C2 (seams must be visible to the owner).** A seam crossing is a transmission, and a transmission is an event — on the sender's tracks when it is projected, and on the receiver's when it is accepted, refused, or lands in the deadband. If it crossed a seam, it is in the stream, with its cost. An owner scrubbing the stream sees every crossing; there is no channel that bypasses the record.

chrono-tensor does not define the seam, the judges, or the consent rules. It defines the paper they are written on.

## 7. Confidence

### Settled — build on these

- **The 4D event format** (§1) and the track allocation (§2). A data format makes no empirical claim; it is settled by being specified (SPEC.md) and used consistently.
- **The scrubbable ring buffer** (§3) as a storage and reading discipline. Append-only event stores with movable read cursors are ordinary, proven engineering.
- **Downmixing as lossy summarization** (§4) — settled *because* it is honestly named. It is summarization with a retention rule for notable events, not a compression miracle; historical depth is traded for size, openly.

### Speculative — do not promise these

- **Microsecond rewind and re-simulation of a live swarm.** Scrubbing the *record* is settled. Re-running the *system* from a scrub point requires deterministic replay: every node must produce identical behaviour from identical recorded inputs. That is unproven for LLM-based nodes, whose sampling is nondeterministic unless seeds, settings, and model versions are all captured — and undefined for nodes whose inputs include physical sensors and human messages. Treat "rewind, mute a track, press play" as a research goal for the recorder tool, not a capability of this format.
- **Lattice routing.** The ideation documents route messages across an Eisenstein A₂ lattice by "topological proximity". No mechanism is given — no addressing rule, no forwarding rule, no failure behaviour — and spatial coordinates on Tracks 9–12 are a recording convention only. Routing is a runtime problem (`continuum-runtime`) and is not specified here.
- **Cross-node tick alignment.** Events are tick-ordered within a node. Aligning ticks *across* nodes to microsecond precision over real fleet links is asserted in the ideation documents and not established anywhere; the format records each node's tick plus provenance and leaves skew visible rather than pretending it away.

### Parked — metaphors that name no mechanism

The ideation documents use a physics vocabulary throughout. Each term, reduced to what it actually describes:

- **Kuramoto phase-locking** → nodes adjusting their cadence toward the cadence they observe in neighbours. A scheduling heuristic; no oscillators are coupled and no equation is evaluated.
- **Hodge decomposition of friction** → sorting disagreements into three buckets: locally fixable noise, loops/deadlocks, and structural bottlenecks. A triage label set; there is no field and no decomposition operator.
- **Laman rigidity ("truth is a rigid graph")** → counting constraints in a dependency graph to check whether the graph is fully determined. A useful under-specification heuristic (see ROADMAP step 3); a rigid graph can be rigidly wrong, so rigidity is never a test of truth.
- **Spectral conservation** → requiring a proposal's cost to be accounted before it crosses a seam; in practice, rate-limiting — don't flood the seam.
- **Standing wave / resonance / harmonic convergence** → several nodes' assessments agreeing, and the agreement being visible. Agreement is recorded (Tracks 5–8, 13); nothing vibrates.

Builders: if a future document uses one of these terms without cashing it out into a rule as plain as the reductions above, the term is decoration. Leave it in `docs/ideation/`.

## Appendix — variant track allocation

The beam-quilt proposed documents (`beam-quilt/docs/proposed/gemini22.md`) contain a variant allocation in which the sixteen tracks are assigned to *conversational* roles — participants and functions in a multi-party conversation — rather than to system telemetry, with Track 16 as a global conductor. It is recorded here so the variant is not lost. It is **not** adopted: this repo's allocation (§2) is the telemetry allocation from `docs/ideation/gemini1.md`, and a stream must declare which allocation it uses (see SPEC.md) rather than mixing the two.
