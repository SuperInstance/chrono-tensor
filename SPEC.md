# chrono-tensor — Specification

The event format as testable rules. A stream either complies or it does not; each rule below can be checked mechanically against a recording. Terms: **event**, **tick**, **track**, **stem** are defined in ARCHITECTURE.md.

## 1. The event

**E1 — Required fields.** Every event carries all of:

| Field | Type | Rule |
|---|---|---|
| `tau` | integer or fixed-point | The event's tick on the producing node's clock. First field in the serialization (R1). |
| `kappa` | integer, 0–15 | Track index. Displayed as Track `kappa + 1`. Must match the allocation table in §3. |
| `v` | numeric vector | The intent vector. Dimension is fixed per track by the producing node's declaration (§5); may be length 0 only on tracks declared stateless-marker tracks. |
| `theta` | number | Pressure/velocity scalar for this event. |
| `provenance` | record | Producing node id, producing patch/process id, and the ids of any prior events this event answers. Node and patch ids are mandatory; prior-event ids are mandatory when the event is a response, a correction, or a judgement of another event. |
| `cost` | record | Cost of producing and transmitting the event, in declared units (compute ticks and bytes mandatory; energy optional). Zero is a value that must be written, not a default that may be omitted. |

**E2 — Event identity.** Each event has a unique id within its stream, assigned at write time, never reused — including after downmixing (stems reference the ids they summarise by range, §4).

**E3 — Immutability.** A written event is never modified or deleted in place. A correction is a new event whose `provenance` references the corrected event. The only removal permitted is downmixing under §4, which leaves a stem.

**E4 — One ending.** An event records a state change that has happened. It is not a request, a lock, or a promise. (Requests and proposals exist in beam-quilt as transmissions; when one is sent, received, accepted, or refused, *that fact* is the event.)

## 2. Serialization

**R1 — Time-first.** `tau` is the first field of every serialized event, and events are stored ordered by `tau` within each track. A reader must be able to establish the tick of an event without parsing its payload.

**R2 — Track-explicit.** `kappa` is stored with every event, not inferred from which file or segment the event sits in. Concatenating two streams must not change any event's track.

**R3 — Self-describing segments.** Every stream segment (file, buffer wrap, network batch) opens with a header declaring: format version, the track allocation in use (the table in §3, or a named variant), the producing node id, the tick epoch the segment's `tau` values count from, and the declared dimension of `v` per track.

**R4 — Bit-packing is optional.** A compact binary encoding (e.g. bit-packed fixed-width words) is a permitted transport/storage encoding. It is not the format. Any encoding must round-trip: decode(encode(event)) yields an event identical in every field of E1. An implementation that only offers the packed form, with no lossless path back to the field-level event, is non-compliant.

**R5 — No silent precision loss.** Quantizing `v` or `theta` (e.g. to INT8) is permitted only if the segment header declares the quantization and its scale. Undeclared precision loss is a spec violation, because a later reader cannot distinguish a small value from a rounded one.

## 3. Track allocation

The allocation of ARCHITECTURE.md §2 is normative:

| kappa | Track | Group | Contents |
|---|---|---|---|
| 0 | 1 | Kinetic | Node phase / cadence |
| 1 | 2 | Kinetic | Execution velocity |
| 2 | 3 | Kinetic | Rest intervals, deadband drag |
| 3 | 4 | Kinetic | Boundary seam friction |
| 4 | 5 | Semantic | Core intent vector |
| 5 | 6 | Semantic | Abstraction depth |
| 6 | 7 | Semantic | Dissonance / judge disagreement flags |
| 7 | 8 | Semantic | Associative drift |
| 8 | 9 | Spatial | Position / coordinate axis 1 |
| 9 | 10 | Spatial | Position / coordinate axis 2 |
| 10 | 11 | Spatial | Position / coordinate axis 3 |
| 11 | 12 | Spatial | Attention density / focus |
| 12 | 13 | Regulation | Environmental entropy (input volatility) |
| 13 | 14 | Regulation | Historical weight — downmixed stems |
| 14 | 15 | Regulation | Hardware / node vitals |
| 15 | 16 | Regulation | Coordination — fleet tempo, rule-state markers |

**T1.** An event whose content belongs to a different group than its `kappa` claims is non-compliant, even if well-formed.

**T2.** A stream using a variant allocation (e.g. the conversational allocation noted in ARCHITECTURE.md's appendix) must name the variant in every segment header (R3). Mixing allocations inside one stream is non-compliant.

**T3.** Track 7 events are flags: `v` carries the disagreeing scores or references, `provenance` must reference the events judged. A Track 7 event with no referenced events is non-compliant.

## 4. Downmixing

**D1 — When.** A segment may be downmixed only after it leaves the stream's declared full-fidelity window. The window (in ticks or wall time) is declared in the segment header (R3) and is the same for the whole stream.

**D2 — What a stem contains.** For each track present in the segment: event count, tick range, min/max/mean of `theta`, and the count of events by ending state where applicable. Every stem also carries: the source segment id, the summarizer name and version, and the tick at which downmixing happened (as its own Track 14 event).

**D3 — What is never downmixed away.** The following are copied into the stem verbatim, in full E1 form: Track 7 flags; deadband breaches; errors; any event referenced by the `provenance` of an event outside the segment; any event marked notable by its producer.

**D4 — Honesty.** Downmixing is lossy. After downmixing, the unretained raw events are unrecoverable, and the stem must not be presented — by any tool — as equivalent to the raw segment. Tools displaying a stem must mark the range as summarized.

**D5 — No double summarization without trace.** A stem may itself be downmixed into a coarser stem only if the new stem lists every stem it absorbs. A reader must always be able to count how many summarization passes a range has been through.

## 5. Producer declarations

**P1.** Before writing to a stream, a producer declares: node id, the tracks it will write, the dimension of `v` on each, its cost units, and its tick source (which clock `tau` counts). The declaration is stored with the stream and versioned; changing it starts a new segment.

**P2.** A producer that cannot supply a required E1 field does not write to the stream. Adapters for legacy flat logs must supply `tau`, `kappa`, and `provenance` at emission time; logs retro-fitted with guessed ticks are non-compliant (see ARCHITECTURE.md contributor rule 2).

## 6. Compliance checklist

A stream is chrono-tensor compliant when all of the following check out:

- [ ] Every event has all E1 fields, with `cost` written even when zero.
- [ ] `tau` is the first serialized field; events are tick-ordered per track (R1).
- [ ] `kappa` is stored per event and matches the §3 table or a header-named variant (R2, T2).
- [ ] Every segment has a complete R3 header, including tick epoch and per-track `v` dimensions.
- [ ] Any binary encoding round-trips losslessly to the field-level event (R4).
- [ ] Any quantization of `v`/`theta` is declared in the header (R5).
- [ ] No event has been edited or deleted in place; corrections are referencing events (E3).
- [ ] Track 7 events reference the events they judge (T3).
- [ ] The full-fidelity window is declared and uniform across the stream (D1).
- [ ] Every downmixed range has a Track 14 stem with source range, counts, and summarizer version (D2).
- [ ] All D3-protected events survive downmixing verbatim.
- [ ] Downmixed ranges are marked as summarized wherever they are displayed (D4).
- [ ] Producer declarations exist and match the segments actually written (P1).
- [ ] Ambiguities and conflicts appear in the record (Track 7) rather than having been silently coerced (ARCHITECTURE.md rule 3).
