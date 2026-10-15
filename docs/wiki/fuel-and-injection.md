# Fuel and injection

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page follows the canonical 2006 E90 330i N52 MSV70 SW0049PP0L921S0AC1S fuel request through modeled corrections, injector-flow conversion, minimum-pulse treatment, staging, and timer dispatch. The static software path is substantially recovered; selected runtime branches and all physical injector behavior remain unverified. This is a reference, not an injector selection or calibration authorization.

## Evidence boundary

The primary source is the [cylinder-mass to injector-pulse dossier](../msv70-injector-pulse-path.md), which records reviewed instructions and calibration components for the canonical ROM. Its [equation closure](../../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json) resolves standard-flow arithmetic, selectors, pointers, pressure arithmetic, voltage acquisition, and minimum-pulse behavior. The [FIC 750-6049 reference](../fic-750-6049-msv70-calibration-reference.md) documents supplier data and its platform-mapping boundary. XDF labels and units indicate calibration intent, not physical proof.

## From cylinder mass to requested fuel

The operational cylinder-mass writer reads adjusted load `ram:r13:6ab6`, converts it with `c_maf_ref` at `0x844bbe` and denominator 10000, and stores saturated unsigned-16 state `ram:r13:2ada`. The neighboring pressure-side operation writes `2aca`; it does not write the cylinder-mass coordinate. The runtime-selected load producer and physical meaning of either coordinate are separate questions. See the [reviewed pulse path](../msv70-injector-pulse-path.md#reviewed-main-pulse-path) and the [airflow/load reference](airflow-and-load.md).

With stock `c_maf_ref=30503`, adjusted-load raw 21484 produces charge 65533; load 21485 produces the 65535 saturation sentinel. These are load-to-charge inputs and outputs, not a conversion from cylinder charge to requested fuel.

The next stage has a different coefficient and purpose. With its gate open, `00457f14` combines cylinder charge `2ada` with `c_mff_fac` at runtime `0x844bd2` (XDF intent 1/14.7) and writes base requested fuel mass `2d6c`. The [reviewed pulse path](../msv70-injector-pulse-path.md#reviewed-main-pulse-path) keeps these adjacent conversions separate.

### Requested-mass gate reported by the archive

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `func:00457f14`, `00457f24–00457f50`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

A nonzero byte at `1fe0` selects zero; zero permits the charge/coefficient helper call. The two `rlwinm` instructions retain the helper result's low 16 bits and then shift that unsigned value right by three before storing `2d6c`. This is the requested-fuel stage, not the earlier `c_maf_ref` load-to-charge conversion.

The control path next applies `c_fac_mff_ti_stnd_1` to form `2da4`, then selector-dependent fast/slow and positive/negative wall-film branches to signed state `2daa`, scaled into `2da2`. Pointer-selected wall-film calibration families are not proven active at runtime. Bank factors, raw bank controller states `41e6/4332` copied to `3dda/3f2a`, mapped correction `2d92`, and wall-film correction feed bank requests `2d6e/2d70`; operation-mode and per-cylinder factors then produce inputs consumed by the main-pulse calculation. Those state bridges are instruction-reviewed; upstream physical identities and the live selection of branches remain open. See the [injector dossier](../msv70-injector-pulse-path.md#reviewed-main-pulse-path) and [lambda/protection reference](ignition-lambda-and-protection.md).

## Injector flow and differential pressure

The software combines requested mass with standard-flow, pressure, and cylinder factors, then accounts for signed dead-time and correction terms. The scalar calibration `c_fac_mff_ti_stnd_1` at `0x844ac0` (value 52643) is distinct from bank-array element `c_fac_mff_ti_stnd[1]` at `0x845aae` (value 26322); their paths use different binary divisors. Do not alias the scalar and bank array or derive new coefficients from rounded six-decimal XDF slopes: under the documented mass/time convention, those displayed slopes differ from dimensional closure by about 4.18%. The exact scales and divisor evidence are in the [equation closure](../../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json).

Injector differential pressure means rail absolute pressure minus manifold absolute pressure. The software correction uses modeled rail and manifold operands: the rail model is constructed from calibrated nominal pressure, ambient and fuel-demand correction, not a rail ADC. Its subtraction does not prove the physical identities of raw operands. The stock pressure axis is approximately 5–6 bar differential; below its lower endpoint the first factor holds at unity rather than extrapolating. Neither this axis nor modeled pressure is evidence of actual rail pressure, boost tracking, injector flow, or a safe operating envelope. Physical rail topology and differential must be measured independently. See the [injector dossier](../msv70-injector-pulse-path.md#pressure-semantics-inherited-from-issue-35), [FIC mapping boundary](../fic-750-6049-msv70-calibration-reference.md#msv70-mapping-targets), and [pressure-model reference](pressure-model.md).

## Voltage compensation and minimum pulse

Logical ADC `0x17`, its eight-sample ring and reviewed consumers produce software voltage state `1fb4`. The voltage-only `ip_ti_add_dly` map uses biased signed 4-microsecond encoding; the calculation subtracts bias 32768, adds signed temperature/mass correction `7496`, and writes `323a`. This describes software arithmetic, not measured voltage at injector terminals, actual latency, or driver behavior. The stock latency map has no pressure coordinate, so a pressure-dependent supplier offset cannot be represented exactly by copying one pressure column or folding offset into a multiplicative flow factor. See the [injector dossier](../msv70-injector-pulse-path.md#reviewed-main-pulse-path) and [FIC reference](../fic-750-6049-msv70-calibration-reference.md#msv70-mapping-targets).

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `func:0043cce0`, `0043ccf8–0043cd10`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

The voltage coordinate is loaded as an unsigned byte. After lookup, `subi` removes the `0x8000` bias and `extsh` sign-extends the low halfword; `lha` also loads correction `7496` as signed. The helper result is stored as a halfword at `323a`. Treating either correction as unsigned would misread this software path; the reported software path does not establish physical injector delay.

The minimum operation adds a uniform delta to the selected cylinders rather than independently flooring each pulse. It can use a minimum retained from a prior accumulation window. The following raw-count examples derive from the [documented equation](../../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json); they are not injector measurements.

| Preliminary values | Configured minimum | Minimum used by the calculation | Final calculation outputs |
|---|---:|---:|---|
| 161, 321 | 325 | 161, from the current window | 325, 485 |
| 161, 321 | 325 | 100, retained from an earlier window | 386, 546 |

This behavior is not a nonlinear short-pulse correction curve or proof of repeatable physical injection. The [supplier-data boundary](../fic-750-6049-msv70-calibration-reference.md#specific-supplierdata-boundary) explains the missing characterization.

## FIC 750-6049: useful data, not a drop-in calibration

The workbook contains universal flow values and voltage-dependent offset rows at two labeled pressures:

| Source pressure labels | Displayed mass flow | Displayed imperial flow |
|---|---:|---:|
| 3.0 bar / 42.5 psi | 9.67 g/s | 76.78 lb/hr |
| 4.0 bar / 58 psi | 11.10 g/s | 88.08 lb/hr |

The paired pressure labels are reproduced from the supplier workbook, not asserted equivalent: 3.0 bar is not exactly 42.5 psi. The actual bench differential needs clarification. Vendor-labeled `New`/`Original` small-pulse pairs also lack a confirmed transformation direction and physical definition. The adjacent `1.00 ms` “Injector Dead Time at 100%” belongs to the MS3 normalization block, not an MSV70 dead-time or minimum-pulse definition. See the [FIC characterization](../fic-750-6049-msv70-calibration-reference.md#canonical-universal-characterization), [supplier questions](../fic-750-6049-msv70-calibration-reference.md#specific-supplierdata-boundary), and source corpus (supporting artifact withheld from public edition).

FIC data can inform characterization inputs for standard flow, pressure dependence, offset, and nonlinear short-pulse behavior. Values require a deliberate projection into MSV70's separate standard-flow coefficients, `ip_fac_ti_prs`, voltage-only `ip_ti_add_dly`, and uniform-minimum `ip_ti_min`. Platform-specific GM/Ford/Chrysler/Nissan sheets implement other ECU models and are not interchangeable. The workbook does not establish target-vehicle fitment, N52 rail/manifold suitability, pressure topology, injector-terminal voltage, driver margin, per-cylinder match, duty margin, or safety. Its 3/4-bar characterization does not alone cover the stock approximately 5–6-bar differential axis; characterize the relevant range and confirm supplier definitions before mapping. See [MSV70 mapping targets](../fic-750-6049-msv70-calibration-reference.md#msv70-mapping-targets) and [what the workbook does not solve](../fic-750-6049-msv70-calibration-reference.md#what-this-data-does-not-solve).

## Staging and timer boundary

`30f6..3100` hold six final main-pulse calculation outputs; they are not by themselves final timer commands. For each cylinder, `0043d6fc` copies the calculation output into staging only when its byte selector `320e..3213` is zero; otherwise an alternate source is staged. `0043d760` converts selected staged data using a two-stage truncating conversion through `004342f0`, clamps at `0xffff`, and writes branch-contingent candidate timer counts. A later selector/record routine may select another word, modify packed data, or return without dispatch. The software timer boundary is therefore downstream and branch-dependent; final physical energized timing is not observed. See the full [timer-path account](../msv70-injector-pulse-path.md#reviewed-main-pulse-path) and [equation closure](../../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json).

Software staging is not proof that an injector was energized. Shared low-level dispatch routines do not establish a mapping from ignition channels to injector channels; keep ignition channel mapping separate. No physical timer/register semantics, injector latch, coil current, phase feasibility, or duty-cycle proof follows from software addresses. See the [ignition/protection reference](ignition-lambda-and-protection.md) and [hardware and recovery](hardware-and-recovery.md).

## Current unknowns

- Runtime-active load producer, lambda/enrichment branch, bank factors, and pointer-selected wall-film family.
- Actual rail and manifold pressure, differential-pressure tracking, fuel temperature, injector-terminal voltage, and latency under operating pressure.
- Injector identity, flow/latency characterization under target conditions, fitment, material compatibility, individual matching, and electrical compatibility.
- Runtime selector values and whether each candidate timer path reaches dispatch; timer hardware semantics and energized output.
- Physical start/end of injection, phase feasibility, injector duty margin, and any safe operating envelope.

## Related pages

- [Airflow and load](airflow-and-load.md), [pressure model](pressure-model.md), and [torque and controls](torque-and-controls.md).
- [Ignition, lambda, and protection](ignition-lambda-and-protection.md), [calibration and limits](calibration-and-limits.md), and [logging and validation](logging-and-validation.md).
- [Evidence rules](evidence-and-tools.md) and [open questions](open-questions.md).
