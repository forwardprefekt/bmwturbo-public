# MSV70 ROM and forced-induction calibration map

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Date: 2026-08-17

Target: project-car Siemens MSV70 `0049PP0L921S0AC1S` (vehicle serial withheld)

## Result

The 2.5 MiB project-car read is now split into its executable, calibration,
boot, and container regions. The complete public MS4X 9PPL921S definition has
been converted into searchable object/component catalogs, and a whole-program
Ghidra pass has joined code consumers to calibration addresses.

The editable parameter block is the 128 KiB range at full-read file offsets
`0x040000..0x05ffff`. XDF offsets are relative to that block. Code normally
refers to the same data through the runtime alias `0x840000..0x85ffff`.

For forced induction there is no single boost table. The first-pass worklist
contains 68 calibration objects across airflow, pressure/load modeling,
injector characterization, lambda, ignition, knock, torque, Valvetronic,
throttle/VANOS, monitoring, component protection, and diagnostics. Thirty-nine
are P0 items that must be resolved before boosted operation. The usable list is
`forced_induction_map_plan.csv` (supporting artifact withheld from public edition).

This is a location and dependency map, not a flash-ready tune. It does not
establish safe boosted breakpoints or cell values.

## Evidence and source control

- Canonical full read: 2,621,440 bytes; SHA-256
  `757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`.
- Native MS4X partial XDF: SHA-256
  `61726421333a4dd02438b867dc665f10ba858cf449baa9c87913c05e17aa2ed4`.
- Public MS4X full XDF: SHA-256
  `282e94c3a87435603f9aede8ca1add02d44feaf2bb4047492158dfb838c1ff23`.
- The MS4X partial and full definitions contain the same 468 categories,
  8,269 calibration objects, and 11,063 addressable components. Their material
  difference is the XDF header: a `0x40000` base offset and 2.5 MiB region for
  the full-read form versus offset zero and 128 KiB for the partial form.
- The XDF itself says it was converted from an A2L. MS4X publishes both files
  on its [Definition Files](https://www.ms4x.net/index.php?title=Definition_Files)
  page and identifies the applicable 9PPL921S software family.
- BMW stock program package `7577968A.0pa` matches the project read byte for
  byte in the external program (`0x080000..0x17ffff`) and MPC internal flash
  (`0x400000..0x47ffff` CPU addresses). Its only differences are in the
  alternate boot/delivery payload at `0x060000..0x07ffff`, which is erased in
  the project-car BDM read.
- The alternate-slot and checksum interpretation agrees with the public
  [MSx70-Flasher](https://github.com/terraphantm/MSx70-Flasher) implementation,
  especially
  [`Checksums_Signatures.cs`](https://github.com/terraphantm/MSx70-Flasher/blob/master/MSx70%20Flasher/Checksums_Signatures.cs).
- The MS4X [Firmware Finder](https://www.ms4x.net/index.php?title=Firmware_Finder)
  independently lists the relevant 7577968 hardware and MAF/NoMAF ZUSB
  variants. Its current public table exposes the metadata but not download
  links for those rows, so those factory MAF/NoMAF pairs are targets for a
  later controlled comparison rather than inputs silently sourced elsewhere.

The exact input hashes, parsed checksum ranges, revision references, and
public-program comparison are machine-readable in
`msv70_rom_map_summary.json` (supporting artifact withheld from public edition).

## Address conventions

| Meaning | Conversion |
|---|---|
| XDF / 128 KiB parameter offset | `x` |
| Offset in the 2.5 MiB BDM read | `0x40000 + x` |
| Runtime calibration alias used by code | `0x840000 + x` |
| External program CPU address | same as full-read offset through `0x17ffff` |
| MPC internal-flash CPU address | `0x400000 + (file_offset - 0x200000)` |

Example: `c_conf_maf` is at XDF offset `0x04996`, full-read offset
`0x044996`, and runtime address `0x844996`. Ghidra finds the latter as
`r2 - 0x365a` in `FUN_00474698`, anchoring the inferred r2 base at `0x847ff0`.

## Full-read layout

| Full-read range | Size | CPU/runtime meaning | Status in this read |
|---|---:|---|---|
| `0x000000..0x01ffff` | 128 KiB | external base, vectors, early code | populated |
| `0x020000..0x03ffff` | 128 KiB | primary boot/security | populated |
| `0x040000..0x05ffff` | 128 KiB | parameter/calibration; alias `0x840000..0x85ffff` | populated; edit domain |
| `0x060000..0x07ffff` | 128 KiB | alternate boot/delivery slot | 100% `0xff` |
| `0x080000..0x17ffff` | 1 MiB | main external PowerPC program | populated; exact public-program match |
| `0x180000..0x1fffff` | 512 KiB | BDM-container gap | 100% `0xff` |
| `0x200000..0x27ffff` | 512 KiB | MPC internal flash at CPU `0x400000..0x47ffff` | populated; exact public-program match |

Region hashes and fill percentages are in
`msv70_rom_regions.csv` (supporting artifact withheld from public edition).

### Checksum coverage

The parameter checksum metadata begins at full-read offset `0x0400a0`; the
range count is stored at `0x0400a8`. This image has two parameter-relative
ranges: `0x00240..0x0ffff` and `0x10000..0x1eaff`.

The program checksum metadata begins at `0x080100`, with 24 covered ranges:
external program `0x080500..0x17ffff` and all eight 64 KiB internal-flash
segments at CPU `0x400000..0x47ffff`. Any edit workflow must regenerate the
appropriate checksum and must not treat the alternate boot payload as part of
the normal installed program merely because it appears in a WinKFP delivery
file.

## Calibration atlas and Ghidra coverage

The generated atlas has two levels:

- `msv70_calibration_objects.csv` (supporting artifact withheld from public edition)
  has one row per XDF object: name, categories, descriptor hint, axes, value
  storage, dimensions, and units.
- `msv70_calibration_components.csv` (supporting artifact withheld from public edition)
  has one row per x/y/z/value component: all three address spaces, byte range,
  equation, and decoded stock physical minimum/maximum.

The historical regex/decompiler-text discovery export found 6,769 literal or
r2-relative references into `0x840000..0x85ffff`, spanning 5,716 unique
runtime addresses and 1,005 functions. Of those reference occurrences, 6,718
have at least one XDF descriptor/component candidate and 51 remain unmatched;
the matches cover 4,925 unique XDF titles. The joined historical result is
`ghidra_calibration_xrefs.csv` (supporting artifact withheld from public edition),
with counts in
`ghidra_calibration_xrefs_summary.json` (supporting artifact withheld from public edition).

The current Ghidra 12.0.4 whole-program authority is
`analysis/facts/msv70/calibration-references.csv` (supporting artifact withheld from public edition):
17,352 observations, comprising 8,428 primary listing/reference/p-code rows
and 8,924 explicitly supplemental decompiled-text rows. The historical file
remains useful for candidate-to-XDF joins but must not be presented as the
current coverage contract.

Exact descriptor and component-start matches are the strongest evidence.
Inside-component and +/- one/two-byte matches are intentionally labeled as
discovery aids, not definitive symbol assignments. Shared axes and packed
descriptors can also produce more than one candidate for a code address.

## First tables to work through

Offsets below are XDF/parameter offsets; add `0x40000` for a full-read file
offset or `0x840000` for a runtime data address. The descriptor column is the
runtime address passed by code to the interpolation helpers.

| Domain / calibration | Descriptor | x | y | z/value | Confirmed Ghidra consumer(s) |
|---|---:|---:|---:|---:|---|
| MAF conversion `id_maf_tab` | `0x8407b3` | `0x007c5` | `0x007b4` | `0x09ca4` | `FUN_00432fc8` |
| Manifold differential sensor `ip_map_dip_mes_bas__v_map` | `0x84110a` | `0x0110c` | — | `0x037d8` | `FUN_0013a260`, `FUN_004576f0` |
| Predicted-airflow pressure correction `ip_maf_kgh_pred_cor_map_tia` | `0x848a46` | `0x08aa1` | `0x08a48` | `0x08c20` | `FUN_000d8668` |
| MAP/wall-film factor `ip_fac_mff_map_wf` | `0x84092c` | `0x0092e` | — | `0x027c2` | `FUN_0043b118` |
| Filling clamp `KL_RFVIST_MX` | `0x8544be` | `0x144c0` | — | `0x144cc` | `FUN_004439b4` |
| Pressure/load clamp `KL_PIRG_MX` | `0x8544a4` | `0x144a6` | — | `0x144b2` | `FUN_004439b4` |
| Injector pressure correction `ip_fac_ti_prs` | `0x8457a0` | `0x057a2` | — | `0x059d6` | `FUN_0043cc84` |
| Injector dead time `ip_ti_add_dly` | `0x845834` | `0x05835` | — | `0x05a2e` | `FUN_0043cce0`, `FUN_0043d02c`, `FUN_0043db3c` |
| Base lambda 1 `ip_lamb_bas_1` | `0x8414d6` | `0x014d8` | `0x015d1` | `0x0b64c` | `FUN_0046e3e8` |
| Base lambda 3 `ip_lamb_bas_3` | `0x8408da` | `0x008dc` | `0x009fc` | `0x0b74c` | `FUN_0046e3e8` |
| Full-load lambda `ip_lamb_fl__n` | `0x840a7c` | `0x00a7d` | — | `0x036a2` | `FUN_0046e7b0` |
| Main VVT spark `KF_ZW_VT98` | `0x8524de` | `0x12508` | `0x124e0` | `0x12528` | `FUN_000b0424` |
| Full-load VVT spark `KF_ZW_VT_VL98` | `0x852b68` | `0x12b92` | `0x12b6a` | `0x12b9a` | `FUN_000b0424` |
| Torque ceiling `KL_MDMAX` | `0x857ee0` | `0x17ee2` | — | `0x17ef8` | `FUN_000b55a4` |
| Optimal-torque model `KF_MDIOP_1` | `0x859814` | `0x19836` | `0x19816` | `0x19856` | `FUN_000bca44` plus four related functions |
| Driver demand `ip_fac_tq_req_driv` | `0x84182c` | `0x018fe` | `0x0182e` | `0x0ad04` | `FUN_000a4a00`, `FUN_000a8248` |
| Throttle target `KF_WDKDPS` | `0x85c34e` | `0x1c37a` | `0x1c350` | `0x1c38a` | `FUN_000c5358` |
| Valvetronic target `KF_HUBE` | `0x85c6b2` | `0x1c6dc` | `0x1c6b4` | `0x1c704` | `FUN_000c5e08`, `FUN_004439b4` |
| Exhaust-temperature protection `KF_TA_VKSOLL` | `0x851134` | `0x1113e` | `0x11136` | `0x11146` | `FUN_0010fcd4` |
| Component-protection lambda `KF_BS_MX` | `0x851166` | `0x11170` | `0x11168` | `0x11178` | `FUN_0010fcd4` |

The full 68-row worklist also contains all six airflow-indexed knock-threshold
maps, injector standard-flow constants, all six base-lambda families, the safe
and throttled spark fallbacks, MDIOP companions, VANOS targets, torque-monitor
copies, and the relevant diagnostic declarations.

## Why these must be changed as a system

The executable trace establishes this path:

```text
pressure-corrected filling (r13+0x6b6a)
  -> normalized/adjusted load (r13+0x6ab6)
     -> ignition KF_ZW_* (FUN_000b0424)
     -> torque KF_MDIOP_* (FUN_000bca44)
     -> converted cylinder mass (r13+0x2ada)
        -> base lambda families (FUN_0046e3e8)
```

The inspected interpolation helpers hold their endpoint cells; they do not
extrapolate. Stock pressure corrections already stop near 900--924 hPa,
primary spark at 108.36% filling, MDIOP at 115%, and base-lambda families at
500 or 650 mg/stk. Raising `KL_RFVIST_MX` alone would therefore leave fuel,
spark, torque, and pressure corrections pinned to naturally aspirated endpoint
values. The detailed proof is in
[`msv70-boost-load-downstream-trace.md`](msv70-boost-load-downstream-trace.md).

## What the revision-matched NoMAF comparison proves

The stock `S7623406.0da` and owner-archived BimmerLabs NoMAF file both declare
`$REFERENZ 0049PP0L921S0AC3S 4`. Comparing those exact revisions removes the
AC1S-to-AC3S update from the result. Across the 125,424 parameter bytes
explicitly present in both Intel-HEX files, the tune changes 2,750 bytes,
80 XDF components, and 78 unique XDF titles. The detailed rows are in
`stock_ac3s_vs_bimmerlabs_nomaf_ac3s_xdf_changes.csv` (supporting artifact withheld from public edition).

The files omit different exchange/control and erased-flash spans. Those bytes
are excluded from the change count instead of being interpreted as changes;
the explicit and omitted coverage is recorded in the JSON summary.

High-confidence mode changes include:

- `c_conf_maf`: 2 to 1;
- `c_abc_inc_maf`: 2 to 0;
- `c_abc_inc_load_tps_plaus`: 2 to 0;
- MAF and load/throttle-plausibility DTC declarations and failure classes:
  nonzero to zero;
- substitute-airflow table `ip_maf_1_diag__n__tps_av`: 36 cells changed.

The file also changes full-load lambda, torque ceilings/models, driver demand,
throttle, Valvetronic, and component-protection calibrations. It does **not**
change the `KF_ZW_*` spark maps. This is useful evidence about the factory
fallback/Alpha-N path, but it is not a turbo speed-density recipe and should
not be copied: it deliberately defeats diagnostics and leaves the stock spark
axis limitation intact.

## Recommended implementation order

1. Freeze identity and provenance: retain the canonical read, software family,
   source hashes, and checksum ranges.
2. Decide the supported airflow architecture and exact hardware. For the
   selected OEM-MAF route, characterize `id_maf_tab`, housing, reference
   airflow constants, injectors, fuel pressure, and dead time together.
3. Validate the conditioned raw-state route
   `r13+0x2b30 -> 0x2b26 -> 0x6980 -> 0x6ac0 -> 0x6ab8 -> 0x6ab6`.
   The first edge requires `r13+0x1f62 == 0`; the final edge requires
   `CW_EISY` bit 0 set. Bind `2b26` to physical measured MAF and prove the
   active runtime branch before treating this static route as the load source.
   The canonical image instead selects `6b9a -> 6b6a -> 6abe -> 6ade`
   with `S_RFST_ON=0`; startup-map/filter alternatives are disabled.
   The conditional airflow/integrator bridge is recovered but inactive with
   canonical `CW_EISY=0`; its modeled-pressure ceiling remains an operating-domain gate.
   Static dataflow is not tuning authorization.
4. Extend each required axis and its data together: pressure corrections,
   lambda, every normal/safe spark path, knock thresholds, and torque model.
5. Align driver demand, torque monitors, throttle, Valvetronic, and VANOS with
   the new load domain.
6. Calibrate component protection and plausibility for the new domain; retain
   DTCs rather than zeroing diagnostic declarations.
7. Regenerate checksums and validate first on a bench, then at wastegate-spring
   pressure with independent wideband lambda, absolute MAP, charge temperature,
   and fuel-pressure logging.

Until those steps are proven, the safe use of this map is inspection and
instrumentation—not flashing guessed values.

## Reproduction

The following scripts generate the map without modifying source firmware:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/analyze_msv70_rom.py \
  --full-read "$CANONICAL_FULL_READ" \
  --xdf references/xdf/BMW_Siemens_MSV70_9PPL921S_128K.xdf \
  --stock-ac3s references/bimmerlabs/raw/S7623406.0da \
  --nomaf-zip references/bimmerlabs/raw/MSV70_S7623406_330i_NoMAF.zip \
  --nomaf-member MSV70_S7623406_330i_NoMAF.0da \
  --stock-program-zip references/bimmerlabs/raw/7577968A.0pa.zip \
  --stock-program-member 7577968A.0pa

python3 scripts/build_forced_induction_map_plan.py \
  --objects analysis/derived/msv70_calibration_objects.csv \
  --xrefs analysis/derived/ghidra_calibration_xrefs.csv \
  --nomaf-diff analysis/outputs/stock_ac3s_vs_bimmerlabs_nomaf_ac3s_xdf_changes.csv \
  --output analysis/derived/forced_induction_map_plan.csv
```

Run Ghidra only through the pinned disposable workflow in the
Ghidra decompilation playbook (supporting artifact withheld from public edition); its current
arguments verify the release archive, loader inputs, baseline, JVM, and
protected projects before publishing evidence. The scripts above only read
canonical inputs and write derived CSV/JSON evidence.
