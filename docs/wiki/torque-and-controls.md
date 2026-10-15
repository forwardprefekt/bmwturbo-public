# Torque and controls

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page summarizes what the canonical AC1S MSV70 image proves about functional torque modeling, independent torque monitoring, and actuator-intervention software. The recovered paths are conditional software calculations; they do not establish measured crankshaft torque, active runtime selectors, diagnostic response, or physical actuator behavior. Source detail: [torque-authority dossier](../msv70-torque-authority-dossier.md).

## Three distinct lanes

| Lane | Reviewed path and useful states | What it establishes | What it does not establish |
|---|---|---|---|
| Functional request/model | Driver mapping writes `r13:2f02`; gated arbitration selects `2f04`, then representation states `2f06/2f08`. A branch-contingent continuation reaches arbitration inputs `2d14`, `6258`, and bounded raw output `66d6`. | Driver demand conditionally participates in model/filling calculations. | A universal winning request, Nm interpretation of raw RAM values, or measured torque. |
| Independent monitor | Two schedulers compare reference and monitored driver-request copies. First path computes difference `5790`, threshold `5791`, and latched result `2182`; second has a separate reference/threshold and debounced bit 6 outcome. | Independent comparisons and debounce logic exist. | Full diagnostic consolidation, active enable state, or monitor-fault causality. |
| Actuator intervention | Periodic `000dc15c` uses `218c` and `ip_tqi_dif_lim` relative to `2e6c`, producing ramped limits `2f30/2f32`. Converted states `6292/6290` conditionally clamp model-domain calculations; a branch-contingent chain can reach bounded software throttle request `2e2a`. | A distinct limiter and a software throttle request path exist. | A monitor-fault trigger, actual throttle plate position, PWM, or physical torque intervention. |

The scheduler converts prior limiter values before the current limiter calculation: this is loop-carried state, not a same-invocation handoff. The monitor flag is not an input in the reviewed limiter path; no monitor-fault-to-intervention edge was recovered. A shared scheduler or nearby state is not proof of causal coupling.

## Torque model and map roles

The reviewed model interpolates `KF_MDIOP_1` and `KF_MDIOP_2` using RPM and adjusted load, then selects a modeled result. `KL_MDMAX` is a distinct model ceiling, not the selector or index for either MDIOP family. Driver demand also contributes to a descriptor-driven blend selector. These are model/filling coordinates, not measured crankshaft torque. The [torque dossier](../msv70-torque-authority-dossier.md#functional-torque-lane) retains the selector and consumer evidence; the [interpolation layout](../msv70-interpolated-table-layout.md) identifies map storage.

### Model candidates are selected, not measured

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `FUN_000bca44`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

The descriptors `0x859814` and `0x859a56` identify the [two MDIOP layouts](../msv70-interpolated-table-layout.md#optimal-torque-calculation-fun_000bca44). Each candidate update has its own selector gate; the subsequent selector helper is a separate operation. The archive reports fixed-point scaling with a correction for negative values before storing a `short`. None of these operations measures physical torque.

### The monitor compares a separate pair of states

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `FUN_000878d8`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

Here `0x7fa180/0x7fa18c` are the reference/monitored request states (`r13:2180/218c`), not the MDIOP candidate stores. The byte-sized difference and threshold (`0x7fd790/0x7fd791`, or `r13:5790/5791`) feed a strict threshold comparison and debounce call. A breach alone does not set the result: the debounce helper must also return nonzero. The surrounding reconstruction (supporting artifact withheld from public edition) retains complemented-state checks, the enable branch, and the final latched store at `0x7fa182` (`r13:2182`). This reported path describes a monitor calculation, not a diagnostic or actuator response; its source-level proof is withheld here.

### Other control boundaries

The reviewed Valvetronic path stops at a bounded software setpoint. `00455110` reads `6836`, applies its gated correction and lower/upper selection, and writes `694c`. That establishes neither measured valve lift nor motor actuation. See the [actuator-intervention boundary](../msv70-torque-authority-dossier.md#actuator-intervention-lane).

`r13:65ac` is read signed in the reviewed VANOS eligibility path and can set/clear `r13:6e14` bit 2 with RPM and threshold gates. Its defensible classification is selected-relative-filling eligibility only: no MDIOP/MDMAX indexing or commanding of VANOS target maps, cam command, or measured cam position is established.

## How to read authority claims

| Evidence level | Safe interpretation |
|---|---|
| Instruction/decompiler path | Listed producers, consumers, gates, signedness, and conditional control flow for the canonical image. |
| Calibration definition | XDF identifies storage and intended parameter; a title alone cannot prove a branch is active. |
| Runtime | Selector population, diagnostic reaction, and branch/timing in the vehicle remain unobserved. |
| Physical | Software request is not measurement of torque, throttle angle, Valvetronic lift, cam position, or hardware actuation. |

## Open questions

- Which functional arbitration branches and model selectors are populated under real operating conditions?
- How are monitor outcomes consolidated into diagnostic reporting, and what are their enable states?
- Are monitor outcomes connected to any intervention outside the specifically reviewed paths?
- What independent measurement validates torque, and what hardware stages turn software throttle/Valvetronic requests into physical response?
- What are the actual intake/exhaust cam positions and runtime VANOS target paths?

## Related pages

- [Calibration and limits](calibration-and-limits.md)
- [Airflow and load](airflow-and-load.md)
- [Pressure model](pressure-model.md)
- [Fuel and injection](fuel-and-injection.md)
- [Ignition, lambda, and protection](ignition-lambda-and-protection.md)
- [Open questions](open-questions.md)
