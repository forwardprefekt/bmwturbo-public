# Identify the firmware and its address spaces

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This reference identifies the image behind the wiki and explains how file offsets, runtime addresses, and small-data displacements relate. Apply these mappings to the canonical AC1S image, not to an arbitrary MSV70 dump.

## Canonical project target

The [vehicle identity record](../../TARGET.md) binds the project to this target:

| Field | Recorded identity |
|---|---|
| Vehicle | 2006 BMW E90 330i |
| Engine | N52 family |
| Engine controller | Siemens/VDO MSV70 |
| Transmission | GS6-37BZ six-speed manual |
| Software ID | `0049PP0L921S0AC1S` |
| Calibration definition family | `9PPL921S` |
| Full-read size | 2,621,440 bytes, `0x280000` |
| Native partial XDF | `BMW_Siemens_MSV70_9PPL921S_128K.xdf` |

The canonical full-read SHA-256 is:

```text
757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495
```

The canonical firmware image is not included. Loader provenance (supporting artifact withheld from public edition) binds the archive’s prepared analysis inputs to that source. The image may contain vehicle identity or security state; discussing its software does not authorize redistribution.

## Full-read regions

The [ROM map](../msv70-rom-map.md) identifies these inclusive file ranges. A container offset is not automatically an executable CPU address.

| Full-read file range | Size | Meaning in this image |
|---|---:|---|
| `0x000000..0x01ffff` | 128 KiB | External base, vectors, early code |
| `0x020000..0x03ffff` | 128 KiB | Primary boot/security region |
| `0x040000..0x05ffff` | 128 KiB | Calibration block |
| `0x060000..0x07ffff` | 128 KiB | Erased alternate boot/delivery slot |
| `0x080000..0x17ffff` | 1 MiB | Main external PowerPC program |
| `0x180000..0x1fffff` | 512 KiB | Erased BDM-container gap |
| `0x200000..0x27ffff` | 512 KiB | Internal flash mapped at CPU `0x400000..0x47ffff` |

The region inventory (supporting artifact withheld from public edition) retains hashes and fill statistics. The stock program package comparison matches the canonical external program and internal flash; it does not make every delivery-container byte interchangeable.

## Translate an address without changing its meaning

The Ghidra loader contract (supporting artifact withheld from public edition) uses a big-endian PowerPC analysis layout and sets `r2=0x847ff0` and `r13=0x7f8000` across executable blocks.

| Address form | Canonical conversion |
|---|---|
| XDF offset `x` | Offset within the 128 KiB calibration block |
| Calibration file offset | `0x40000 + x` |
| Runtime calibration address | `0x840000 + x` |
| `r2`-relative calibration access | `0x847ff0 + signed_displacement` |
| `r13`-relative RAM access | `0x7f8000 + displacement` |
| Mapped external CPU address | Same numeric address as its full-read offset |
| Internal-flash CPU address | `0x400000 + (file_offset - 0x200000)` |

For example, `c_conf_maf` has XDF offset `0x04996`, file offset `0x044996`, and runtime address `0x844996`. Code accesses it as `r2 - 0x365a`. Those are four descriptions of one calibration byte, not four independent objects.

A RAM ID such as `ram:r13:6ab6` means absolute address `0x007feab6`. It is not file offset `0x6ab6`, XDF offset `0x6ab6`, or a portable A2L symbol.

### Read the address bases in an instruction

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `FUN_00474698`, `00474698–004746ac`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

`lbz` loads one byte and zero-extends it. The first address is `0x7f8000 + 0x1f62 = 0x7f9f62`; the second is `0x847ff0 - 0x365a = 0x844996`. In the withheld listing, `ref` rows denote Ghidra reference annotations rather than executable instructions. A nonzero RAM gate skips the calibration read; a zero calibration byte takes the same exit. The [airflow page](airflow-and-load.md#eligibility-selection-and-retained-values) explains the remaining availability checks.

## Preserve access width and signedness

The instruction determines how a particular access interprets stored bits. `LHZ` zero-extends a halfword; `LHA` sign-extends it. A shared address can have consumers with different interpretations.

| Raw identity | Bounded role in the reviewed paths | Further reading |
|---|---|---|
| `ram:r13:2b26` | Corrected acquisition or feedback airflow state | [Airflow and load](airflow-and-load.md) |
| `ram:r13:2b30` | Selected or retained airflow state | [Airflow and load](airflow-and-load.md) |
| `ram:r13:6ab6` | Final adjusted-load coordinate with mixed consumer signedness | [Airflow and load](airflow-and-load.md) |
| `ram:r13:2ada` | Unsigned cylinder-charge coordinate | [Fuel and injection](fuel-and-injection.md) |
| `ram:r13:6c48` | Sensor-reconstructed pressure candidate | [Pressure model](pressure-model.md) |
| `ram:r13:6c6a` | Integrated modeled-pressure candidate | [Pressure model](pressure-model.md) |
| `ram:r13:30f6..3100` | Six main-pulse calculation outputs, not physical injector timing | [Fuel and injection](fuel-and-injection.md) |

Use the unit/type registry (supporting artifact withheld from public edition) and direct RAM-access facts (supporting artifact withheld from public edition) before assigning one type to an entire state.

## Keep other software revisions separate

The archived BimmerLabs NoMAF package uses AC3S, not the project-car AC1S base. The structure-730 A2L has a different software identity from the canonical image. Both can guide interpretation; neither supplies drop-in addresses or compatibility proof.

Read the [BimmerLabs comparison](../bimmerlabs-file-analysis.md) and [findings-index address conventions](../msv70-findings-index.md) before transferring a label, equation, or patch between builds.

## Current unknowns

The recorded artifact does not prove that the vehicle currently runs identical bytes. A live identity/readback check must establish that before a capture or programming decision. This page also does not establish write-tool compatibility, recovery readiness, or electrical characteristics.

Related pages: [Hardware and recovery](hardware-and-recovery.md), [calibration and limits](calibration-and-limits.md), and [evidence and tools](evidence-and-tools.md).
