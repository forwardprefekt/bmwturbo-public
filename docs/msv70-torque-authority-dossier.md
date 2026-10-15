# MSV70 torque authority, monitoring, and intervention dossier

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Scope and evidence boundary

This dossier is a static, manifest-bound description of the stock MSV70 binary. It separates a **Functional torque lane**, an **Independent monitor lane**, and an **Actuator intervention lane**. It does not identify measured torque, prove a runtime selector state, establish a monitor-fault-to-intervention trigger, command hardware, or establish a physical safety margin. Tune 29 and NoMAF changes are clues only. This dossier does not approve boosted torque values, monitor bypass, or operation outside stock validated conditions.

The focused evidence is bound to `analysis/manifests/msv70-analysis-targets.json` (supporting artifact withheld from public edition) and `analysis/derived/focused/generation.json` (supporting artifact withheld from public edition). RAM interpretation is per-access signedness: `lha` is a signed read, `lhz`/`lbz` are unsigned or raw-bit reads, and stores alone do not assign read signedness. XDF titles identify calibration intent and storage; they do not, by themselves, prove that a runtime branch is active.

## Functional torque lane

`FUN_000a4a00` maps the driver input into `ram:r13:2f02` under explicit mode and hold gates (`analysis/derived/focused/decompiled/000a4a00.c` (supporting artifact withheld from public edition), lines 18-79). `FUN_000a4c74` conditionally chooses the greater of `ram:r13:2f02` and `ram:r13:2efe` into `ram:r13:2f04`, applies the representation boundary into `ram:r13:2f06`, and optionally combines that state with `ram:r13:2f00` into `ram:r13:2f08` (`analysis/derived/focused/decompiled/000a4c74.c` (supporting artifact withheld from public edition), lines 18-39; `insn:000a4c8c` through `insn:000a4d08`). This is conditional arbitration, not a universal final-torque authority.

One branch-contingent continuation is instruction-backed. `FUN_00475230`
uses `ram:r13:2f04` among several gated inputs while updating
`ram:r13:2d14`; it is not the only input or the only branch
(`insn:00475420`, `insn:00475644`). `FUN_0046bf80` combines
`ram:r13:2d14`, `ram:r13:2d12`, and another raw state into
`ram:r13:6258` (`insn:0046c080`, `insn:0046c09c`). `FUN_000be77c`
then admits `ram:r13:6258` as one contributor to its max/min request
arbitration and publishes the bounded raw result `ram:r13:66d6`
(`insn:000be860`, `insn:000be88c`, `insn:000be8a0`,
`insn:000be8a8`). This proves conditional participation, not that the
driver request wins arbitration in every mode or that these raw states are Nm.

The stock model has two separately selected inverse-model families, `KF_MDIOP_1` and `KF_MDIOP_2`. `FUN_000bca44` indexes their descriptors from RPM and adjusted load, writes separate candidates, and selects the model result (`analysis/derived/focused/decompiled/000bca44.c` (supporting artifact withheld from public edition), lines 20-40). `FUN_0046dab4` exports that modeled candidate into `ram:r13:2af0` (`analysis/derived/focused/decompiled/0046dab4.c` (supporting artifact withheld from public edition), lines 13-17). `KL_MDMAX` is a distinct model-ceiling family used by `FUN_000b55a4`; it is not the selector or index for `KF_MDIOP_1`/`KF_MDIOP_2`.

The driver request is also converted into `ram:r13:62a0` by `FUN_0046c5d4`, and `FUN_000bcb2c` uses that state as a descriptor-driven blend selector in its filling calculation (`analysis/derived/focused/decompiled/0046c5d4.c` (supporting artifact withheld from public edition), lines 26-39; `analysis/derived/focused/decompiled/000bcb2c.c` (supporting artifact withheld from public edition), lines 181-215). The recovered path therefore ends in a conditional software model/filling state, not measured crankshaft torque.

## Independent monitor lane

The first monitor scheduler independently computes a reference and monitored driver request before comparison: `FUN_00082904` calls `FUN_00087644`, `FUN_00088ba0`, and then `FUN_000878d8` (`analysis/derived/focused/decompiled/00082904.c` (supporting artifact withheld from public edition), lines 43-51). `FUN_000878d8` checks complemented state, computes the absolute difference between `ram:r13:2180` and `ram:r13:218c` into `ram:r13:5790`, computes the calibrated threshold into `ram:r13:5791`, debounces a breach, and stores the latched result at `ram:r13:2182` (`analysis/derived/focused/decompiled/000878d8.c` (supporting artifact withheld from public edition), lines 24-57; `insn:00087940`, `insn:00087944`, `insn:00087950`, `insn:00087974`, `insn:0008798c`, and `insn:000879e8`).

The second monitor lane is independently scheduled by `FUN_000828d0`. `FUN_0008909c` builds its monitored driver-request copy; `FUN_000892cc` builds a separate reference using the runtime storage corresponding to `ip_tqi_ref_mon2`; and `FUN_00089474` compares the two, applies the `ip_tqi_dif_max_mon2` threshold, and debounces bit 6 of its outcome byte (`analysis/derived/focused/decompiled/000828d0.c` (supporting artifact withheld from public edition), lines 14-20; `analysis/derived/focused/decompiled/0008909c.c` (supporting artifact withheld from public edition), lines 28-84; `analysis/derived/focused/decompiled/000892cc.c` (supporting artifact withheld from public edition), lines 21-52; `analysis/derived/focused/decompiled/00089474.c` (supporting artifact withheld from public edition), lines 17-32).

These monitor results are endpoints of the reviewed comparison paths. Their diagnostic consolidation and runtime enable states remain an explicit unknown. No recovered instruction connects either monitor-fault result to the intervention outputs below, and monitoring is required evidence—not a candidate for disabling or bypassing.

## Actuator intervention lane

`FUN_000dc15c` is a separate, periodic functional-limit calculation. It uses the independently calculated driver factor `ram:r13:218c` with `ip_tqi_dif_lim`, compares the result with `ram:r13:2e6c`, maintains its own sampled intervention state, and produces ramped limits `ram:r13:2f30` and `ram:r13:2f32` (`analysis/derived/focused/decompiled/000dc15c.c` (supporting artifact withheld from public edition), lines 18-104). It does **not** consume either reviewed monitor-fault result. Calling this a monitor-fault response would invent causality.

`FUN_0046c5d4` converts those two limits into model-domain states `ram:r13:6292` and `ram:r13:6290` (`analysis/derived/focused/decompiled/0046c5d4.c` (supporting artifact withheld from public edition), lines 34-37). `FUN_000b9bc4` uses them as conditional upper clamps, including the `ram:r13:6290` clamp that selects `ram:r13:64c4` (`analysis/derived/focused/decompiled/000b9bc4.c` (supporting artifact withheld from public edition), lines 44-53, 199-205, and 280-293). The scheduler calls the conversion before the current invocation of `FUN_000dc15c`, so this connection is loop-carried rather than a same-invocation handoff (`analysis/derived/focused/decompiled/000aca84.c` (supporting artifact withheld from public edition), lines 19-22 and 61-65).

The selector-contingent throttle path is:

1. `ram:r13:6290` limits `ram:r13:64c4` in `FUN_000b9bc4`.
2. `FUN_000bd0b0` uses that limited state in the selected inverse-model calculation ending at `ram:r13:65ac` (`analysis/derived/focused/decompiled/000bd0b0.c` (supporting artifact withheld from public edition), lines 41-60 and 89-105).
3. `FUN_000c4100` derives filling states from `ram:r13:65ac` (`analysis/derived/focused/decompiled/000c4100.c` (supporting artifact withheld from public edition), lines 55-105).
4. One guarded branch of `FUN_000c5358` admits that candidate, while other selector outcomes substitute other sources; its delayed output is converted by `FUN_0046c4dc` into `ram:r13:2a72` (`analysis/derived/focused/decompiled/000c5358.c` (supporting artifact withheld from public edition), lines 35-58 and 365-394; `analysis/derived/focused/decompiled/0046c4dc.c` (supporting artifact withheld from public edition), lines 13-17).
5. `FUN_000a0828` priority-selects, bounds, and rate-limits the candidate into the software throttle-controller request `ram:r13:2e2a` (`analysis/derived/focused/decompiled/000a0828.c` (supporting artifact withheld from public edition), lines 30-91).

That endpoint is a bounded software request, not measured throttle position, plate angle, PWM, duty cycle, or physical actuation. `FUN_004530a8` only proves that further controller stages are scheduled (`analysis/derived/focused/decompiled/004530a8.c` (supporting artifact withheld from public edition), lines 11-19).

The Valvetronic/VVL review stops at a manifest-bound bounded setpoint boundary.
`FUN_00455110` reads `ram:r13:6836`, applies its guarded correction and
lower/upper selection, and retains the bounded raw result at `ram:r13:694c`
(`insn:0045555c`, `insn:004555e0`, `insn:004555f4`, and `insn:004555fc`;
`analysis/derived/focused/decompiled/00455110.c` (supporting artifact withheld from public edition), lines 145-176). This does not
prove measured Valvetronic lift or motor actuation.

## VANOS boundary

`ram:r13:65ac` is read with signed `lha` operations at `insn:000c096c` and `insn:000c0998`. Together with RPM and two thresholds, those reads set or clear `ram:r13:6e14` bit 2; the bit then participates in a later eligibility decision (`analysis/derived/focused/decompiled/000c093c.c` (supporting artifact withheld from public edition), lines 25-30 and 74-80). The safe classification is `selected_relative_filling_to_vanos_enable_gate`.

This is a narrow eligibility-gate bridge only: there is **no direct MDIOP/MDMAX indexing or commanding of VANOS target maps**. It is not a torque-indexed VANOS target, a cam-position command, or evidence of measured intake or exhaust VANOS position.

## Limits and deferred validation

Numeric representation boundaries, arithmetic saturation, calibration-axis endpoints, helper endpoint holds, diagnostic plausibility thresholds, actuator limits, and physical-safety limits remain separate categories. A software maximum is not automatically a hardware, durability, emissions, or boosted-safety maximum.

The five issue #37 rows in `analysis/validation/msv70-dynamic-observations.jsonl` (supporting artifact withheld from public edition) are deferred issue #39 work. Every row has `execution_status=not_run`; correlation cannot upgrade an alias, choose a static branch, prove causality, or establish safety. Runtime selector state, diagnostic reaction, actual throttle position, actual Valvetronic lift, intake and exhaust VANOS position, and independent torque remain future instrumented acceptance work.
