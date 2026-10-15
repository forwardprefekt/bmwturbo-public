# FIC delivered-set characterization and MSV70 mapping boundary

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Direct answer

The delivered FIC workbook contains the characterization needed to begin an
MSV70 injector **quantity/latency** mapping: measured flow at two differential
pressures, offset versus voltage, pressure dependence, minimum/short-pulse
behavior, and nonlinear-region examples. It is **not injection phasing**, it is
not ignition timing, and it is not an MSV70-ready tune.

The exact source, hashes, and handling rules live in the
`FIC 750 cc reference corpus` (supporting artifact withheld from public edition).
The original workbook remains the private characterization authority. Supplier
tables and numeric platform projections are withheld here pending an explicit
redistribution-rights decision; derivation alone does not establish permission.

## Canonical universal characterization

The retained workbook describes flow at two differential pressures, additive
offset versus voltage and pressure, and small-pulse `New`/`Original` pairs.
The numeric offset/flow/small-pulse tables are withheld from this public edition
because supplier-data redistribution rights remain unresolved. No public table
has been substituted or fabricated.

The adjacent “Injector Dead Time at 100%” field belongs to an **MS3
normalization block**, not a universal MSV70 dead-time or minimum-pulse
definition. The transformation direction and test conditions of the
`New`/`Original` small-pulse pairs remain unspecified.

## Why the platform sheets are not interchangeable

The workbook includes GM, Ford, Chrysler, and Nissan projections. They implement
different ECU models; a nominally similar pressure/voltage condition need not
produce an equivalent additive offset. Their copied comparison table and
platform-specific minimum/nonlinear pulse quantities are withheld pending rights
review. None is automatically the MSV70 `ip_ti_min` definition.

Selecting a familiar-looking table would invent an injector model without
proving its equation. The required method is one properly defined physical
characterization followed by a tested projection into the target ECU model.

## MSV70 mapping targets

The reviewed executable path is documented in
[`msv70-injector-pulse-path.md`](msv70-injector-pulse-path.md). The source data
must be mapped, not copied, into four relevant calibration families:

| MSV70 calibration | Physical source input | Required proof before values |
|---|---|---|
| `c_fac_mff_ti_stnd_1` and separate bank coefficient family | measured standard flow | binary divisors/scales and pointers recovered; supplier reference conditions and actual flow still required |
| `ip_fac_ti_prs` | flow change versus injector differential pressure | equation recovered; reviewed operands are modeled rail/manifold, not measured boosted pressure |
| `ip_ti_add_dly` | appropriately defined additive offset | biased signed 4 µs encoding and voltage source recovered; map is voltage-only, unlike a general pressure-dependent offset surface |
| `ip_ti_min` | acceptable minimum total command at stated error/repeatability | uniform uplift recovered; not a consumer of the FIC `New`/`Original` correction curve |

Injector differential pressure is **rail absolute pressure minus manifold
absolute pressure**. A nominal or gauge rail-pressure number alone is insufficient. With a
manifold-referenced regulator, rail pressure should track manifold pressure so
the differential remains approximately constant. With fixed rail pressure,
the differential falls as boost rises, changing both effective flow and
potentially latency.

The [offline equation closure](../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json)
recovers the standard-flow divisors, pressure arithmetic, voltage acquisition,
biased latency representation, bank/pointer selectors and minimum-pulse behavior.
It does not establish actual rail pressure, injector-terminal voltage or
energized behavior.

### Specific supplier/data boundary

- `3 Bar` and `42.5 psi` on the same source column are not exact equivalents;
  ask which differential was actually used on the bench.
- Define test fluid, temperature, driver and the physical meaning of the
  additive offset. The universal voltage rows and platform-specific extended
  voltage rows are not interchangeable without those definitions.
- Define the `New`/`Original` transformation direction, effective versus total
  pulse, voltage, differential pressure and error/repeatability criterion.
- The documented OEM 5-bar returnless topology and canonical approximately
  5–6-bar differential axis require characterization **above 4 bar**, including
  the nonboosted envelope, unless a different verified topology is selected.
- The stock delay map is voltage-only. A variable-pressure latency surface
  cannot be represented exactly by copying either FIC pressure column or
  folding its additive offset into a multiplicative flow correction.
- Recovering equations does not supply these physical definitions. Exact
  coefficient conversion also must not use the rounded six-decimal XDF slopes;
  the source report gives the explicit unit convention and binary divisors.

## What this data does not solve

The workbook does not establish:

- start/end of injection or phase feasibility versus crank angle;
- ignition advance or knock-safe timing;
- N52 rail/manifold fitment and nozzle position;
- fuel/material compatibility beyond an explicit supplier statement;
- actual base pressure or regulator behavior on the car;
- coil current and MSV70 driver electrical margin;
- safe maximum pulse width or boosted duty margin; or
- correct cylinder-to-cylinder compensation for this exact set.

No individual six-injector serial/flow-match report or material-compatibility
statement was located in the workbook's seven visible sheets. Supplier claims
that the set is dynamically matched do not replace those absent per-injector
numbers.

## Safe next implementation slice

Before proposing calibration values:

1. Measure and record base rail pressure, manifold pressure, and regulator
   reference topology; compute differential pressure at each condition.
2. Use the recovered equations and resolve supplier/mapping definitions,
   then separately authorize deterministic conversion and behavior checks.
3. Verify fitment, connector retention, seal spacing, and nozzle protrusion.
4. Validate naturally aspirated idle/cruise first with synchronized DME pulse,
   battery voltage, rail pressure, independent MAP, wideband lambda, trims,
   RPM, and temperature channels.
5. Treat boost as a separate acceptance phase after leak, wastegate/base
   pressure, fuel-pressure, wideband, oil, coolant, and conservative load gates
   pass.

Until those steps are complete, the universal 3-bar or 4-bar table is a physical
reference selected by measured differential pressure—not authorization to flash
MSV70 values.
