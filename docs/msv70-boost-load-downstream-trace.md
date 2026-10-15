# MSV70 pressure-corrected load: downstream trace and boost limits

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Date: 2026-08-17

Target: project-car MSV70 `0049PP0L921S0AC1S` (vehicle serial withheld)

## Result

The final pressure-corrected cylinder-filling value does reach the principal
fuel, ignition, and torque paths. The stock executable has explicit 16-bit
saturation, but those arithmetic guards are not the first practical boosted
limits. Stock calibration axes end first, and the inspected interpolation
helpers hold the last row or column rather than extrapolate.

The important consequence is that increasing the RFVIST neural-network clamp
alone is insufficient. Above the stock axes, spark, torque, lambda, and several
pressure corrections silently reuse their endpoint values.

## Provenance and address conventions

- Canonical full read: 2,621,440 bytes, SHA-256
  `757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`.
- Native XDF: SHA-256
  `61726421333a4dd02438b867dc665f10ba858cf449baa9c87913c05e17aa2ed4`.
- External/LAR code is loaded at CPU address `0x000000`.
- MPC5xx internal flash is extracted from full-read file offset `0x200000`
  and loaded at CPU address `0x400000`.
- XDF storage offset `x` corresponds to CPU address `0x40000 + x`.
- Runtime descriptor constants correspond to XDF descriptors with a
  `0x840000` bias. The inferred r2 base is `0x847ff0`; this is independently
  anchored by `c_conf_maf` at XDF offset `0x4996` and code use at
  `r2 - 0x365a` in `FUN_00474698`.

Raw ECU and WinKFP source files are retained only in the separate private archive. Their
hashes and metadata are recorded in `references/bimmerlabs/manifest.json` (supporting artifact withheld from public edition) and the generated
`analysis/ghidra/input/provenance.json` (supporting artifact withheld from public edition).

## Executable dataflow

```text
FUN_004439b4
  RFVIST neural-network result
    -> KL_RFVIST_MX clamp (NN core only)
    -> pressure/model corrections
    -> r13+0x6b6a final selected filling
    -> r13+0x6b6e converted filling, uint16-saturating

FUN_00442f7c
  r13+0x6b6a / r13+0x6b68
    -> r13+0x6abe normalized final filling
       (explicit signed positive ceiling 0x7fff)
    -> r13+0x6ab6 principal adjusted load

  ignition: FUN_000b0424
    r13+0x6ab6 -> delayed/weighted r13+0x634e -> KF_ZW_* axes
    r13+0x65ac -> separate r13+0x6352/0x6354 histories
                 -> selector-contingent r13+0x6350 map index

  torque: FUN_000bca44
    r13+0x6ab6 + rpm -> KF_MDIOP_1 / KF_MDIOP_2

  fuel: FUN_00434c98 -> FUN_0046e3e8
    r13+0x6ab6 -> saturating conversion -> r13+0x2ada (mg/stk)
    -> ip_lamb_bas_* / ip_lamb_bas_thr_* axes
```

### Filling selection and conversion

At the end of `FUN_004439b4`, canonical XDF calibration `S_EVNN_AUS` at
`0x85e00b` selects either r13 `0x6b9a` or `0x6b5a` into r13 `0x6b6a`. The
function then computes `(0x6b6a << 10) / 0x0ccd` into `0x6b6e`, storing
`0xffff` if the result no longer fits in 16 bits. This is downstream of the
`KL_RFVIST_MX` use and is why that table is not a final 112% filling ceiling.
The mismatched structure-730 A2L projects 38 `EISY_EV` anchors into this
function and supplies strong candidate names: `Rfv_ist_nn` for `6b9a`,
`Rfv_ist_hub` for `6b5a`, and selected `Rfv_ist` for `6b6a`.

`FUN_00442f7c` normalizes the final filling against r13 `0x6b68`, scales by
`0x1400`, and explicitly caps the r13 `0x6abe` intermediate at signed `0x7fff`.
Its subsequently adjusted result at r13 `0x6ab6` is the common load input
proven below. The 18-anchor A2L `EISY` projection makes `Rf_ubeg` and `Rf`
strong lexicon candidates for `6abe` and `6ab6`. Because A2L EPK `9PP73000`
does not match canonical `9PP92L00`, these are semantic candidates rather than
exact address-bound symbols.

### Ignition

`FUN_000b0424` reads r13 `0x6ab6` into the `0x63b0` history and derives
delayed/weighted `0x634e`. Separate histories rooted at `0x65ac` produce
`0x6352` and `0x6354`; these are not filtered copies of `0x6ab6`.
`0x6350` normally selects `0x634e`, but can select those separate histories
under the recovered selectors. The function calls the signed 2-D interpolator
`FUN_00415070` with:

- `0x8524de`: `KF_ZW_VT98`
- `0x852668`: `KF_ZW_VT98S`
- `0x852b68`: `KF_ZW_VT_VL98`

The [typed consumer matrix](msv70-maf-to-load-decompilation.md#consumer-matrix)
and retained spark instructions (supporting artifact withheld from public edition)
preserve the distinct histories and selector boundaries.

The stock normal and safe base-spark load axes end at **108.36%**. The
full-load variants contain only 70, 80, 90, and 108.36% columns. Inspection of
`FUN_00415070` shows first/last-axis endpoint selection outside the breakpoint
range: no extrapolation and no wrap.

### Torque

`FUN_000bca44` passes r13 `0x6ab6` and rpm directly to `FUN_00414978` for:

- `0x859814`: `KF_MDIOP_1`
- `0x859a56`: `KF_MDIOP_2`

Both load axes end at **115%**. `FUN_00414978` also selects endpoint cells when
an input falls outside its axes. `KL_MDMAX` (`0x857ee0`, used by
`FUN_000b55a4`) is a separate torque ceiling; it is not a replacement for
extending and calibrating the MDIOP load axis.

### Fuel / lambda

`FUN_00434c98` passes r13 `0x6ab6` through saturating multiply/divide helper
`FUN_0040b46c` and stores the result at r13 `0x2ada`. `FUN_0046e3e8` uses
`0x2ada` as the mg/stk coordinate for the base-lambda families at runtime data
addresses `0x84b64c`, `0x84b6cc`, `0x84b74c`, `0x84b7cc`, `0x84b84c`, and
`0x84b8cc`.

The load ceilings are:

| Calibration family | Stock load-axis maximum | Stock rpm-axis maximum |
|---|---:|---:|
| `ip_lamb_bas_1/2` | 500.011 mg/stk | 5024 rpm |
| `ip_lamb_bas_3/4` | 650.008 mg/stk | 7008 rpm |
| `ip_lamb_bas_thr_1/2` | 500.011 mg/stk | 5024 rpm |
| `ip_ti_cor_maf_sp_wf_opm_1/2` | 700.007 mg/stk | 5600 rpm |

The fuel interpolator `FUN_00408ab4` has the same endpoint behavior. Which base
lambda family is active in every operating region is not yet proven, so the
500 and 650 mg/stk values must be treated as family-specific limits rather
than one global fuel ceiling.

## Earliest stock boosted limits

| Domain | Observed stock endpoint | What happens beyond it |
|---|---:|---|
| Pressure correction `ip_maf_kgh_pred_cor_map_tia` | 899.992 hPa | Last pressure row held |
| Pressure correction `ip_fac_mff_map_wf` | 924.038 hPa | Last pressure column held |
| Base ignition `KF_ZW_*` | 108.36% filling | Last load column held |
| Torque `KF_MDIOP_1/2` | 115% filling | Last load row held |
| Base lambda, family 1/2 and throttled | 500.011 mg/stk | Last load column held |
| Base lambda, family 3/4 | 650.008 mg/stk | Last load column held |
| Injector/wall-film operating-point correction | 700.007 mg/stk | Last load column held |
| Normalized filling intermediate (`0x6abe`) | signed `0x7fff` | Explicit saturation |
| Generic mass conversion `FUN_0040b46c` | unsigned `0xffff` | Explicit saturation |

The pressure tables end below atmospheric pressure and therefore clamp before
positive manifold pressure. Among the named load tables, ignition is the first
percentage-based endpoint at 108.36%, followed by MDIOP at 115%. The mg/stk
axes should not be converted to boost without proving the ECU scaling and
charge-state assumptions.

## Reproduction

Prepare the generated Ghidra binary inputs and XDF labels:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/prepare_ghidra_inputs.py \
  --full-read "$CANONICAL_FULL_READ" \
  --xdf references/xdf/BMW_Siemens_MSV70_9PPL921S_128K.xdf
```

Import and analyze only through the exact disposable workflow in the
Ghidra decompilation playbook (supporting artifact withheld from public edition). The playbook
pins the Ghidra archive and JVM, verifies inputs and memory blocks, refuses
tracked project names, and compares the fresh result before recording evidence
in the separate private archive.

The current structured exporters retain listing/reference/p-code observations
and label decompiled-text constant search as supplemental evidence. Direct
Ghidra references can be absent when MSV70 passes descriptor constants and
r2-relative pointers, so both evidence kinds remain visible instead of being
collapsed into one confidence level.

Regenerate axis limits:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/extract_xdf_axis_limits.py \
  --full-read "$CANONICAL_FULL_READ" \
  --xdf references/xdf/BMW_Siemens_MSV70_9PPL921S_128K.xdf \
  --output analysis/derived/boost_axis_limits.csv \
  ip_fac_mff_map_wf ip_maf_kgh_pred_cor_map_tia \
  ip_lamb_bas_1 ip_lamb_bas_2 ip_lamb_bas_3 ip_lamb_bas_4 \
  ip_lamb_bas_thr_1 ip_lamb_bas_thr_2 \
  ip_ti_cor_maf_sp_wf_opm_1 ip_ti_cor_maf_sp_wf_opm_2 \
  KF_ZW_VT98 KF_ZW_VT98S KF_ZW_GD KF_ZW_VT_VL98 KF_ZW_GD_VL \
  KF_MDIOP_1 KF_MDIOP_2
```

## Confidence and remaining risk

High confidence: function addresses, r13 offsets, descriptor literals, axis
values, endpoint behavior, and explicit saturation are directly reproducible
from the canonical image and XDF.

Medium confidence: semantic names for r13 `0x6b5a`, `0x6b9a`, `0x6b6a`,
`0x6abe`, `0x6ab6`, and `0x2ada` combine canonical transformations/consumers
with a strong but revision-mismatched A2L function-group lexicon. Runtime
capture is still required for exact address-bound naming and physical scaling.

Not established: safe boosted axis values, correct boosted cell contents,
complete diagnostic/torque-monitor coverage, or on-engine behavior. This is a
reverse-engineering result, not a flash-ready tune. RAM logging and independent
wideband/MAP/fuel-pressure instrumentation remain required before boost.
