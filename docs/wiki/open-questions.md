# Track what remains unproven

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page turns the wiki’s uncertainty into an evidence checklist. It distinguishes missing measurements from missing code relationships and from decisions that depend on both. It does not prescribe a turbo calibration or authorize an experiment.

## Questions that block stronger conclusions

The [findings index](../msv70-findings-index.md) and domain dossiers identify these boundaries:

| Question | What is already known | Evidence needed to resolve it | Knowledge page |
|---|---|---|---|
| Which source and branch does the running DME use? | Canonical selector bytes and conditional producers are recovered | Current image identity plus synchronized selector, state, and update-timing observations | [Airflow and load](airflow-and-load.md) |
| Is the airflow coordinate accurate physical mass flow? | Acquisition, averaging, correction, selection, and representation bounds are traced | Calibrated sensor/housing transfer, raw acquisition behavior, independent airflow, and active-gate evidence | [Airflow and load](airflow-and-load.md) |
| Can a pressure input truthfully represent positive manifold pressure? | Stock conversion is nonnegative differential followed by ambient subtraction; integrated pressure uses a modeled ceiling | Electrical characterization, physical sensor transfer, diagnostic behavior, and a separately reviewed software domain | [Pressure model](pressure-model.md) |
| Is a second pre-throttle pressure/temperature input available? | N54 wiring offers matching connector cavities; reviewed MSV70 WDS marks them unused | PCB population, supply, protection, ADC/temperature acquisition, and exact firmware consumer evidence | [Hardware and recovery](hardware-and-recovery.md) |
| Does charge temperature match the air entering the engine? | Current source-bound model and correction paths are documented | Sensor location/transfer, model validation, and independent post-intercooler measurements | [Airflow and load](airflow-and-load.md) |
| Are the injectors and fuel system characterized in the relevant domain? | Main-pulse equations, parameter identities, pressure-map limits, staging, and timers are recovered | Exact injector data definitions, applicable pressure/voltage coverage, actual rail pressure, latency, flow, and driver behavior | [Fuel and injection](fuel-and-injection.md) |
| Which torque request wins, and what causes intervention? | Functional, monitor, and limiter paths are separately recovered | Live arbitration/enable states, missing causal joins where applicable, and independent torque/actuator measurement | [Torque and controls](torque-and-controls.md) |
| What do protections achieve physically? | Spark, knock, lambda, component-model, and diagnostic software paths exist | Accepted synchronized observations, independent knock/lambda/temperature data, and actuator response | [Ignition, lambda, and protection](ignition-lambda-and-protection.md) |
| How does a single-turbo exhaust affect both lambda channels? | Separate records, histories, and delay-related calibrations remain in the stock software | Actual exhaust/sensor topology, transport delays, bank observability, and controller behavior | [Ignition, lambda, and protection](ignition-lambda-and-protection.md) |
| Is a change recoverable and compatible with this ECU? | Canonical image, map, checksum metadata, and bench preparation are recorded | Tool-specific verified backup/recovery, current hardware identity, and independently established revision compatibility | [Identity and memory](identity-and-memory.md) |

A single missing measurement can affect several rows. Resolving one software relationship must not silently close the corresponding physical question.

## Current validation dependency

The [dynamic-validation protocol](../msv70-dynamic-validation-protocol.md) places acquisition and timing ahead of physical hypothesis testing. Its outstanding work is concrete:

1. Establish current canonical identity and retain the raw diagnostic probe results.
2. Demonstrate read-only grouped or event-driven access to the focused state set.
3. Record logger configuration, state update rates, channel calibration, and fault status.
4. Establish the measured DME/logger clock relationship and uncertainty.
5. Retain baseline and controlled naturally aspirated observations with complete metadata.
6. Assign capture-bound support, contradiction, or unresolved outcomes without inventing causal edges.

The [memory-capture runbook](../msv70-memory-capture-runbook.md) owns the operational sequence. The [hardware page](hardware-and-recovery.md) separately covers recovery and electrical prerequisites; those are not waived by transport access.

## Conclusions that the current evidence rejects

The domain pages explain why the following shortcuts are not valid:

| Shortcut | Missing or contradictory evidence |
|---|---|
| Enable `S_EISY_TURBO` and treat the ECU as boost-ready | The flag changes a request/control normalizer, not the physical pressure input |
| Change only the two pressure-transfer endpoints | Unsigned differential semantics and ambient subtraction remain |
| Treat a NoMAF package as a portable turbo strategy | Revision differences and performance-tune changes are not a pure feature patch |
| Infer headroom from a larger integer type | Earlier table clipping, intermediate arithmetic, map endpoints, and physical limits remain |
| Treat six calculated pulse values as measured injector operation | Alternate selectors, timer conversion, dispatch, and physical driver behavior are separate |
| Treat a diagnostic report as proof of a specific protective intervention | The required causal join may not exist in the reviewed path |
| Accept a capture because a validator reports success | Raw identity, timing, calibration, fault status, and scientific relevance still require review |

## Record a resolution

A resolved item needs a source identity, a bounded claim, the supporting code or capture artifact, and the remaining limitations. Update the nearest wiki page and its underlying dossier or observation record; do not close uncertainty by editing this summary alone.

Related pages: [Evidence and tools](evidence-and-tools.md), [logging and validation](logging-and-validation.md), and the [research backlog](../research-backlog.md).
