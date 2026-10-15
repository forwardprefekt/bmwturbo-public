# Airflow and load

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page traces the canonical MSV70 airflow acquisition, conditional selection, and charge/load representations. Its central distinction is provenance: acquired MAF-table samples, selected airflow, normalized relative load, modeled filling, and derived cylinder charge are separate states, not interchangeable measurements. Static software paths are known; live branch activation and physical accuracy are not.

## Canonical scope

The reference target is the 2006 E90 330i N52, MSV70 `0049PP0L921S0AC1S`, XDF `9PPL921S`, canonical full-image SHA-256 `757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`. RAM displacements below are relative to `r13=0x007f8000`; they are not file offsets. The calculations and limits here refer to that image and its matching XDF, not another MSV70 software version or an A2L address projection. See the [identity and memory reference](identity-and-memory.md).

Canonical selectors are `CW_EISY=0` (runtime `0x85e000`), `S_RFST_ON=0` (`0x85e005`), `S_EVNN_AUS=0` (`0x85e00b`), and `c_conf_maf=2` (runtime `0x844996`, XDF offset `0x04996`). Selector bytes establish configured branches, not proof that a particular task ran. The [acquisition closure](../../analysis/derived/review-2026-09-04-offline-maf-closure/findings.json) and [load continuation](../../analysis/derived/review-2026-09-04-load-continuation/findings.json) retain the source bindings.

## Acquisition: sensor conversion to averaged airflow

`FUN_00432fc8` reads logical ADC channel `0x1b`, shifts the result right two bits, and truncates to an 8-bit index (`r13+0x1f40`). It splits that byte into row and column indices for a direct 16×16 `u16` lookup at runtime `0x849ca4` (file offset `0x49ca4`), the `id_maf_tab` transfer. There is no interpolation or ADC-domain clamp in that operation. XDF scale is Q6, `0.015625 kg/h/count`; `65535` therefore means `1023.984375 kg/h`, and stock table cells from index 233 onward are saturated. A different sensor cannot make clipped table values informative above that ceiling.

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `FUN_00432fc8`, `00432fd8–00433008`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

`STB` discards all but the low byte before `LBZ` reloads the index. The high nibble supplies a 32-byte row stride; the low nibble supplies a two-byte cell offset. `LHZx` loads one unsigned halfword from `0x849ca4`, with no interpolation call.

When acquisition is enabled (`1f62 != 0`), alternating banks accumulate Q6 samples in `u32` sums `4008`/`400c` and increment `u16` counters `2c80`/`2c82`. `FUN_00435048` consumes the opposite bank and computes `floor(sum / (2 × count))` into `2b2c`, converting Q6 sum to Q5 airflow (`1/32 kg/h` per count). The bank sums and counters are ordinary-width arithmetic; safety from overflow depends on sample scheduling/count bounds, not a saturating add. The averaging expression has no local zero-count guard. Runtime cadence, physical ADC path, and sample delivery are not established by code alone.

In the acquisition-enabled branch, `2b2c` feeds `2b28`, then an IAT/flow Q7 correction gain produces `2b26`. When `1f62 == 0`, the feedback branch instead copies selected `2b30` into `2b28` and `2b26`. Thus `2b26` is not unconditionally a fresh MAF measurement.

**Derived bound:** a coherent nonempty bank of `u16` samples, without accumulator or count wrap, averages to at most 32767. Even byte gain 255 then produces only `floor(32767*255/128)=65278`, below the helper's unsigned saturation boundary. The canonical correction map tightens the acquired maximum to 33790, attained at base 32767 and gain 132. This bound does not apply to every possible feedback value. The [acquisition closure](../../analysis/derived/review-2026-09-04-offline-maf-closure/findings.json) retains the averaging code and correction-map bytes.

## Eligibility, selection, and retained values

The selected-airflow publisher is `FUN_00435180`. Its outer byte gate is `1fea`: zero permits an update; nonzero leaves `2b30` unchanged, so an observed value may be retained rather than newly selected. With the gate open, `1e8c != 0` selects `2b26` into `2b30`; `1e8c == 0` selects the substitute/model candidate `2ac2`. `2ac2` is copied from `6ab4` in the recovered path, but its physical scale and possible overrides must not be guessed from its selector role.

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `FUN_00435180`, `00435190–004351b4`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

The first branch skips this store; the second chooses its input. Neither branch clears a stale `2b30` value when updates are inhibited. The native listing and its reference annotations are withheld.

Availability and eligibility have multiple producers. The acquisition closure shows `0046dec4` clears `1e8c` when `1f62==0`, `2266!=0`, `1e88!=0`, or when `2314!=0` with a status nibble of 1. Canonical `c_conf_maf=2` permits the `00474698` availability latch when `1fe1!=0`; configuration 1 has additional index/count checks, not universal electrical fault limits. `00438d94` writes `1fea=0` unless `1fe2!=0` and `21d8>150`; otherwise it writes `21da & 1`. Opaque status inputs are not physical aliases. The [acquisition closure](../../analysis/derived/review-2026-09-04-offline-maf-closure/findings.json) and [selector/filling boundary review](../msv70-airflow-filling-boundary.md) preserve these conditions.

The canonical `CW_EISY=0` means modeled load, not direct measured-airflow load, and disables the optional bit-1 measured-flow input to the pressure integrator. `S_RFST_ON=0` disables the modeled load startup-map and transition-filter alternatives. `S_EVNN_AUS=0` selects the neural-network filling candidate `6b9a` rather than lift-based candidate `6b5a`; those semantic labels are A2L-informed candidates, not exact canonical symbols. The A2L software revision differs, so it supplies vocabulary/group corroboration only.

## Two routes to adjusted load

### Conditioned measured-airflow route

`00434c48` copies `2b26` into `6980`. `00442f7c` filters that input into `6ac0` through the `u32` PT1 accumulator at absolute `0x007fe0f8` (`r13+0x60f8`), retaining 16 fractional bits. The stock gain `K_ZK_MSHFM` is raw 64887. The normalization instructions (supporting artifact withheld from public edition) and reference producer (supporting artifact withheld from public edition) are the archive’s basis for this **authored explanatory pseudocode, not firmware source**:

```text
RPM_effective = s16(6984) if 6eda.bit3 else max(s16(6984), 400)
N = trunc0(RPM_effective * 31219 / 16384)
6b68 = low16(min(N, 65535))
6ab8 = min(32767, floor(u16(6ac0) * 5120 / u16(6b68)))
        or 32767 when 6b68 == 0
```

The `6e9b.bit1` gate controls the normalizer update. Negative RPM is not asserted a valid running state. When `CW_EISY` bit 0 is set, `6ab8` less signed `6b62` is stored as low 16 bits at `6ab6`. That final subtraction has neither an upper saturation nor the modeled branch's floor of 600. The sole direct `6b62` producer writes zero even on its cached-update path; a negative-correction wrap is hypothetical, not demonstrated stock behavior.

The feedback edge `2b30 -> 2b26` requires `1f62==0`; fresh corrected acquisition uses `1f62!=0`. Both can feed `2b26 -> 6980`, but they have different provenance. The direct bit-0 load path reads `2b26`, not selected `2b30`, so rejecting measured selected airflow through `1e8c` does not itself reject that direct-load input.

### Canonical modeled-filling route

`004439b4` selects `6b9a`/`6b5a` using `S_EVNN_AUS`, then computes `6b6e=sat_u16((u16(6b6a)<<10)/0x0ccd)`. `00442f7c` normalizes selected filling into signed-saturated `6abe`, copies it to `6ade` with canonical `S_RFST_ON=0`, then stores `max(s16(6ade)-s16(6b62),600)` to `6ab6` as a halfword. The final modeled assignment has a lower floor but no upper saturation at that final subtraction. `6b6e` is a conversion/publication, not a branch selector.

**Derived bound:** because the input to the `6b6e` conversion is loaded unsigned-16, its maximum output is `floor(65535*1024/3277)=20478`. The subsequent `0xffff` guard exists but cannot activate for that loaded width. This does not remove the separate load-to-charge saturation.

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `FUN_004439b4`, `00444964–0044498c`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

`LHZ` zero-extends the input; the rotate/mask shifts it left ten bits, and `DIVWU` performs unsigned division. The resulting width bound of 20478 follows before the saturation branch. The reported conversion publishes `6b6e`, not adjusted load `6ab6`; the native listing and its reference annotations are withheld.

The two producers meet at `6ab6`, the shared adjusted-load coordinate (nominally 0.01 percentage point/count). It has mixed signed and unsigned consumers, so its bits must not be described as universally signed or unsigned. It feeds spark, torque, control, monitors, and cylinder-charge conversion. Actual calibration coverage is narrower than the storage range; for example, stock base-spark axes end at 108.36% and MDIOP torque axes at 115%, with endpoint hold rather than extrapolation. See the [downstream pressure/load trace](../msv70-boost-load-downstream-trace.md).

## Load-to-charge conversion and independent bounds

`00434c98` calls rounded saturating multiply/divide helper `0040b46c`: `2ada = sat_round(u16(6ab6) * c_maf_ref / 10000)`. The helper returns `0xffff` for a zero divisor or a saturated result; this call uses divisor 10000. Stock `c_maf_ref=30503` corresponds to 646.511085 mg/stk under the reviewed engineering convention. The documented charge scale is `0.021195 mg/stk/count`; `0xffff` is the saturation sentinel, not validated usable charge. Load raw 21484 produces 65533; raw 21485 produces sentinel 65535. See the [load-to-charge derivation](../msv70-maf-to-load-decompilation.md#4-the-load-to-cylinder-mass-conversion).

Keep these separate ceilings in view:

| Boundary | Stock executable/calibration result | Meaning |
|---|---:|---|
| MAF transfer table | 65535 Q6 = 1023.984375 kg/h | Table encoding clips; later width cannot restore lost signal. |
| Direct normalized load | 32767 = 327.67% | Relative-load arithmetic limit, not calibration coverage. |
| Load-to-charge conversion | load 21485 -> `2ada=0xffff` | Charge sentinel with stock `c_maf_ref`. |
| Optional pressure-integrator inflow | `6c72 <= 32767` Q5 = 1023.96875 kg/h | Separate bit-1 path limit; direct bit-0 load route bypasses it. |
| Calibration lookup axes | e.g. spark 108.36%, torque 115% | Endpoint hold; not arithmetic saturation or safe operating limits. |

The broader inventory has 35 direct `2b30` readers in 23 functions; only `00435048` has a reviewed narrow role as the feedback copy. Do not infer each reader's physical purpose from the common input. The stock airflow correction axis ends at 800 kg/h and holds the last correction cell beyond its endpoint. The ideal pressure-ratio screen in the measured-MAF dossier is a representation-risk estimate, not engine output prediction.

## Evidence and unresolved boundaries

The principal source-bound records are the [measured-airflow decompilation](../msv70-maf-to-load-decompilation.md), [whole-program airflow/filling boundary](../msv70-airflow-filling-boundary.md), [MAF capture/decision contract](../msv70-measured-maf-envelope.md), and [load-continuation findings](../../analysis/derived/review-2026-09-04-load-continuation/findings.json). The MAF envelope remains `insufficient_evidence`; its templates are not a runtime result.

Unresolved: live selector/gate values and task timing; sensor voltage-to-ADC and physical MAF/housing transfer; whether `2b26` is accurate airflow; `2ac2` scale and override provenance; real RPM/IAT correction behavior; actual charge and volumetric efficiency; full consumer behavior outside mapped axes; and operating safety. Independent MAF, MAP, ambient pressure, post-intercooler IAT, lambda, and fuel-pressure measurements are necessary for a physical validation study. Numeric headroom is not a safe envelope or authorization to operate under boost.

## Related pages

- [Pressure model](pressure-model.md)
- [Fuel and injection](fuel-and-injection.md)
- [Torque and controls](torque-and-controls.md)
- [Calibration and limits](calibration-and-limits.md)
- [Logging and validation](logging-and-validation.md)
- [Glossary](glossary.md)
