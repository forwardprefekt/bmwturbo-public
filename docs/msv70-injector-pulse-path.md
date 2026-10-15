# MSV70 cylinder-mass to injector-pulse path

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Evidence boundary

This dossier is the static issue #36 trace for dataset `msv70:sha256:757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`. It uses the reviewed manifest, pinned Ghidra 12.0.4 focused output, canonical listing/p-code facts, and exact XDF storage components. XDF names and units establish calibration intent; they do not by themselves name RAM, prove a runtime branch, or establish hardware adequacy.

The result **does not select an injector**, authorize calibration changes, establish that injector duty is safe, or prove the hardware driver latch. It traces the main-pulse software calculation through the six per-cylinder halfwords based at `ram:r13:30f6`, then through staging, timer-count conversion, saturation, packing, and software control-register dispatch. The energized injector timing/current and physical duty margin remain explicit unknowns.

## Exhaustive cylinder-mass coordinate inventory

Canonical RAM facts contain exactly 80 direct two-byte `lhz` reads of `ram:r13:2ada` across 35 functions. `analysis/classifications/msv70-injector-path.jsonl` (supporting artifact withheld from public edition) preserves one row per instruction. Only consumers with reviewed instruction dataflow are given a bounded role; the remainder are `explicit_unknown`. The operational writer is `func:00434c98` at `insn:00434d64`; `func:00140b88` at `insn:00140bd8` is a separate initialization/reset zero store.

## Reviewed main-pulse path

1. **Cylinder-mass coordinate.** `00434c98` reads adjusted load `6ab6` at `00434d50` and writes `2ada` at `00434d64` through rounded, uint16-saturating `0040b46c`, using `c_maf_ref` (`0x844bbe`) and denominator 10000. The adjacent pressure-side operation is `6c6a * 0x79 / 0x101 -> 2aca`; it does not copy pressure into `2ada`. Engineering units and the runtime-selected load producer remain separate questions.
2. **Lambda/enrichment/controller corrections.** `0046e3e8` is a separately reviewed cylinder-mass-indexed base-lambda selector. On the contiguous main-pulse path, `0046d598` copies raw bank controller states `41e6/4332` into `3dda/3f2a`, which `00457f68` consumes while assembling requested mass. Separately, `004584f4` writes mapped correction `2d92`, also consumed by `00457f68`. These exact raw bridges are proven; their upstream physical lambda identities and runtime-active selection remain explicit unknowns.
3. **Base requested fuel mass.** With its gate open, `00457f14` combines `2ada` with `c_mff_fac` (`cal:runtime:00844bd2`, XDF intent 1/14.7) and writes `ram:r13:2d6c`.
4. **Standard-flow conversion and wall film.** `00458918` combines `2d6c` with `c_fac_mff_ti_stnd_1` and writes `2da4`. `00458a44` applies selector-dependent fast/slow and positive/negative wall-film branches and writes signed `2daa`; `004588e4` scales it into `2da2`. The exact pointer-selected calibration family active at runtime remains unknown.
5. **Per-bank and per-cylinder requested mass.** `00457f68` combines base mass, reviewed bank factors, enrichment/correction inputs, and the wall-film correction into bank values `2d6e/2d70`. `00459738` applies operation-mode and per-cylinder factors and writes the indexed requested-mass inputs consumed by `0043d02c`.
6. **Injector differential-pressure correction.** `0043cc84` forms the bounded subtraction of modeled rail `2b24` and modeled manifold `2aca>>5`. `0046ddf4` constructs rail pressure from calibrated nominal pressure, ambient and a fuel-demand correction, not a rail ADC. `ip_fac_ti_prs` and the additional fuel-temperature-intent factor produce `3228`; physical rail sag/boost correspondence remain unmeasured.
7. **Voltage/dead-time compensation.** Logical ADC `0x17`, its eight-sample ring, `00474174` and `00474b38` produce `1fb4`. `0043cce0` evaluates voltage-only `ip_ti_add_dly`, subtracts bias `32768`, adds signed temperature/mass correction `7496`, and writes `323a`. The engineering time domain is 4 µs/count; actual injector-terminal voltage and driver behavior remain unmeasured.
8. **Pulse calculation and constraints.** `0043d02c` multiplies per-cylinder requested mass by bank standard-flow, pressure, and cylinder factors, adds signed dead time/corrections, and stores preliminary values. It records `3238` as the maximum preliminary value, not the final pulse. It then either adds a uniform delta to meet `ip_ti_min`, copies the preliminary values, or applies the manual override.
9. **Final software timer command.** The **final commanded injection time** is a selector-contingent converted per-cylinder timer-count and packed dispatch path, not by `30f6..3100` alone. `30f6..3100` are the final main-pulse *calculation* outputs, not the timer command. `0043d6fc` copies them into per-cylinder staging (`3158`, `3170`, `3188`, `31a0`, `31b8`, `31d0`) only when the corresponding byte selector `320e..3213` is zero; otherwise it stages an alternate source. `0043d760` converts each selected staged value, clamps it to `0xffff`, and writes branch-contingent timer-count candidates `4d54`, `4da4`, `4df4`, `4e44`, `4e94`, `4ee4`. `004271c8` then evaluates record offsets `+0x46`, `+0x47`, `+0x48`, `+0x4a`, and `+0x4d`, which can select another record word, modify the packed value, or return without dispatch. The reviewed branch can dispatch through the `0x704000/0x704400` control-register regions; runtime selector state and the energized physical result are not measured.

## Pressure semantics inherited from issue #35

**Injector differential pressure** is **rail pressure minus manifold absolute pressure**. That is the physical validation contract required by issue #36 and consistent with the **issue #35 pressure boundary**; issue #35 does not itself prove a rail-pressure signal. It is what controlled validation must measure. The internal `ip_fac_ti_prs` subtraction does not independently prove that either raw RAM operand is rail pressure or manifold pressure. An **assumed constant rail pressure** is therefore not accepted, and no calibration label is promoted into a raw-state alias without instruction evidence.

### Offline equation closure

The [source-bound equation report](../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json)
resolves standard-flow arithmetic and 58 selector/pointer words. Scalar
`c_fac_mff_ti_stnd_1` at `844ac0` is **not** bank array element
`c_fac_mff_ti_stnd[1]` at `845aae`; their paths use different binary divisors.
Under the explicitly stated mass/time engineering convention, the six-decimal
XDF coefficient slopes lose about 4.18% relative to dimensional closure.
Use the exact documented arithmetic, not rounded display slopes, for any
future encoding design.

The stock pressure axis covers approximately 5–6 bar differential and holds
the first factor below its domain. The latency map has no pressure coordinate.
The minimum-pulse operation applies a uniform uplift to selected cylinders;
it is not a nonlinear short-pulse correction curve. These are mapping
constraints, not supplied injector values or authorization to flash.

## Characterization inputs and limits

The reviewed path identifies calibration intent for standard flow, pressure correction, dead-time latency, wall film, minimum pulse, and manual override. It preserves these gaps:

- the runtime-active lambda/enrichment and pointer-selected wall-film branch must be observed rather than inferred from stock values;
- physical voltage at the injector and its driver-specific latency remain unmeasured;
- rail and manifold operands are modeled; actual pressure and boosted correspondence are unproven;
- a maximum commanded pulse calibration was not recovered;
- physical injector flow, coil current, latency under real rail pressure, and hardware driver behavior are outside firmware-only proof; and
- EOI/SOI phase feasibility, 16-bit representation, arithmetic saturation, table-axis endpoints, and physical duty cycle are different limit classes.

Consequently, issue #39 must synchronously capture the six software pulse values, RPM, battery voltage, rail pressure, independent absolute MAP, independent wideband lambda, and relevant temperature channels. Correlation may falsify a hypothesis but cannot create a static dataflow edge or establish hardware safety.
