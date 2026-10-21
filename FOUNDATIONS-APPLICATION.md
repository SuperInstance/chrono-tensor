# Foundations application — chrono-tensor

What beam-quilt's verified foundations (beam-quilt `FOUNDATIONS.md`, Lane A
pull list) mean for this repo specifically. Tags: **SETTLED / PROPOSED /
SPECULATIVE**. The builder docs here were already the programme's most
careful; the foundations mostly *specify mechanisms the docs named but
left open* — and flag two places where a pulled mechanism does not fit
cleanly.

## ADAPT here

### ADAPT — LTTng session/channel/ring-buffer semantics (LGPL/GPL per-component; docs CC BY 4.0) → ARCHITECTURE §3, SPEC §4
- **Replaces:** the from-scratch model of "one scrubbable multi-track
  stream." Mapping: session = the fleet stream; track (kappa) = channel,
  independently enabled/disabled (= mute/solo at the format level);
  rotation/chunk archives + snapshot mode = downmix extraction;
  per-user/per-process buffer ownership is the shipped analogue of
  owner-consented recording (beam-quilt C1/C2).
- **Rebuild the surface, keep the semantics.** No LTTng code or licence
  obligation enters this repo; this is a semantics reference for the
  ring buffer's behaviour contract.
- Tag: **PROPOSED** (the §3 ring-buffer discipline itself stays SETTLED;
  the LTTng mapping is the vocabulary for specifying its edge cases).

### ADAPT — Kafka log-compaction contract (Apache-2.0 docs) → SPEC §4 downmixing
- **Replaces:** a downmixing rule that was named but, before the
  foundations, not specified as a contract. Properties to encode:
  addresses never change, order preserved, tombstones for deletion,
  latest-value-per-key.
- **Partial fit — read before adopting wholesale:**
  - *Addresses never change / order preserved* — already guaranteed here
    (SPEC E2 ids never reused; R1 tick order). **SETTLED**, no work.
  - *Tombstones* — map to E3 correction events (a referencing event that
    retires an earlier one), never to in-place deletion, which E3
    forbids. **PROPOSED** as the explicit mapping.
  - *Latest-value-per-key* — chrono-tensor events are immutable
    state-changes, not key-value state; this property applies only to a
    reader reconstructing "state as of tick τ" per track, not to the
    stream itself. Adopt it as a *reader* rule, not a storage rule.
  - *Current state rebuildable from the tail alone* — **does not hold
    here and must not be promised.** After downmixing, D4's honesty rule
    governs: stems are lossy, unretained raw events are unrecoverable.
    Kafka's rebuild property is exactly what downmixing trades away.
    Any builder importing the compaction contract must strike this clause
    and say so.
- Kafka the broker is rejected (central-cluster JVM system); only the
  contract transfers. Tag: **PROPOSED**, with the strike above.

### ADAPT — Chronicle Queue named tailers + mmap cycles (Apache-2.0) → ARCHITECTURE §3 scrubbing, §4 retention seam
- **Replaces:** an anonymous "movable playhead" as the only cursor model.
  Named, persistent, read-without-consume tailers are how each sovereign
  reader keeps its own scrub position across sessions; cycle-rolling is
  the retention seam where downmix/archive hooks in — which is precisely
  §3's wrap point ("downmixed first, then released").
- The cursor *implementation* lives in chrono-flight; this repo takes
  only the format-level consequence: a reader's position is a named,
  durable thing, not session state. Tag: **PROPOSED**.

### ADAPT (minor) — OpenTelemetry span context (Apache-2.0) → SPEC E1 `provenance`
- The primary home of this adaptation is quilt-engine's envelope. Here it
  is only a cross-check: if fleet tooling ever needs OTel interop, E1's
  provenance record (node, patch, prior-event ids) is the field set a
  span-context mapping would be built from. No format change now.
  Tag: **SPECULATIVE** for this repo.

## BUILD OUR OWN

- **The assembly itself.** Every recording mechanic exists somewhere; an
  owner-consented, per-patch-log, multi-track fleet stream with declared
  lossy downmixing has never been assembled. The format in SPEC.md is
  that assembly — the ADAPTs above supply vocabulary and edge-case
  semantics, not the design.
- **Cross-node tick alignment stays unbuilt and unclaimed** (ARCHITECTURE
  §7): the format records each node's tick plus provenance and leaves
  skew visible. Foundations found no prior art that changes this.

## Open decisions — NOT settled by the foundations

### Deadband semantics (Casey's ruling: clarify vs abstain as fourth outcome)
This repo does not implement the deadband; it *records its outcomes*.
That makes the recording surface ruling-sensitive in exactly three places:
- **SPEC D2 stem tallies** — "count of events by ending state" gains a
  category if abstain becomes a producible seam outcome; stems written
  before and after a ruling would not be tally-comparable unless the
  category set is versioned in the stem.
- **ARCHITECTURE §5 contributor rule 3** — currently "(in beam-quilt
  terms, the deadband: accept, refuse, or ask)". Under an abstain ruling
  the parenthetical gains a fourth recorded outcome; the rule itself
  (ambiguity survives into the record) is unchanged.
- **Track 3 (kappa 2, "rest intervals, deadband drag")** — under
  clarify-only, deadband drag measures time-to-clarification; under
  abstain, a coast is a legitimate terminal reading on this track, not
  drag. The track's contents do not change; their interpretation does.
Under either ruling: the format records whatever outcome occurred —
invent no outcome category in the format ahead of the SPEC.

### State model (continuum-runtime's open decision)
No direct effect here: this format is stored, append-only record by
construction. Indirect effect only — if a trajectory-only ruling ever
amended beam-quilt P5, the streams this format defines would have less to
record from that runtime. Nothing in SPEC.md anticipates or accommodates
that branch; do not design for it from here.

## Conflicts found against the verified foundations

1. **SPEC D1 vs the LTTng adaptation.** D1 requires the full-fidelity
   window to be "the same for the whole stream." LTTng's model — the one
   being adapted — makes discard-vs-overwrite policy *per-channel*. If
   per-track policies are ever wanted, D1 must be amended to per-track
   windows declared in the R3 header; as written, the uniform window
   wins and the LTTng policy granularity is the part of the adaptation
   that does **not** transfer. Flagged, not resolved.
2. **Kafka "rebuildable from the tail" vs SPEC D4** — see the strike in
   the compaction item above. The one hard incompatibility in this repo's
   pull list; any future doc importing the compaction contract without
   the strike contradicts D4.
3. **README wording — "can replay it."** The README's framing sentence
   says the record exists so an investigator "can replay it."
   ARCHITECTURE §7 and the foundations are explicit: scrubbing the
   record is settled, re-running the system is speculative/unproven.
   Read "replay" in the README strictly as playback; flagged so the word
   is not quoted into a promise.
4. Builder docs contain **no** refuted claims — the foundations pass
   verified ARCHITECTURE.md:112 and ROADMAP.md:19,24 scope Laman
   correctly. The refuted assertions (sub-millisecond alignment,
   rigid-graph-as-truth, OPAL→MIDI lineage, Kuramoto misstatement) live
   only in `docs/ideation/`, which is annotated provenance: never quote
   ideation prose into builder docs.
