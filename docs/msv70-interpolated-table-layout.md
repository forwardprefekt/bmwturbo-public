# MSV70 interpolated-table offsets and sizes

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Date: 2026-08-19

Target: 2006 E90 330i, N52, MSV70 `0049PP0L921S0AC1S`

This reference enumerates the 25 constant-address tables directly passed to
the interpolation helpers by the airflow/load consumers examined in:

- `FUN_0046e3e8`: lambda selection;
- `FUN_000b0424`: ignition calculation;
- `FUN_000bca44`: optimal-torque calculation.

Offsets in the tables below are XDF/calibration-block offsets. Convert them to
the project full read or an executable runtime address as follows:

```text
full-read file offset = XDF offset + 0x40000
runtime address       = XDF offset + 0x840000
```

For example, the `KF_ZW_VT98` Z data at XDF offset `0x12528` is at full-read
offset `0x52528` and runtime address `0x852528`.

The descriptor-runtime column identifies the table/axis descriptor passed to
the signed 8-bit and signed 16-bit interpolation helpers. The lambda helper is
different: its call receives the Z runtime address plus separate X/Y pointers.
For those maps, the direct Z call address is `0x840000 + Z offset`.

## Lambda selection: `FUN_0046e3e8`

| Map | Descriptor runtime | X axis offset + bytes | Y axis offset + bytes | Z offset + bytes | X count x Y count |
|---|---:|---:|---:|---:|---:|
| `ip_lamb_afl_1` | `0x8409f2` | `0x010d0` + 16 B | `0x009f3` + 8 B | `0x0b5cc` + 128 B | 8x8 |
| `ip_lamb_bas_1` | `0x8414d6` | `0x014d8` + 16 B | `0x015d1` + 8 B | `0x0b64c` + 128 B | 8x8 |
| `ip_lamb_bas_2` | `0x8414d6` | `0x014d8` + 16 B | `0x015d1` + 8 B | `0x0b6cc` + 128 B | 8x8 |
| `ip_lamb_bas_3` | `0x8408da` | `0x008dc` + 16 B | `0x009fc` + 8 B | `0x0b74c` + 128 B | 8x8 |
| `ip_lamb_bas_4` | `0x8408ec` | `0x008ee` + 16 B | `0x00a05` + 8 B | `0x0b7cc` + 128 B | 8x8 |
| `ip_lamb_bas_thr_1` | `0x8414d6` | `0x014d8` + 16 B | `0x015d1` + 8 B | `0x0b84c` + 128 B | 8x8 |
| `ip_lamb_bas_thr_2` | `0x8414d6` | `0x014d8` + 16 B | `0x015d1` + 8 B | `0x0b8cc` + 128 B | 8x8 |
| `ip_lamb_hom_afl` | `0x840a0d` | `0x010e2` + 16 B | `0x00a0e` + 8 B | `0x0bb14` + 128 B | 8x8 |

The six `ip_lamb_bas_*` maps are the relevant cylinder-mass/RPM base-lambda
families. `ip_lamb_afl_1` and `ip_lamb_hom_afl` are torque/RPM maps included
because the same function interpolates them.

The base-lambda layout is:

```text
X: 8 x uint16 cylinder mass
Y: 8 x uint8 RPM
Z: 64 x uint16 lambda
```

`ip_lamb_bas_1`, `ip_lamb_bas_2`, `ip_lamb_bas_thr_1`, and
`ip_lamb_bas_thr_2` share the X axis at `0x014d8` and Y axis at `0x015d1`.
Editing either shared axis changes the interpretation of all four Z tables.
They are nevertheless distinct XDF objects (`5221`, `5222`, `5225`, and
`5226`) with separate Z storage. A shared descriptor address is not object
identity.

## Ignition calculation: `FUN_000b0424`

| Map | Descriptor runtime | X axis offset + bytes | Y axis offset + bytes | Z offset + bytes | X count x Y count |
|---|---:|---:|---:|---:|---:|
| `KF_ZW_VT98` | `0x8524de` | `0x12508` + 32 B | `0x124e0` + 40 B | `0x12528` + 320 B | 16x20 |
| `KF_ZW_VT98S` | `0x852668` | `0x12692` + 32 B | `0x1266a` + 40 B | `0x126b2` + 320 B | 16x20 |
| `KF_DZW_TD98` | `0x8527f2` | `0x12804` + 16 B | `0x127f4` + 16 B | `0x12814` + 64 B | 8x8 |
| `KF_ZW_GD` | `0x852854` | `0x1287e` + 32 B | `0x12856` + 40 B | `0x1289e` + 320 B | 16x20 |
| `KF_ZW_NL` | `0x8529de` | `0x12a08` + 32 B | `0x129e0` + 40 B | `0x12a28` + 320 B | 16x20 |
| `KF_ZW_VT_VL98` | `0x852b68` | `0x12b92` + 8 B | `0x12b6a` + 40 B | `0x12b9a` + 80 B | 4x20 |
| `KF_ZW_GD_VL` | `0x852bea` | `0x12c14` + 8 B | `0x12bec` + 40 B | `0x12c1c` + 80 B | 4x20 |
| `KF_FZW_LAMKOR` | `0x852d30` | `0x12d3a` + 8 B | `0x12d32` + 8 B | `0x12d42` + 32 B | 4x4 |
| `KF_DZW_WL_VT` | `0x852d62` | `0x12d74` + 12 B | `0x12d64` + 16 B | `0x12d80` + 48 B | 6x8 |
| `KF_DZW_VNS_VT` | `0x852db0` | `0x12dc2` + 12 B | `0x12db2` + 16 B | `0x12dce` + 48 B | 6x8 |
| `KF_DZW_WL_GD` | `0x852dfe` | `0x12e10` + 12 B | `0x12e00` + 16 B | `0x12e1c` + 48 B | 6x8 |
| `KF_DZW_KR_GD` | `0x852e4c` | `0x12e5e` + 12 B | `0x12e4e` + 16 B | `0x12e6a` + 48 B | 6x8 |
| `KF_DZW_KR_VT` | `0x852e9a` | `0x12eac` + 12 B | `0x12e9c` + 16 B | `0x12eb8` + 48 B | 6x8 |
| `KF_DZW_DYN` | `0x852fc8` | `0x12fd2` + 8 B | `0x12fca` + 8 B | `0x12fda` + 16 B | 4x4 |
| `KF_DZW_DYNN` | `0x85307e` | `0x13088` + 8 B | `0x13080` + 8 B | `0x13090` + 16 B | 4x4 |

For the principal ignition maps, X is load/filling and Y is RPM. The axes are
16-bit and most Z cells are signed 8-bit ignition values. `KF_FZW_LAMKOR` uses
16-bit Z cells, which accounts for its 32-byte 4x4 data block.

## Optimal-torque calculation: `FUN_000bca44`

| Map | Descriptor runtime | X axis offset + bytes | Y axis offset + bytes | Z offset + bytes | X count x Y count |
|---|---:|---:|---:|---:|---:|
| `KF_MDIOP_1` | `0x859814` | `0x19836` + 32 B | `0x19816` + 32 B | `0x19856` + 512 B | 16x16 |
| `KF_MDIOP_2` | `0x859a56` | `0x19a78` + 32 B | `0x19a58` + 32 B | `0x19a98` + 512 B | 16x16 |

The MDIOP layout is:

```text
X: 16 x uint16 RPM
Y: 16 x uint16 load
Z: 256 x int16 torque
```

`KF_MDIOP_1` has five XDF object aliases (`6093..6097`) and `KF_MDIOP_2`
has three (`6098..6100`) at their respective shared descriptors. The table
rows above identify exact storage/components; they do not collapse those
aliases into one semantic object.

## Evidence and scope

The call sites are preserved in:

- `analysis/derived/focused/decompiled/0046e3e8.c` (supporting artifact withheld from public edition);
- `analysis/derived/focused/decompiled/000b0424.c` (supporting artifact withheld from public edition);
- `analysis/derived/focused/decompiled/000bca44.c` (supporting artifact withheld from public edition).

Names, component offsets, element widths, and dimensions were cross-checked
against `analysis/derived/msv70_calibration_objects.csv` (supporting artifact withheld from public edition),
`analysis/ghidra/input/xdf_symbols.csv` (supporting artifact withheld from public edition), and the canonical project XDF.

This list is exhaustive for constant-address interpolation calls made by these
three functions. It is not an exhaustive inventory of every interpolated table
used elsewhere in MSV70 firmware.
