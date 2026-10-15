# Issue #39 / #62 measured-MAF validation and decision workflow

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

This work extends the existing synchronized runtime-validation architecture from
issue #39. It does **not** introduce a second logger, database, relationship
graph, or clock model. Issue #39 remains the authority for capture provenance,
raw artifact retention, state widths, per-access signedness, instrument
calibration, fault status, and timebase acceptance. Issue #62 adds a focused
capture overlay and a deterministic decision analysis.

No artifact in this workflow authorizes a flash, positive-pressure operation,
diagnostic suppression, public-road full-load testing, or reduction of knock,
lambda, thermal, torque, or pressure protection.

## What is implemented

The separate private archive contains:

- `analysis/validation/msv70-maf-envelope-plan.json` (supporting artifact withheld from public edition) — the reviewed study and
  decision contract.
- `scripts/msv70_maf_envelope_capture.py` (supporting artifact withheld from public edition) — deterministic generation of a focused,
  structurally valid **not-run** issue-#39 capture manifest from the retained #39 example.
- `analysis/validation/msv70-maf-envelope-analysis.example.json` (supporting artifact withheld from public edition) — a hash-bound
  **not-run** analysis manifest.
- `scripts/msv70_maf_envelope.py` (supporting artifact withheld from public edition) — preparation, validation, and analysis CLI.
- `analysis/notebooks/msv70_measured_maf_envelope.ipynb` (supporting artifact withheld from public edition) — clean-kernel
  notebook for the current contract, planning envelope, and future retained
  results.
- `tests/test_msv70_maf_envelope.py` (supporting artifact withheld from public edition) — positive, negative, sentinel, scope, and
  deterministic contract tests.

There is still no runtime conclusion. The current outcome remains
`insufficient_evidence`.

## Why this is an overlay rather than a replacement

The existing issue-#39 protocol already solves the difficult bookkeeping
problems:

- exact `r13` state addresses and reviewed capture widths;
- per-instruction signed versus unsigned access semantics;
- raw artifact hashes and repository provenance;
- instrument calibration and pre-capture fault records;
- host-bracketed affine clock synchronization;
- support, contradiction, unresolved, and not-evaluated outcomes;
- a guard preventing correlation from creating `DERIVES_FROM` edges.

The issue-#62 overlay narrows that machinery to two issue-#39 selector
contracts: the airflow candidate/update gate and source-bound active filling
selection. Additional load, charge, lambda, fuel, ignition, knock, throttle,
Valvetronic, and VANOS comparisons remain issue-#62 dynamic observations. This
avoids pretending that selecting a large dossier observation automatically
makes every one of its states and instruments available.

## Focused state set

The capture overlay requires these raw states:

| State | Role in this study | Evidence boundary |
|---|---|---|
| `r13+0x1f62` | acquisition-availability gate: nonzero uses averaged/corrected acquisition; zero copies `2b30` into `2b26` | producer predicates are source-bound; installed state and physical truth still need observation |
| `r13+0x1e8c` | inner airflow-candidate selector | zero versus nonzero coverage only while `1fea == 0`; distinct from `0x1f62` |
| `r13+0x1fea` | selected-airflow update gate | zero updates; nonzero retains prior `2b30` |
| `r13+0x2ac2` | model/substitute-side selector candidate A, copied from `6ab4` | active producer/override and physical scale need binding |
| `r13+0x2b26` | corrected acquisition when `1f62 != 0`, otherwise feedback candidate | source route is recovered; physical MAF accuracy requires independent binding |
| `r13+0x2b30` | selected/retained airflow | conditioned load and bit-1 integrator routes exist; neither proves physical MAF provenance |
| `r13+0x6b5a` | modeled filling candidate A | physical scale not promoted; complete alternate airflow dependency not asserted |
| `r13+0x6b9a` | modeled filling candidate B | conditional airflow/integrator route recovered, inactive with canonical `CW_EISY=0` |
| `r13+0x6b6e` | converted/scaled filling (`filling_scaled_raw`) | `sat_u16((6b6a << 10) / 0x0ccd)`, never a selector |
| `r13+0x6b6a` | selected modeled filling | mixed signed/unsigned consumers |
| `r13+0x6ade` | selected normalized modeled filling | canonical startup/filter alternatives are inactive |
| `r13+0x6b62` | subtraction input before final adjusted-load publication | preserve signed access semantics |
| `r13+0x6b68` | filling-normalization denominator | capture alongside the numerator and selected result |
| `r13+0x6ab6` | adjusted-load state | selected by `CW_EISY` branch; mixed signed/unsigned consumers |
| `r13+0x2ada` | charge-like state | `0xffff` remains a sentinel, not maximum air mass |
| `r13+0x6984` | torque/load-related state | consumer-specific signedness must be preserved |
| `S_EVNN_AUS`, runtime `0x85e00b` | fixed filling-configuration byte | canonical file offset `0x5e00b` is zero: select NN `6b9a`; nonzero selects lift `6b5a` |
| `CW_EISY`, runtime `0x85e000` | fixed load-configuration byte | canonical file offset `0x5e000` is zero: modeled-load branch, not direct MAF selection |

The overlay does not rename these states from correlation. It consumes the
issue-#39 worksheet and unit/type registry.

Static review now proves the conditioned raw-state route
`2b30 -> 2b26 -> 6980 -> 6ac0 -> 6ab8 -> 6ab6`: the first edge requires
`1f62 == 0`, and the last requires `CW_EISY` bit 0 set. The complementary
bit-clear branch is `6b6a -> 6abe -> 6ade`, then lower-bounded
`6ade - 6b62 -> 6ab6`: canonical `S_RFST_ON=0` disables the nested startup
alternative and transition filter. This
workflow must still observe the active branches and bind `2b26` to independent
MAF before using the measured-MAF interpretation. Correlation cannot supply
that identity, and the static route does not authorize tuning.

The plan and generated analysis template retain `calibration_binding`, including
the AC1S source path, SHA-256
`757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`,
dataset ID, EPK `9PP92L00`, and each reviewed selector byte's address, width,
and value. These are retained static source facts, not a runtime capture.
The analyzer rejects a changed source binding, missing selector columns, or
aligned selector values that disagree with those fixed bytes. An arbitrary
per-row annotation cannot demonstrate another calibration branch.

For the unchanged canonical image, the active filling test compares `6b6a`
against `6b9a` and checks the saturating `6b6e` arithmetic. At least one sample
must distinguish the two candidates; identical candidates leave selection
unresolved. It reports `filling_selection.active_branch_status` separately from
`alternate_branch_status=untested`. Varying `6b6e` supplies no selector coverage,
and neither fixed calibration is required or authorized to change.

Runtime `1f62` coverage requires both zero and nonzero predicate outcomes;
two different nonzero byte values count as one outcome. `1e8c` uses the same
predicates, but only samples with `1fea == 0` can exercise its choice.
Selected-airflow fit, plateau, and association calculations use only full-load
samples with `1fea == 0` and `1e8c != 0`, without promoting the physical identity
of `2b26`. Retained nonzero-gate samples are excluded from measured-candidate
linearity/clipping attribution; changing `2b30` inside such a retained interval
blocks the analysis rather than describing continuous updates.

`load_selection` retains the canonical modeled branch and the untested
bit-set alternative. Its nested selection is statically resolved for the bound
image: `S_RFST_ON=0` selects `6abe -> 6ade` without the startup override/filter.
This is not runtime evidence of direct-MAF load selection, so the direct-MAF
architecture decision remains blocked even if transfer/headroom checks pass.

This analyzer answers a **direct-MAF load architecture** question. Its blocked
result is not a rejection of the separate native pressure-aware modeled route,
nor a reason to require a MAF-to-NN bridge. Establish the unchanged modeled
baseline first; compare alternate architectures under separately reviewed
source bindings, never by changing a selector merely to satisfy coverage.

The [later offline acquisition closure](../analysis/derived/review-2026-09-04-offline-maf-closure/findings.json)
identifies the acquired producer and a source-matched but untested local
MAF-range proposal. This does not change the analyzer contract or turn a
synthetic/retained sample into measured airflow. Do not unplug sensors, alter
selectors or manufacture unhealthy branch coverage to satisfy this study.
The current pass deliberately leaves runtime tooling and tests unchanged.

## Required channel set

The analysis needs DME/service channels and independently calibrated channels.

DME/service provenance:

- RPM, pedal, throttle and vehicle speed;
- gear;
- DME HFM table output in kg/h;
- HFM raw count and sensor voltage;
- Valvetronic lift;
- intake and exhaust VANOS;
- ignition timing and maximum cylinder knock correction;
- DME lambda target.

Independent instrumentation:

- manifold absolute pressure;
- ambient absolute pressure;
- fuel-rail absolute pressure;
- wideband lambda;
- post-intercooler IAT;
- independent MAF.

The independent MAF reference is the expensive/awkward part of the final
issue-#39 selector-gate contract. DME HFM output plus independently acquired
sensor voltage can complete a useful headroom baseline, but it cannot be
silently relabeled as an independent airflow reference. Without a characterized
external flow reference—or a separately reviewed alternative experiment—the
final #62 bridge/selector decision remains `insufficient_evidence`.

The decision code uses:

\[
\Delta P_\text{injector}
=
P_\text{rail,absolute}
-
P_\text{manifold,absolute}
\]

It does not substitute nominal rail pressure for measured injector
differential pressure.

## Capture sequence

### 1. KOEO HFM transfer repeat

Record at least two key-on/engine-off windows. This establishes zero-flow
voltage/count repeatability, ambient conditions, source health, and timestamp
behavior. It does not establish a complete physical MAF transfer.

### 2. Warm-idle repeatability

Record at least two stable warm-idle windows. Keep accessory state and thermal
conditions documented. This checks low-flow noise, retained-state behavior,
lambda provenance, and fuel-pressure provenance.

### 3. Stationary RPM holds

Record at least three bounded stationary holds using the existing issue-#39
procedure. These windows provide low-to-mid airflow excitation for scale, lag,
selector, and `mg/stk` convention checks.

### 4. Controlled MAF-availability observation

Record at least two approved bench or naturally aspirated service-state
windows that exercise the reviewed MAF-availability alternatives. Do not
disable diagnostics, falsify the MAF signal, or perform this test under boost.
The objective is branch discrimination, not making the engine operate through
an unsafe fault.

### 5. Naturally aspirated full-load envelope

The default study contract requires:

- six accepted windows;
- at least two gears;
- at least two sessions;
- at least two accepted windows per gear;
- at least 50 synchronized samples per window.

Use a dyno or an appropriate closed course with a qualified operator. The plan
explicitly sets public-road full-load authorization to `false`.

The numeric sample/window counts are experimental-repeatability defaults, not
firmware limits or engine-safety thresholds.

## Preparing the focused capture manifest

Generate a fresh not-run overlay:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_maf_envelope.py prepare-capture \
  --repo-root . \
  --output ./output/issue62-maf-capture.json
```

The generated manifest contains only the required issue-#39 observations,
states, perturbation classes, and issue-#62 channels. Populate a new directory
under:

```text
evidence/captures/issue-39/<capture-id>/
```

Then validate it through the issue-#39 validator:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_dynamic_validation.py validate-capture \
  --repo-root . \
  --capture evidence/captures/issue-39/<capture-id>/capture.json \
  --require-executed
```

A not-run template passing structural validation is not runtime evidence.

## Aligned-table contract

After issue-#39 capture acceptance, preserve the raw streams and create a
separate aligned CSV. The analysis manifest binds that CSV by path, byte size,
and SHA-256.

The default columns include:

```text
time_ns
window_id
session_id
scenario_id
gear
rpm
maf_table_kg_h
maf_raw_count
maf_sensor_voltage_v
independent_maf_kg_h
ram_r13_2b30_raw
ram_r13_1f62_raw
ram_r13_1e8c_raw
ram_r13_1fea_raw
cal_xdf_1e00b_raw
cal_xdf_1e000_raw
ram_r13_6b6e_raw
ram_r13_6b5a_raw
ram_r13_6b9a_raw
ram_r13_6b6a_raw
ram_r13_6ab6_raw
ram_r13_2ada_raw
ambient_kpa_abs
map_kpa_abs
fuel_rail_kpa_abs
lambda_measured
lambda_target
post_ic_iat_c
throttle_pct
pedal_pct
valvetronic_lift_mm
ignition_deg
knock_correction_max_deg
window_valid
```

The aligned CSV is a projection, not a replacement for raw samples. Raw
timestamps and original transport/sample order remain retained in the capture.

The semantic column keys are `filling_scaled_raw -> ram_r13_6b6e_raw`
(`raw_count`), `filling_configuration_selector -> cal_xdf_1e00b_raw` (`raw_u8`),
and `load_configuration_word -> cal_xdf_1e000_raw` (`raw_u8`). Filling candidate
A remains lift `6b5a`, candidate B remains NN `6b9a`, and selected filling is
`6b6a`. Calibration columns must project the exact source-bound bytes; they are
not mutable runtime switch requests. No old scaled-filling-as-selector alias
is accepted.

## Preparing the analysis manifest

Generate a fresh non-accepting template:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_maf_envelope.py prepare \
  --repo-root . \
  --output ./output/issue62-maf-analysis.json
```

Populate:

- executed capture-manifest paths and hashes;
- aligned CSV path, size, and hash;
- the declared target manifold absolute pressure;
- declared target post-intercooler IAT;
- an explicit VE-ratio assumption;
- required MAF table and raw-signal headroom;
- minimum acceptable injector differential pressure;
- maximum allowed lean lambda error;
- any reviewed minimum throttle threshold.

These values are not inferred from the firmware datatype.

For the pressure/temperature projection, the reference pressure is the
observed manifold absolute pressure in the high-airflow baseline rows, not
ambient pressure:

```text
projected MAF = observed stock p99 MAF
              × target MAP / observed high-airflow MAP
              × observed absolute IAT / target absolute IAT
              × declared VE ratio
```

Using ambient pressure in the denominator would understate the required
multiplier whenever the naturally aspirated engine has real inlet pressure
drop. The projection remains planning arithmetic, not boosted validation.

Validate before analysis:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_maf_envelope.py validate-analysis \
  --repo-root . \
  --analysis evidence/captures/issue-39/<capture-id>/issue62-analysis.json \
  --require-executed
```

## Analysis outputs

Run:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_maf_envelope.py analyze \
  --repo-root . \
  --analysis evidence/captures/issue-39/<capture-id>/issue62-analysis.json \
  --output-dir ./output/issue62-maf-result
```

The output directory contains:

- `summary.json`;
- `window_metrics.csv`;
- `runtime_association_metrics.csv`;
- `decision.md`.

The analyzer reports:

1. exact `id_maf_tab` representation margin;
2. raw-count or voltage headroom;
3. selected-airflow linearity and plateau evidence;
4. lag-bounded selected-airflow/filling/load/charge associations;
5. the working six-cylinder four-stroke dimensional check:

\[
\text{kg/h}
\approx
\frac{\text{mg/stk}\times6\times\text{RPM}\times30}{10^6}
\]

6. injector differential pressure;
7. delivered-versus-target lambda;
8. target-envelope airflow projection using observed high-airflow
   **manifold** pressure plus explicit target pressure, temperature, and
   VE-ratio assumptions;
9. zero/nonzero runtime predicate coverage, source-bound active filling
   selection, untested calibration alternatives, the load-source boundary,
   and scenario coverage;
10. evidence blockers, MAF-architecture rejection findings, and separate
    adverse stop conditions.

`0xffff` charge samples are excluded and counted as sentinels. They are never
scaled into a plausible-looking air mass.

## Decision logic

The permitted outcomes match issue #62.

### `insufficient_evidence`

Returned when captures, runtime predicate coverage, source-bound active
selection, load-source proof, scenario repetition, instrument calibration,
charge convention, raw-sensor headroom, target assumptions, or safety
requirements are incomplete. Untested fixed-calibration alternatives are
reported explicitly, not demanded as switch changes. With canonical
`CW_EISY == 0`, direct-MAF load selection remains unproven and the architecture
decision remains blocked.

### `stock_measured_maf_viable_for_declared_envelope`

Requires all study evidence, no observed clipping, acceptable selected-state
linearity, validated charge convention, adequate encoding and raw-signal
headroom, and passing declared fuel/lambda/throttle requirements.

This means only that the measured-MAF architecture passed the declared
envelope analysis. It does not authorize boost or prove every spark, lambda,
torque, diagnostic, monitor, and protection consumer.

### `characterized_housing_rescale_viable`

Requires both a retained MAF transfer characterization and a completed
consumer/plausibility audit. It is considered only for MAF-specific transfer or
encoding limits. The retained rescale evidence must also quantify encoding and
physical-signal headroom at the declared target. Raw signal headroom alone is
not enough, and a fuel, lambda, or torque problem cannot be relabeled as
evidence that a larger housing works.

A stock transfer/headroom failure with no completed rescale study remains
`insufficient_evidence`; it is not automatically promoted to either rescale
viability or wholesale rejection of measured MAF.

### `measured_maf_rejected`

Used when retained evidence rejects the measured-MAF architecture rather than
merely the stock transfer. Examples include selected-state clipping/nonlinearity
that a housing transfer cannot repair, or a completed quantitative rescale study
that still fails the declared encoding or physical-signal headroom requirements.

Fuel differential pressure, lean deviation, and unexpected throttle
intervention are reported separately as **adverse stop conditions**. They make
the declared run/envelope unacceptable but do not, by themselves, reject the
measured-MAF architecture or justify a rescale.

## Critical limitations

The analysis can support or contradict runtime hypotheses. It cannot prove a
static producer/consumer edge. Every lag or correlation row is marked
`dynamic_observation` and `causal_edge_created=false`. A contradiction to the
current issue-#39 selector hypothesis blocks the #62 decision and forces the
hypothesis to be revised; it does not, by itself, prove that measured MAF is
physically unusable.

A strong stock naturally aspirated envelope also does not prove boosted
protection behavior. Issue #38, pressure-sensor work, injector characterization,
and independent stop criteria remain separate gates.
