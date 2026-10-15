# MSV70 selected-airflow consumers and filling producers

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Scope and authority

This dossier resolves issue #34 as an evidence-boundary review. It changes no
firmware or calibration. Direct accesses come from the pinned whole-program
`analysis/facts/msv70/ram-accesses.csv` (supporting artifact withheld from public edition); the original reviewed expressions use
the manifest-bound Ghidra 12.0.4 focused generation. The continuation also uses
the retained full instruction exports under
`analysis/derived/review-2026-09-04-pressure-continuation/` (supporting artifact withheld from public edition). The reviewed classifications
are in
`analysis/classifications/msv70-airflow-filling-boundary.jsonl` (supporting artifact withheld from public edition), and unresolved
physical names transfer to
`analysis/validation/msv70-dynamic-observations.jsonl` (supporting artifact withheld from public edition) for issue #39.

The access census is exact: selected airflow `ram:r13:2b30` has **35 reads
across 23 functions**, plus one initialization-or-reset store and one gated
operational selector store. Filling candidates `ram:r13:6b5a` and
`ram:r13:6b9a` have five operational stores in `func:004439b4`. The outer gate
`ram:r13:1fea` has seven byte reads and two byte writes.

## Selected-airflow consumer census

A direct `lhz` proves that a function consumes the stored bits; it does not by
itself establish the function's physical purpose. `func:00435048` has one
narrow reviewed role: when `ram:r13:1f62 == 0`, focused C lines 30–33 copy
`2b30` into raw states `2b28` and `2b26`. The `2b30 -> 2b26` copy is the first
edge of a conditioned raw-state route to adjusted load, but neither raw state's
engineering name is proven. Every other purpose remains explicitly unknown,
even though every function has manifest-bound focused C for review.

| Function | Exact read instructions | Reviewed purpose |
|---|---|---|
| `func:000ca2d4` | `insn:000ca474` | Explicitly unknown |
| `func:000cb754` | `insn:000cb990` | Explicitly unknown |
| `func:000ce958` | `insn:000cebb4` | Explicitly unknown |
| `func:000d4ad8` | `insn:000d4c90` | Explicitly unknown |
| `func:000d4d7c` | `insn:000d4f90`, `insn:000d50a0` | Explicitly unknown |
| `func:000d5338` | `insn:000d5488`, `insn:000d551c`, `insn:000d5540` | Explicitly unknown |
| `func:000d5e08` | `insn:000d5e9c`, `insn:000d5f78` | Explicitly unknown |
| `func:000df4e8` | `insn:000df4fc` | Explicitly unknown |
| `func:000ed574` | `insn:000ed5cc` | Explicitly unknown |
| `func:000edbe4` | `insn:000edcc0`, `insn:000edd18` | Explicitly unknown |
| `func:000edfa4` | `insn:000ee3f4` | Explicitly unknown |
| `func:000ee494` | `insn:000ee7f4` | Explicitly unknown |
| `func:000ef2b8` | `insn:000ef360`, `insn:000ef45c`, `insn:000ef518` | Explicitly unknown |
| `func:000f1224` | `insn:000f125c` | Explicitly unknown |
| `func:001083fc` | `insn:00108508`, `insn:0010854c`, `insn:00108570`, `insn:001086c8`, `insn:001086f4` | Explicitly unknown |
| `func:001088b4` | `insn:00108934` | Explicitly unknown |
| `func:0010d304` | `insn:0010d380` | Explicitly unknown |
| `func:0010d3e8` | `insn:0010d540` | Explicitly unknown |
| `func:0013cee4` | `insn:0013cee4` | Explicitly unknown |
| `func:00435048` | `insn:004350c0` | Reviewed input to the `1f62 == 0` copy |
| `func:0043718c` | `insn:00437608`, `insn:00437cd8` | Explicitly unknown |
| `func:0045da08` | `insn:0045da70` | Explicitly unknown |
| `func:00473228` | `insn:00473250` | Explicitly unknown |

The two stores are deliberately separate:

- `func:00435180` / `insn:004351b4` is the operational selector. When
  `ram:r13:1fea == 0` and `ram:r13:1e8c == 0`, it stores `ram:r13:2ac2` at
  `2b30`. When `1fea == 0` and `1e8c != 0`, it instead stores `2b26`. When the
  outer gate is nonzero, `2b30` retains its prior value.
- `func:00140b88` / `insn:00140bdc` writes zero amid a state-clearing sequence.
  It is **initialization-or-reset**, not boot-only: structured calls show more
  than one invocation site.

Combined with `func:00435048`, the operational edges form a conditioned cycle:
`2b30 -> 2b26` only when `1f62 == 0`, while the return
`2b26 -> 2b30` requires both `1fea == 0` and `1e8c != 0`. Physical
measured-MAF identity and runtime activation are not implied by that cycle.

## Outer-gate writer

Manifest-bound `func:00438d94` recovers both exact byte-wide conditions:

```text
if 1fe2 == 0 or u8(21d8) <= u8(cal:runtime:00844ca4):
    1fea = 0                         # insn:00438dcc
else:
    1fea = u8(21da) & 1              # insn:00438dc0
```

`cal:runtime:00844ca4` resolves to the XDF object `c_n_seg_2_ena`, but that
label does not establish the engineering meaning of `1fe2`, `21d8`, `21da`,
or `1fea`. `func:00440624` calls `00438d94` immediately before `00435048` and
`00435180` under its task gates, establishing scheduling order rather than a
physical alias.

## Filling-candidate derivations

`func:004439b4` never reads `ram:r13:2b30`. Its five direct `sth` sites have reviewed manifest-bound decompiler
expressions. The exact upstream instruction/p-code slices remain explicitly
unknown, so these are not claimed as fully recovered derivations:

- `ram:r13:6b5a` at `insn:004447e4`, `insn:004447f8`, and
  `insn:00444800` stores the lower-clamped, in-range, or upper-clamped result.
  The path selects/scales `6b58` or `6b90`, uses `6bb8`, `6bb4`, `6b72`, and
  `6b68`, forms `6b56` from a calibration lookup and RPM-related state, then
  applies a bounded ratio involving `max(678e - 6b56, 0)` and
  `max(0x654d - 6b56, 1)`. Focused C lines 397–477 retain the arithmetic;
  the classification rows bind each final store instruction.
- `ram:r13:6b9a` at `insn:00444920` and `insn:00444928` stores the in-range or
  upper-clamped result of the positive difference `max(6ac2 - 6b76, 0)`,
  scaled by `6b66` and `6b70`. Focused C lines 498–525 retain the expression.
- Canonical calibration `S_EVNN_AUS` at runtime `0x85e00b` then selects `6b9a`
  or `6b5a` into `ram:r13:6b6a`; that selector does not retroactively connect
  either candidate to selected airflow.

The mismatched structure-730 A2L cannot name these addresses directly, but its
function grouping is independent supporting evidence: 38 `EISY_EV`
calibration anchors project into canonical `func:004439b4`, whose older output
lexicon includes `Rfv_ist_nn`, `Rfv_ist_hub`, and `Rfv_ist`. Thus `6b9a`,
`6b5a`, and `6b6a` are strong neural-network, lift-based fallback, and selected
modeled-unthrottled-relative-filling candidates. These remain candidate names,
not validated units or exact symbols.

## Bridge assessment

The earlier broad statement that no selected-airflow path reaches load is
superseded. A conditioned raw-state route is recovered:
`2b30 -> 2b26 -> 6980 -> 6ac0 -> 6ab8 -> 6ab6`, with `1f62 == 0` required
for the first copy and `CW_EISY` bit 0 set required for the final load producer.
This proves software dataflow only; the physical identity of `2b26` and the
active runtime branches remain unobserved.

The continuation also recovers the narrower **conditional** route:
`2b30 -> 2b26 -> 6980 -> 6ac0 -> 6c6e -> 6c6a -> 6ac2 -> 6b9a`.
The first copy requires `1f62 == 0`; `CW_EISY` bit 1 set selects `6ac0` as
integrator inflow at `00447030`, and bit 2 clear selects integrated pressure
`6c6a` into `6ac2`. These are distinct from bit 0's final-load selection.
`00447004` applies the intervening inflow corrections, signed flow-error
saturation, variable integration gain, and modeled-pressure upper limit;
`004439b4` uses selected pressure in the positive-difference filling equation.
The graph retains the conditioned intermediate edges rather than inventing an
unconditional `2b30 -> 6b9a` shortcut. See the
[source-bound continuation](../analysis/derived/review-2026-09-04-load-continuation/findings.json).

Canonical `CW_EISY=0` disables the bit-1 airflow input. Its integrator instead
uses density-corrected throttle flow `6b4c`, and its upper limit still comes
from ambient-derived modeled pre-throttle pressure. Thus the software bridge
is recovered, but neither runtime activation nor truthful boosted charge is
proven. A complete alternate derivation into `6b5a` is not asserted here.

One tempting function-level false positive was examined and rejected with
instruction-level dataflow. `func:000df4e8` reads `2b30` and also writes globals
later consumed by `func:004439b4`, but its focused C separates the expressions:

- `2b30 -> 7fdc6c -> global:01802fda`;
- `2ce8 -> global:01802da6` independently;
- `2cea` (or zero under `7bb3`) -> `global:01802d96` independently.

The only two structured readers of `global:01802fda`, `func:001021bc` and
`func:0012ad04`, directly write neither the RAM inputs nor the globals read by
`func:004439b4`. This rules out that concrete co-occurrence shortcut, not every
possible deeper, pointer-mediated, or runtime path.

## Resolution and remaining unknowns

The original bounded-search result is superseded by the conditional
airflow/integrator/pressure/filling route above. No GitHub issue state is changed
by this document. Exact access provenance, initialization versus operational
stores, and unproven physical identities remain distinct.

Issue #39's focused baseline now captures 17 states, adding `6b68`, `6ade`, and
`6b62` to validate the canonical normalized/corrected modeled-load producer.
It retains canonical `S_EVNN_AUS=0`, `CW_EISY=0`, and `S_RFST_ON=0`; it does not
change firmware merely to exercise an alternate branch. Independent MAF, MAP,
RPM, throttle, lambda, ambient pressure, and IAT remain required.
Any later test of the bit-1 route needs its own source-bound configuration and
capture of the integrator states and limits. Correlation cannot establish
physical identity, a software edge, or boosted safety.
