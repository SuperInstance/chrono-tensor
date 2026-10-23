# Prose-alignment report — chrono-tensor ideation vs the verified foundations

Branch `prose-alignment`, 2026-10-23. Method: grep the repo for the refuted/scoped
claim patterns and their paraphrases, then annotate each occurrence in the ideation
prose with a dated blockquote note — annotation, not erasure, per the programme
convention. Corrected facts and verdicts are taken from beam-quilt `FOUNDATIONS.md`
(verdict table) and its Lane B fact-check (§3.4 for Kuramoto); beam-quilt's own
sibling-repo scan ("chrono-tensor — builder docs already correct; ideation carries
the overclaims") lists the load-bearing lines, which this pass matched and extended
to the retry-branch duplicates of the same claims.

## Result in one line

All refuted/scoped claims live in `docs/ideation/` only; the three ideation files
now carry 39 annotations (3 file-wide header notes + 36 per-passage notes, 79
inserted lines, 0 deletions). Builder docs verified correct and untouched. No
normative-doc flags.

## Builder docs — verified, not assumed

- **ARCHITECTURE.md** — correct. §7 scopes Laman ("rigidity is never a test of
  truth"), Kuramoto, Hodge, A₂/lattice routing, microsecond re-simulation, and
  cross-node tick alignment. No changes.
- **SPEC.md** — correct. Testable rules only; no refuted claims. No changes.
- **README.md** — correct. Explicitly disclaims the continuum metaphysics and
  marks microsecond-rewind/lattice-routing speculative. No changes. (Its "can
  replay it" wording was already flagged by FOUNDATIONS-APPLICATION.md item 3,
  with "read strictly as playback" — a pre-existing flag, not repeated here as new.)
- **ROADMAP.md** — correct. Step 3 scopes rigidity as an under-specification
  heuristic; "Not scheduled" parks the physics vocabulary. No changes.
- **FOUNDATIONS-APPLICATION.md** — correct; it is itself the application of the
  foundations pass. No changes.

## Files touched

Only `docs/ideation/gemini1.md`, `docs/ideation/gemini2.md`, `docs/ideation/gemini3.md`.
Line numbers below are post-annotation; each annotation sits immediately after the
quoted passage (after the closing fence, for the two code-tree comments).

### docs/ideation/gemini1.md — 11 annotations

| Line (note) | Claim annotated (quoted) | Correction |
|---|---|---|
| 5 | *(file-wide)* Kuramoto phase-locking, Hodge decomposition, Eisenstein A₂ routing/optimality used as mechanisms | MISLEADING-SCOPE: each names a heuristic (cadence adjustment, three-bucket triage, a recording grid); no oscillators coupled, no decomposition evaluated (ARCHITECTURE.md §7) |
| 213 | "(sub-millisecond serialization)" — `transport/encoder.py` tree comment | Refuted as MIDI-derived: one 3-byte message at 31.25 kBaud (10-bit framing) ≈ 0.96 ms wire time; ~65 events/s/track |
| 269 | "**near-zero overhead, lightning-fast serialization, and native sub-millisecond temporal alignment**" | Sub-ms refuted (≈0.96 ms/message; best MIDI sync is 8.33 ms MTC quarter-frame); "near-zero overhead" misleading-scope (1-byte status header only; no checksums/sequence numbers) |
| 339 | "*Truth is mathematically defined as a rigid graph*; under-constrained structures collapse instantly under tension" | Refuted: Laman is a 2D bar-joint result (no 3D analogue known); rigidity is geometric determinacy, not a truth predicate |
| 362 | "When coupling density ($K$) exceeds local variance ($\omega_i$), phase angles collapse…" | Misstates the model: ωᵢ is a natural frequency, not a variance; threshold is K > K_c set by g(ω) (Lane B §3.4) |
| 481 | "mathematically defining structural validity (\"Truth is a rigid graph\")" | Same refutation; determination ≠ truth |
| 499 | "When coupling density ($K$) exceeds local variance, nodes spontaneously collapse…" | Same misstatement (second occurrence) |
| 512 | "sub-millisecond binary serialization layer" (`transport/bus.py`) | Refuted as MIDI-derived (≈0.96 ms/message) |
| 553 | "sub-millisecond binary transmission envelopes" | Refuted as MIDI-derived (≈0.96 ms/message) |
| 570 | "truth is mathematically defined as a rigid graph" (CTC-v1 §2.B) | Same refutation |
| 582 | "When coupling density ($K$) exceeds local variance, phase angles collapse…" | Same misstatement (third occurrence) |

### docs/ideation/gemini2.md — 12 annotations

The whole essay depends on the refuted OPAL→MIDI lineage, so the header note
carries that verdict file-wide; per-passage notes mark the load-bearing spots
(FOUNDATIONS names :93, :111, :128, :144, :149, :188, :240, :296 pre-annotation).
The file contains a verbatim-duplicated retry branch; both copies of each
duplicated passage are annotated identically.

| Line (note) | Claim annotated (quoted) | Correction |
|---|---|---|
| 5 | *(file-wide)* the essay's OPAL→MIDI lineage thesis | REFUTED: MIDI's documented lineage is Oberheim/Roland DCB → 1981 Universal Synthesizer Interface → 1983 spec; no PLATO/CERL/OPAL connection in any source; converged, not inherited. Kuramoto/Hodge/Laman/A₂ scoped to heuristics |
| 97 | "from PLATO and TUTOR … through … OPAL, straight into the 1983 MIDI standard" | Lineage refuted (see above) |
| 109 | "parsed in just 1 or 2 clock cycles via native barrel shifts and bitmasking" | REFUTED: TUTOR was interpreted; CDC 6600's fastest functional unit took 3 cycles; "barrel" is the I/O round-robin. Same passage scoped: Empire 30-player is Empire IV by 1977; Avatar's "60" is system-wide, not per-dungeon |
| 120 | "OPAL …, integrated directly into TUTOR" + "When MIDI 1.0 was formalized …, it inherited this exact philosophical blueprint" | OPAL-inside-TUTOR misleading-scope (separate language, own compiler; TUTOR interpreted; 60-bit words never left the host); "inherited" refuted |
| 138 | "native OPAL … integration directly onto your modern repositories … reinvent a wheel" (ancestry assertion) | Ancestry refuted; convergence, not inheritance |
| 163 | "OPAL, where a TUTOR script compiled a musical score…" (§II "OPAL to MIDI") | TUTOR-script claim overstates; section's lineage framing refuted |
| 197 | "extracted in 1–2 clock cycles using hardware bitmasks" | Refuted; figure has no source |
| 205 | "It is the modern reincarnation of the OPAL protocol" | Refuted lineage framing; converged independently |
| 258 | retry-branch duplicate of the ancestry paragraph | Same refutation |
| 283 | retry-branch duplicate of "a TUTOR script compiled a musical score" | Same refutation |
| 313 | retry-branch duplicate of "extracted in 1–2 clock cycles" | Same refutation |
| 321 | retry-branch duplicate of "reincarnation of the OPAL protocol" | Same refutation |

### docs/ideation/gemini3.md — 16 annotations

| Line (note) | Claim annotated (quoted) | Correction |
|---|---|---|
| 5 | *(file-wide)* CTC-v1/v2 use Kuramoto, Hodge, A₂, Laman as operating mechanisms | MISLEADING-SCOPE: heuristics/recording conventions only; rigidity never a test of truth (ARCHITECTURE.md §7) |
| 15 | CTC-v2 thesis "fuses … Sherwin Gooch's OPAL serial data streams" into the architecture | Lineage refuted: converged, not inherited |
| 65 | "*Truth is mathematically defined as a rigid graph*" (CTC-v2 pillars) | Refuted: Laman 2D-only; rigidity ≠ truth |
| 72 | "When coupling density exceeds variance, phase angles collapse into a unified frequency" | Misstates the model: threshold is K_c set by g(ω), not a variance (Lane B §3.4) |
| 139 | "**Truth is mathematically defined as a rigid graph**" (CTC-v1) | Same refutation |
| 220 | "sub-millisecond event bus" ("operational workflow" variant) | Refuted as MIDI-derived (≈0.96 ms/message; ~65 events/s/track) |
| 236 | "1–2 CPU clock cycles, mirroring the 60-bit word optimization of CDC Cyber mainframes" | Historical basis refuted (TUTOR interpreted; 3-cycle fastest unit); mirrored claim inherits an unsourced figure |
| 255 | "*Truth is mathematically defined as a rigid graph*" ("floppy logic" variant) | Same refutation |
| 313 | "sub-millisecond event bus" ("operational workspace" variant) | Same refutation |
| 329 | "1–2 CPU clock cycles" (short variant) | Same refutation |
| 348 | retry-branch duplicate of the "floppy logic" passage | Same refutation |
| 403 | "sub-millisecond binary event bus" (developer guide) | Same refutation |
| 414 | "1–2 CPU clock cycles" (prose §II.2) | Same refutation |
| 427 | "*Truth is mathematically defined as a rigid graph*" (`counterpoint-engine` variant) | Same refutation |
| 459 | "(sub-millisecond serialization)" — `transport/bus.rs` tree comment (note after the fence) | Same refutation |
| 521 | "because truth is mathematically defined as a rigid graph, under-constrained logic collapses…" | Same refutation |

## Patterns with no occurrences in this repo

- **Dec-POMDP global-optimality language** — zero hits (`POMDP`, `NEXP`, `partially
  observable`, `globally optimal`, `optimal polic`, `optimal solution`). The
  refuted Dec-POMDP claim lives in other programme repos, not here.
- **Procrustes** — zero hits.

## Coverage decisions (judgement calls, recorded for review)

- **Kuramoto/Hodge/A₂ beyond scope** is the framing of all three files, not a
  passable passage count — covered by the three file-wide header notes per the
  brief's whole-file rule. The *specific* Kuramoto threshold misstatement and the
  "truth is a rigid graph" sentence got per-passage notes because they are discrete
  factual errors a reader could quote.
- **"Scrub backward to any microsecond" / rewind-and-re-simulate claims** (gemini1,
  gemini3, several places) were **left unannotated**: they are not in the refuted
  list — ARCHITECTURE.md §7 already scopes them as *speculative* (scrubbing the
  record is settled; re-running the system is unproven), and the README repeats
  that scoping. Annotating them would have blurred the refuted/speculative line.
- **Empire/Avatar and "near-zero overhead"** (MISLEADING-SCOPE per FOUNDATIONS, but
  not in the brief's six patterns) were folded into a note only where they share an
  already-annotated passage (gemini2:109; gemini1:269). Empire/Avatar-only passages
  (e.g. gemini2's "30 concurrent players" intros) were left alone.
- **Verified claims left untouched**: TUTOR common blocks, FAA-2006, GSW/GCS 4/16
  voices — all VERIFIED by FOUNDATIONS; no annotation needed.

## Normative-doc flags

**None.** No document a builder would follow repeats a refuted claim as
instruction. The only borderline wording — README's "can replay it" — is already
flagged and scoped by FOUNDATIONS-APPLICATION.md item 3 (pre-existing, unchanged).
