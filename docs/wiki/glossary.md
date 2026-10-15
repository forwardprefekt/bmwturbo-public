# Read the MSV70 terminology

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This glossary defines terms used in the wiki without turning candidate names into physical measurements. Keep a raw state ID beside a semantic interpretation when the canonical firmware does not supply an authenticated symbol name.

## Controller, files, and evidence

| Term | Meaning in this wiki |
|---|---|
| DME | BMW’s engine-control unit, from Digital Motor Electronics |
| MSV70 | The Siemens/VDO controller family studied here; conclusions are scoped to the identified AC1S image |
| AC1S / AC3S | Different software revisions within the compared lineage, not interchangeable file labels |
| ROM / full read | Archived firmware bytes; the container includes mapped code, calibration, and non-executable regions |
| BDM | Background debug mode, the MPC5xx low-level debug interface used in the bench research |
| XDF | Calibration-definition file describing object locations, storage, axes, and display conversions |
| A2L | ASAM measurement/calibration description; a different-build A2L can supply candidate vocabulary, not canonical addresses |
| EPK | Software-identification information used when checking whether an A2L belongs to an image |
| SHA-256 | Content digest used to bind an artifact to exact bytes |
| P-code | Ghidra’s intermediate representation of instruction behavior |
| Raw state | An address/width/access identity whose complete physical meaning may remain unresolved |
| `r2` / `r13` | PowerPC registers used as calibration and RAM bases in the canonical analysis layout |

See [identity and memory](identity-and-memory.md) and [evidence and tools](evidence-and-tools.md) for address and authority rules.

## Air, load, pressure, and temperature

| Term | Meaning and limitation |
|---|---|
| MAF | Mass airflow; a physical quantity or sensor description must be distinguished from a firmware coordinate |
| HFM | Hot-film air-mass sensing terminology used in the calibration material |
| IAT | Intake-air temperature; sensor location matters, especially before versus after the compressor/intercooler |
| MAP | Manifold absolute pressure; not interchangeable with differential pressure or gauge boost |
| Absolute pressure | Pressure referenced to vacuum |
| Gauge pressure / boost | Pressure relative to an ambient reference |
| Differential pressure | Difference between two pressures with identified reference points |
| `Ps` | Manifold-pressure terminology projected onto reviewed pressure-model states; not a blanket sensor alias |
| `Pvdk` | Pre-throttle-pressure terminology; reviewed stock states are ambient-derived models, not a proven second sensor |
| `Pirg` | Residual-gas-pressure terminology in the filling model; not itself fresh-air partial pressure |
| Relative filling / load | Model coordinate used by several consumers, distinct from direct mass-flow measurement |
| Cylinder charge | Per-cylinder air-mass coordinate; the reviewed `2ada` conversion has a saturation sentinel |
| `mg/stk` | Milligrams per stroke in the calibration convention; the physical interpretation needs the stated engine/event basis |
| VE | Volumetric efficiency; not automatically the same quantity as every relative-filling coordinate |
| PT1 | First-order lag/filter terminology; stored fractional precision and update cadence affect its behavior |
| EISY | A firmware/calibration namespace used in the reviewed filling/control material; its name alone does not identify a sensor or active branch |

The [load/lambda/VE primer](../msv70-load-lambda-ve-primer.md), [airflow page](airflow-and-load.md), and [pressure page](pressure-model.md) explain the relevant distinctions.

## Fuel, actuators, and protection

| Term | Meaning and limitation |
|---|---|
| Lambda | Air-fuel ratio divided by the fuel’s stoichiometric air-fuel ratio; commanded and measured lambda differ |
| AFR | Air-fuel mass ratio; the stoichiometric value depends on the fuel |
| Injector differential pressure | Rail pressure minus manifold pressure, using compatible pressure references |
| Dead time / latency | Injector/driver delay characterization, not a universal constant independent of voltage and pressure |
| Wall film | Fuel-film storage/transport effects represented by separate correction paths |
| Valvetronic | BMW variable valve-lift system; a software setpoint is not measured lift |
| VANOS | BMW cam-timing adjustment system; eligibility, target, command, and measured position are distinct |
| DISA | Variable intake-manifold geometry terminology used by the model’s selectable configurations |
| EGT | Exhaust-gas temperature; modeled component protection does not substitute for independent measurement |
| DTC | Diagnostic trouble code; a Check Control display request is not a stored DME fault record |
| Fail-safe / fallback | A conditional software response that needs its own trigger and consequence evidence |

See [fuel and injection](fuel-and-injection.md), [torque and controls](torque-and-controls.md), and [ignition, lambda, and protection](ignition-lambda-and-protection.md).

## Arithmetic and logging

| Term | Meaning and limitation |
|---|---|
| `u16` / `s16` | Unsigned/signed 16-bit interpretation; apply it per access, not automatically to all users of an address |
| Q format | Fixed-point scale; Q6 divides raw counts by 64, Q5 by 32, and Q15 by 32768 where that convention is established |
| Saturation | Arithmetic clipping to a representable or declared bound |
| Sentinel | A reserved or saturated result that must not be promoted into a validated usable measurement |
| Endpoint hold | Lookup behavior that keeps the boundary cell/row/column outside an axis instead of extrapolating |
| Retained state | A previous value preserved when its update gate is closed |
| ADC | Analog-to-digital converter; a software channel identity does not by itself identify a physical wire |
| CAN / PT-CAN | Controller Area Network / BMW powertrain CAN network |
| DLC | CAN data-length code used when checking a candidate message layout |
| BLF | Binary Logging Format used by the retained CAN recording |
| DAQ | Data acquisition; the logger’s timing, calibration, and fault handling are part of the evidence |
| Host-bracketed affine alignment | A measured clock mapping using host timing brackets, scale/offset, and retained uncertainty |

## Read the PowerPC excerpts

This glossary explains mnemonics referenced by the archive’s retained listings; the listings and firmware-code excerpts are withheld from this public edition pending rights review. These operations describe software behavior, not the physical meaning of a loaded value:

| Notation | Meaning in an archived listing |
|---|---|
| `li` / `lis` | Load an immediate / load an immediate shifted left 16 bits; later instructions may complete an address |
| `lbz` / `lhz` / `lha` / `lwz` | Load a zero-extended byte / zero-extended halfword / sign-extended halfword / word |
| `stb` / `sth` / `stw` | Store the low 8 / 16 / 32 bits; narrowing a store does not itself saturate |
| `cmpw` / `cmplw` | Signed / unsigned word comparison; `cmpwi` and `cmplwi` use immediate operands |
| `beq` / `bne` / `bge` / `ble` | Conditional branches using condition-register results; read the preceding comparison |
| `bl` / `blr` | Branch with link, normally a call / branch to the link register, normally a return |
| `rlwinm` | Rotate a 32-bit word left, then mask; PowerPC mask-bit numbering starts at the most significant bit |
| `srawi` | Arithmetic right shift; negative-value rounding can differ from division truncated toward zero |
| `ref READ ->` / `ref WRITE ->` | Ghidra’s resolved data-reference annotation, not another instruction |
| `FUN_00474698` | Analysis-generated function name based on its address, not an authenticated BMW source symbol |

Start with the [address-base example](identity-and-memory.md#read-the-address-bases-in-an-instruction) and the [excerpt representation rules](evidence-and-tools.md#read-code-excerpts-without-overstating-them). Follow a called helper before assuming its rounding, overflow, or saturation behavior.

See [calibration and limits](calibration-and-limits.md) and [logging and validation](logging-and-validation.md). Definitions explain the vocabulary; they do not resolve the [open physical and runtime questions](open-questions.md).
