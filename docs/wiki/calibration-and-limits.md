# Calibration and limits

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page is a practical guide to the canonical MSV70 calibration definitions, address spaces, interpolation behavior, and distinct meanings of “limit.” The tables describe the stock 2006 E90 330i N52 AC1S image (`0049PP0L921S0AC1S`), not a safe operating envelope or an authorization to extend calibration ranges. See [interpolated-table layout](../msv70-interpolated-table-layout.md), [boost/load downstream trace](../msv70-boost-load-downstream-trace.md), and [BimmerLabs file analysis](../bimmerlabs-file-analysis.md) for source detail.

## Address and revision identity

The canonical full read is 0x280000 bytes, SHA-256 `757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`. An XDF block offset is not a file or runtime address. For the reviewed image, the calibration block relationship is:

| Namespace | Conversion |
|---|---|
| XDF/calibration offset | Offset shown by the calibration layout. |
| Full-read file offset | XDF offset + `0x40000`. |
| Runtime address | XDF offset + `0x840000`. |
| Example: `KF_ZW_VT98` Z at `0x12528` | Full-read `0x52528`; runtime `0x852528`. |

These conversions are image-specific. A RAM `r13` offset is a separate namespace; do not confuse it with XDF offsets or file offsets. Similar-looking addresses from another software/A2L revision do not identify the same object or prove matching layout. The project car’s AC1S read and BimmerLabs file’s AC3S reference are different revisions.

## Table layout and interpolation

The inspected callsites pass descriptors to signed 8-bit or signed 16-bit value interpolators; the lambda helper receives a Z runtime address plus separate axis pointers. Component aliases may share a descriptor or axis without identifying the same object. Specifically, `ip_lamb_bas_1`, `ip_lamb_bas_2`, `ip_lamb_bas_thr_1`, and `ip_lamb_bas_thr_2` share axes but have distinct Z blocks. Families 3 and 4 have separate axes. Editing a shared axis affects every associated table.

For the maps below, the dimensions are X count × Y count; component offsets and byte sizes are exact for the canonical XDF. Full inventory and lambda/ignition detail: [interpolated-table layout](../msv70-interpolated-table-layout.md).

| Family / map | X and Y convention | Storage and dimensions |
|---|---|---|
| `KF_MDIOP_1/2` | X = RPM; Y = load | 16×16, `uint16` axes, 256 `int16` Z cells. |
| Principal `KF_ZW_*` ignition | X = load/filling; Y = RPM | Usually 16×20 with signed 8-bit Z; exceptions have their own dimensions/storage. |
| `ip_lamb_bas_*` | X = cylinder mass; Y = RPM | 8×8, 64 `uint16` lambda cells. |

The reviewed interpolation helpers hold the boundary row/column outside the calibrated axis; they do not extrapolate. That is a helper-specific behavior established for the inspected consumers, not a universal claim about every MSV70 table. Axis endpoint, helper behavior, active map selection, and physical operating envelope are separate facts.

### Descriptor bytes and signed axis comparisons

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `FUN_00414978`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

The first two descriptor bytes are counts, followed by two contiguous 16-bit axes; Z storage follows both axes. The first-axis state is `0` at/below its first knot, `1` strictly between endpoints, and `2` at/above its last knot. The backward search steps by two bytes, not by a displayed engineering-unit increment. Although the layout inventory describes the MDIOP axis storage as `uint16`, this helper compares the loaded knots as signed `short`; storage width and consumer signedness are separate facts.

For `KF_MDIOP_1`, the resulting **layout reconstruction, not a raw byte dump**, is below. Addresses and component lengths come from the [retained layout inventory](../msv70-interpolated-table-layout.md#optimal-torque-calculation-fun_000bca44); the count/header interpretation is reported from the withheld helper analysis. This is an authored layout diagram, not firmware source.

```text
Runtime address   Role                         Length
0x859814          two axis-count bytes         2 bytes
0x859816          Y/load axis (16 knots)        32 bytes
0x859836          X/RPM axis (16 knots)         32 bytes
0x859856          signed 16-bit Z (256 cells)   512 bytes
```

Here the first stored axis is Y/load, not X/RPM. Descriptor order must therefore be checked against the consumer, not inferred from the X/Y labels in an editor.

### Endpoint hold is a branch, not extrapolation

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `FUN_00414978`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

With the first axis held at its final index, an in-range second coordinate still interpolates along that boundary using a `0x8000` fixed-point fraction. If both coordinates reach or exceed their upper endpoints, the helper returns the final signed 16-bit Z cell directly. It does not extend the first-axis slope. This is the archive’s reported helper endpoint policy, not independently reproducible code proof in this edition and not evidence that changing an axis or operating beyond it is safe.

## Useful stock endpoints

The following values are endpoints in particular stock calibration families or reviewed arithmetic paths, not one global “boost limit.” `mg/stk` means milligrams per stroke; filling percentages and mass are not directly convertible to boost without validated scaling and charge-state assumptions.

| Domain / family | Stock endpoint | Reviewed beyond-range behavior / qualification |
|---|---:|---|
| `ip_maf_kgh_pred_cor_map_tia` pressure | 899.992 hPa | Last pressure row held. |
| `ip_fac_mff_map_wf` pressure | 924.038 hPa | Last pressure column held. |
| Base ignition `KF_ZW_*` load | 108.36% filling | Last load column held. |
| Torque `KF_MDIOP_1/2` load | 115% filling | Last load row held. |
| Base lambda `ip_lamb_bas_1/2`, throttled `ip_lamb_bas_thr_1/2` | 500.011 mg/stk | Family-specific last load column; active family in every region is unknown. |
| Base lambda `ip_lamb_bas_3/4` | 650.008 mg/stk | Family-specific last load column; not a global fuel ceiling. |
| Injector/wall-film operating-point correction | 700.007 mg/stk | Last load column held. |
| Normalized filling intermediate `0x6abe` | signed `0x7fff` | Explicit arithmetic saturation. |
| Generic mass conversion | unsigned `0xffff` | Explicit arithmetic saturation. |

These pressure endpoints are below standard sea-level atmospheric pressure; their relationship to local ambient depends on altitude. Among the named load maps, ignition's 108.36% filling endpoint precedes MDIOP's 115%. Neither arithmetic saturation nor a table endpoint demonstrates engine safety or sensor capability. The [airflow](airflow-and-load.md), [pressure](pressure-model.md), and [fuel](fuel-and-injection.md) pages explain the separate equations.

## Limit classes are not interchangeable

| Class | What it bounds | Example / what not to infer |
|---|---|---|
| Numeric representation | Encodable integer range or stored field width. | `uint16` maximum is not a safe load, pressure, or pulse width. |
| Arithmetic saturation/clamp | One reviewed calculation’s bounded intermediate/result. | Saturation does not prove physical or calibrated limit. |
| Calibration axis endpoint | Defined input range of a map. | Endpoint does not reveal whether that map is active or safe beyond range. |
| Helper endpoint hold | Specific interpolation implementation’s out-of-range choice. | A held cell is not extrapolation and not an ECU-wide policy. |
| Diagnostic/plausibility threshold | A comparison/debounce or diagnostic-policy input. | Threshold alone does not establish complete DTC mapping or fallback consequence. |
| Actuator/software request bound | A bounded request or setpoint in firmware. | Software throttle or Valvetronic value is not measured physical actuation. |
| Sensor transfer/electrical range | Requires sensor characteristic, wiring, and interface facts. | XDF voltage/pressure labels do not prove impedance, protection, or accuracy. |
| Physical/durability/safety limit | Requires validated engine, fuel, sensor, and operating evidence. | Not derivable from ROM/XDF numeric maxima. |

## Current selectors and NoMAF comparison

The [canonical selector evidence](../msv70-maf-to-load-decompilation.md) establishes these stored values. Calibration bytes choose configured branches; accepted runtime captures still need to establish task execution and state history.

| Calibration | Runtime address | Canonical raw value | Bounded significance |
|---|---|---:|---|
| `c_conf_maf` | `0x844996` | 2 | MAF availability/configuration policy |
| `CW_EISY` | `0x85e000` | 0 | Modeled-load topology and separate pressure/control selectors |
| `S_RFST_ON` | `0x85e005` | 0 | No startup-map/transition-filter alternative in the reviewed modeled-load assignment |
| `S_EISY_TURBO` | `0x85e00a` | 0 | Ambient-ratio request/control normalization branch |
| `S_EVNN_AUS` | `0x85e00b` | 0 | Selects modeled filling candidate `6b9a` |

The uploaded BimmerLabs NoMAF package identifies AC3S (`S7623406`), whereas the canonical car read is AC1S. Their direct differences mix BMW revision changes and tune changes and are not valid attribution of “NoMAF-only” edits. A revision-matched comparison of BimmerLabs NoMAF AC3S against stock AC3S covers 125,424 common parameter bytes and reports 2,750 changed bytes, 80 XDF components, and 78 titles. That is the relevant scope for characterizing that package, but it remains a full performance tune, not a pure MAF-delete patch or evidence that its changes are suitable for this car. Identifying BMW’s actual NoMAF strategy instead calls for matched factory NoMAF-minus-MAF comparisons on comparable vehicles and their intersection. See [BimmerLabs analysis](../bimmerlabs-file-analysis.md).

## Extending tables or borrowing revisions

An endpoint extension changes more than a displayed number: confirm exact image identity, component/axis/Z storage, consumers, units/scaling, helper behavior, selector gates, downstream map endpoints, and diagnostic/protection coverage. A shared axis can alter multiple distinct tables. Do not assume spare bytes, neighboring maps, or similarly named tables provide safe extension space. Another software revision can move tables, change descriptors, calibration semantics, logic, and axes; copying addresses or definitions across builds requires independent identity/layout and call-path evidence. The reviewed stock endpoints are constraints in that specific evidence set, not recommended targets for extension.

## Current unknowns

- Runtime execution and state histories under the known canonical selector configuration.
- Active lambda-family selection in every operating region and the physical meaning of out-of-axis operation.
- Runtime diagnostic consolidation, complete fallback consequences, and physical sensor/electrical characteristics.
- A validated operating envelope for any altered sensor, fuel system, engine, or calibration.
- Equivalent instruction and XDF evidence for other software revisions.

## Related pages

- [Torque and controls](torque-and-controls.md)
- [Airflow and load](airflow-and-load.md)
- [Pressure model](pressure-model.md)
- [Fuel and injection](fuel-and-injection.md)
- [Ignition, lambda, and protection](ignition-lambda-and-protection.md)
- [Evidence rules](evidence-and-tools.md)
