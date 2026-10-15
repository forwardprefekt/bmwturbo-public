# Distinguish hardware evidence from connection assumptions

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This reference separates documented MSV70 hardware, project bench observations, connector assignments, and unresolved electrical interfaces. It is not a connection or programming procedure. The cited build-specific diagrams and bench records must be obtained and reviewed before any physical work; this public summary is not a substitute for withheld artifacts.

## Documented controller hardware

The [MSV70 bench notes](../notes/msv70-ktag-bdm-bench-2026-08-29.md#authoritative-msv70-bdm-facts) cite EVC documentation for these MSV70/MSS70 components:

| Component or interface | Documented identity |
|---|---|
| Processor | Motorola/Freescale MPC563 |
| External flash | AM29BDD160 |
| Serial EEPROM | ST95320 |
| Debug connection | MPC5xx background debug mode, BDM |
| EVC probe | BDM141 |
| ECU supply during the documented BDM procedure | External power through the main ECU connector |

The [identity and memory page](identity-and-memory.md) describes the archived software container. Its size and readable code do not, by themselves, prove a complete recoverable backup of every device or security-dependent state.

## What the bench records establish

The [dated bench record](../notes/msv70-ktag-bdm-bench-2026-08-29.md) documents an opened donor ECU, the ten-pad BDM area, the adjacent enable-jumper location, and available adapter hardware. That record states preparation rather than a completed ECU programming operation.

The later [direct BDM solder map](../notes/msv70-bdm-direct-solder-map.md) records a photograph-specific pad orientation and the observed ground continuity. Its candidate T104 cable-color map still requires cable-level continuity verification. Temporary visual pad numbers are not factory pin numbers, and a similar-looking JTAG/Nexus adapter is not the documented BDM Motorola interface.

**The BDM header is not a 12 V ECU power connector.** Target-VDD sense is a logic-level reference. Do not transfer a pin orientation or cable color from a different tool, board, or photograph without its own electrical verification.

## Connector assignments and the second-pressure-input hypothesis

The [MSV70/MSD80 wiring comparison](../msv70-msd80-pressure-sensor-wiring.md) distinguishes WDS connector assignments from populated PCB input stages:

| X60005 pins | Reviewed MSV70/N52 assignment | Boundary |
|---|---|---|
| 25 / 26 / 27 | MAF supply / signal / ground | Connector assignment, not sensor transfer or ADC characterization |
| 28 | Intake-air-temperature signal | Does not establish post-intercooler charge temperature |
| 31 / 32 / 33 | Manifold differential-pressure sensor supply / ground / signal | Firmware reconstructs a pressure candidate; the name does not imply an absolute boost sensor |
| 9 / 10 / 11 / 17 | Unused in the reviewed MSV70 WDS | Matching N54/MSD80 charge-pressure/temperature cavities are a research lead only |

N54 connector alignment does not establish an MSV70 supply, ADC route, protection circuit, temperature pull-up, or software acquisition behind unused cavities. The N54 production sensor pinout is also not the Bosch PST 3 pinout. Match electrical function using exact source documentation, not shared connector numbers.

The [hardware instrumentation plan](../hardware-io-and-instrumentation.md) and [pressure model](pressure-model.md) preserve the electrical and software requirements separately.

### What an ADC callsite does not identify

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `FUN_00432fc8`, `00433058–00433084`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

The rotate/mask keeps the low 16 bits returned by `0041f184`. With `1f47=0`, the code adds that sample to the 32-bit accumulator `4010` and increments the 16-bit count `2c84`; the nonzero branch selects the other bank. The archive reports a logical acquisition path, not a connector pin, populated input circuit, voltage range, or supported sensor; the listing needed for independent code review is withheld here. The [pressure acquisition account](pressure-model.md#differential-pressure-acquisition) follows the bank into scaling and fallback behavior.

## Sensor and instrumentation evidence still required

A candidate sensor’s published pressure range does not establish compatibility with the DME input. Resolve the following before treating a replacement as usable:

- Supply and reference behavior, input voltage range, impedance, and analog protection
- Sensor identity, transfer function, connector orientation, and temperature dependence
- Filtering, ADC routing, settling, update timing, and fault behavior
- Linearization, plausibility diagnostics, substitute values, and adaptation
- Independent calibrated measurements that can contradict the DME’s model

Fuel-system validation needs rail and manifold pressures on compatible reference scales. Charge-temperature validation needs the temperature entering the engine, not an assumed equivalent compressor-inlet temperature. See [fuel and injection](fuel-and-injection.md) and [logging and validation](logging-and-validation.md).

## Preserve recovery evidence before modification

The retained bench plan makes a verified original backup the prerequisite for development. Keep raw component reads, tool and protocol identity, hardware identity, hashes, and repeated-read comparisons separate from modified outputs. A successful-looking dialog is not a verified backup or a demonstrated recovery path.

Follow the [bench note’s backup prerequisites](../notes/msv70-ktag-bdm-bench-2026-08-29.md#first-read-procedure) and the selected tool’s exact family documentation. The wiki does not authorize a write, cloning operation, security-state change, or cross-revision flash.

## Current unknowns

The repository does not establish every analog input characteristic, replacement-sensor transfer, actual populated second-pressure path, or validated donor recovery procedure. The chronological notes retain their own observation dates and limitations; they do not prove the present physical setup is unchanged.

Related pages: [Identity and memory](identity-and-memory.md), [pressure model](pressure-model.md), [logging and validation](logging-and-validation.md), and [open questions](open-questions.md).
