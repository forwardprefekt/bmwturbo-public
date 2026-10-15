# MSV70 synchronized dynamic-validation protocol

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.
> The target serial is intentionally withheld. Any private diagnostic workflow
> still requires an exact match to its independently recorded authorized target;
> the public text does not supply or relax that identity gate.

Issue #39 (supporting artifact withheld from public edition) owns the
runtime phase of the static dossiers. This document describes the portion that
can be completed before sensors and capture data exist. **Actual capture data
remains pending.** The protocol does not authorize firmware flashing, diagnostic
suppression, actuator bypass, or a boosted run.

## Current result

The separate private archive has one generated, machine-readable worksheet compiled from
the existing **24 planned observations**, the issue-#39 initial state set, and
the authoritative issue-#27 access facts:

| Item | Current count/status |
|---|---:|
| Planned observation contracts | 24, all `not_run` |
| Unique r13 RAM states | 242 |
| Exact direct RAM-access rows | 2,344 |
| Source globals | 5 |
| Exact direct global-access rows | 43 |
| Canonical capture channels | 25 |
| Reviewed indirect/mixed-width capture views | 73 |
| Unresolved static address/width views | 0 |

Seventy-two states are indirect array/record addresses without direct rows in
`analysis/facts/msv70/ram-accesses.csv` (supporting artifact withheld from public edition). The existing injector views cover six
byte staging selectors, six halfword timer-count candidates, and thirty byte
dispatch selectors. Fresh knock evidence adds eighteen threshold/feedback/
correction bytes, six acquisition halfwords, and six record-mode bytes.
Filtered words `2d48..2d52` and selected dispatch bytes `1feb..1ff0` already
have direct facts and do not need invented indirect memberships. The remaining
mixed-width state, `ram:r13:5d74`, retains its reviewed byte/bit-6 view.
These views are not direct-access facts or intrinsic state-wide signedness.
Logger configuration artifacts and actual update rates remain pending.

The issue-#39 initial states `ram:r13:1f62` and `ram:r13:6abe`, which were not
named in a dossier observation, are still included by the explicit initial-set
union. All other worksheet states come from the issue-#34 through issue-#38
handoffs. Pressure observations that deliberately have no instruction-backed
RAM ID remain blocked rather than receiving a guessed Ps/Pvdk alias.

## Artifact boundaries

- `analysis/validation/msv70-dynamic-observations.jsonl` (supporting artifact withheld from public edition) remains the dossier
  handoff source. It is not duplicated or rewritten by the compiler.
- `analysis/validation/msv70-dynamic-validation-protocol.json` (supporting artifact withheld from public edition) contains the
  canonical channel aliases, safe perturbations, clock contract, and
  predeclared support/unresolved rules.
- `analysis/validation/msv70-dynamic-validation-worksheet.jsonl` (supporting artifact withheld from public edition) is the
  deterministic generated projection. It binds the observations, #27 RAM and
  global facts, capture schema, and relationship manifest by SHA-256.
  Supplied SHA-256 bindings on reviewed state-view evidence are checked against
  exact file bytes before a capture view is admitted, even when line counts
  are unchanged.
- `analysis/validation/msv70-capture-manifest.schema.json` (supporting artifact withheld from public edition) is a standard JSON
  Schema Draft 2020-12 envelope for retained captures. The semantic validator
  adds repository hashes, safe paths, exact channel/state coverage, artifact
  integrity, and timebase checks that JSON Schema alone cannot express.
- `analysis/validation/msv70-capture-manifest.example.json` (supporting artifact withheld from public edition) is a structurally
  valid `not_run` template. It is not acceptance evidence.
- `analysis/validation/msv70-can-capture-scenarios.json` (supporting artifact withheld from public edition) converts the generic
  perturbation classes into five required stationary pre-swap recordings, an
  optional hot restart, and one optional gentle road/load comparison. Its
  numeric setpoints are operator-procedure defaults, not proven firmware or
  safety thresholds.
- `analysis/validation/msv70-can-session-log.example.csv` (supporting artifact withheld from public edition) is the minimal
  operator filename/deviation sheet. The corresponding
  [CAN baseline runbook](msv70-can-baseline-capture-runbook.md) defines what
  raw CAN can and cannot validate and keeps fault-injection tests deferred
  until internal logging can answer them.
- `analysis/validation/msv70-memory-probe-plan.json` (supporting artifact withheld from public edition) is the deterministic,
  source-bound identity, F0, minimal-RAM, focused-RAM, and benchmark plan.
  `scripts/msv70_memory_probe.py` (supporting artifact withheld from public edition) executes only its read-only allowlist and
  retains raw process, telegram, trace, hash, and aggregate-timing evidence.
  Its synthetic contract tests are in `tests/test_msv70_memory_probe.py` (supporting artifact withheld from public edition).
- The [memory capture runbook](msv70-memory-capture-runbook.md) defines the
  next-session operator sequence, success ladder, exact blockers, immutable
  outputs, and the persistent per-job timestamp logger required after access is
  proven. EdiabasTest wrapper output remains transport evidence, not an
  issue-#39 accepted capture.
- `evidence/captures/issue-39/` (supporting artifact withheld from public edition) is the future capture root. Raw capture bytes
  remain separate from normalized or analysis outputs.

This uses the existing issue-#22 logger architecture and existing relationship
graph. It does not reinvent a second DAQ, database, or graph server.

## DME acquisition implementation gate

The immediate blocker is not another address list. It is a demonstrated,
read-only DME transport that can return a grouped or event-driven snapshot with
known sample timing. The first implementation should target only the following
17 direct, single-width states:

| State ID | Absolute address | Width | First-pass purpose |
|---|---:|---:|---|
| `ram:r13:1e8c` | `0x007f9e8c` | 1 byte | measured/model selector |
| `ram:r13:1f62` | `0x007f9f62` | 1 byte | MAF acquisition/availability state |
| `ram:r13:1fea` | `0x007f9fea` | 1 byte | selected-airflow write gate |
| `ram:r13:2ac2` | `0x007faac2` | 2 bytes | modeled/fallback airflow candidate |
| `ram:r13:2ada` | `0x007faada` | 2 bytes | derived cylinder-mass coordinate |
| `ram:r13:2b26` | `0x007fab26` | 2 bytes | corrected measured-MAF candidate |
| `ram:r13:2b30` | `0x007fab30` | 2 bytes | selected-or-retained airflow |
| `ram:r13:6984` | `0x007fe984` | 2 bytes | RPM coordinate |
| `ram:r13:6ab6` | `0x007feab6` | 2 bytes | principal corrected relative-filling/load candidate |
| `ram:r13:6abe` | `0x007feabe` | 2 bytes | pre-correction signed-saturated relative-filling candidate |
| `ram:r13:6ade` | `0x007feade` | 2 bytes | selected normalized modeled-load intermediate |
| `ram:r13:6b5a` | `0x007feb5a` | 2 bytes | lift-based fallback filling candidate |
| `ram:r13:6b62` | `0x007feb62` | 2 bytes | signed correction subtracted before the final load floor |
| `ram:r13:6b68` | `0x007feb68` | 2 bytes | modeled-filling normalization denominator |
| `ram:r13:6b6a` | `0x007feb6a` | 2 bytes | selected modeled-unthrottled relative filling |
| `ram:r13:6b6e` | `0x007feb6e` | 2 bytes | converted/scaled filling companion |
| `ram:r13:6b9a` | `0x007feb9a` | 2 bytes | neural-network filling candidate |

All addresses use the proven `0x007f8000 + displacement` relation. Widths are
the direct access widths needed by these observations, not a claim that every
firmware access to each address has the same signed interpretation.

### Structure-730 neighborhood and canonical-code fingerprints

Every focused address is now checked against all A2L measurements within
`±0x80` and against every canonical direct-access row. This avoids both failure
modes: importing an old exact-address symbol as current truth, or discarding a
nearby old layout that still exposes useful subsystem alignment.

| State | Exact/nearby structure-730 evidence | Canonical direct-access fingerprint | Alignment decision |
|---|---|---|---|
| `1e8c` | exact byte `cyl_lam_psa_2.lv_crk_cyl_lam_int_sel`; old lambda flags at `-4..+4` | 1 `lbz`, 2 `stb`; 3 functions | width match inside the wrong subsystem; revision collision |
| `1f62` | exact byte `lv_var_lsh_down`; hardware-variant flags at `-4..+4` | 10 `lbz`, 3 `stb`; 12 functions | width match only; old exhaust-variant name conflicts with canonical MAF-state use |
| `1fea` | exact byte `lv_clc_2seg_ena`; `lv_clc_2seg` at `+1` | 7 `lbz`, 2 `stb`; 8 functions | strongest exact semantic corroboration |
| `2ac2` | no exact value; `fac_maf_rel -6`, `maf_max_cor -4`, `isapwm -2`, predicted `maf +4` | 3 `lhz`, 1 `sth`; 3 functions | old substitute-airflow cluster supports a small local shift, not an exact name |
| `2ada` | no exact value; neighborhood crosses airflow/pressure, voltage, then torque | 80 `lhz`, 2 `sth`; 37 functions | high-fanout canonical state; old proximity does not select one symbol |
| `2b26` | exact `prs_up_thr_max`; `maf_mmv -0x0a`, followed by the old pressure cluster | 4 `lhz`, 1 `sth`; 4 functions | exact old pressure name conflicts; shifted MAF neighbor is useful alignment evidence |
| `2b30` | exact signed `v_map_dif_vld`, `-2.5..2.5 V` at `0.0048828125 V/count`; `maf_mmv -0x14` | 35 `lhz`, 2 `sth`; 25 functions | decisive exact-address and signedness/physical-domain collision; preserve it as version evidence |
| `6984` | no exact value; old neighborhood contains rough-running `Lur*` states | 379 `lha`, 2 `lhz`, 2 `sth`; 88 functions | substantial local layout divergence from the canonical RPM coordinate |
| `6ab6` | exact unsigned `Msdk_ur`, throttle mass flow | 36 `lha`, 1 `lhz`, 3 `sth`; 25 functions | related air/filling domain, but canonical consumers reinterpret it as adjusted load |
| `6abe` | exact unsigned `Msndk_roh`, uncorrected normalized throttle mass flow | 1 `sth`; 1 function | semantically adjacent to pre-correction filling; producer behavior supports, but does not prove, an exact name |
| `6ade` | exact `Dpirgn`, a pressure difference | 1 `lha`, 3 `sth`; 1 function | canonical selected modeled-load intermediate; old pressure name/scale are not transferable |
| `6b5a` | exact signed `Mareg_dknn`; neural/controller values at `-8..+8` | 3 `lhz`, 3 `sth`; 2 functions | shared controller neighborhood, but canonical selection and function grouping support the `Rfv_ist_hub` candidate instead |
| `6b62` | exact `Mshfm_kor`, corrected HFM flow | 6 `lha`, 1 `sth`; 5 functions | canonical signed final-load correction; revision collision |
| `6b68` | no exact measurement | 13 `lhz`, 3 `sth`; 8 functions | canonical modeled-filling denominator; no exact physical alias assigned |
| `6b6a` | no exact value; gap after `Mshfm_kor -8`, before `Psreg_out +0x42` | 1 `lha`, 11 `lhz`, 2 `sth`; 11 functions | canonical selector code plus `EISY_EV` projection outweigh nearest-address naming |
| `6b6e` | no exact value in the same gap | 1 `lhz`, 2 `sth`; 2 functions | canonical saturating conversion defines the role; old neighborhood constrains only the domain |
| `6b9a` | no exact value; nearest forward old state is `Psreg_out +0x12` | 1 `lhz`, 2 `sth`; 1 function | canonical producer and neural-filling group fingerprint are stronger than proximity |

The complete relation rows, conversions, limits, CCP declarations, A2L
function memberships, canonical function counts, and representative access
sites are retained in
`focused-ram-neighborhood-measurements.jsonl` (supporting artifact withheld from public edition)
and
`focused-ram-neighborhoods.jsonl` (supporting artifact withheld from public edition).

Three existing observation contracts form the focused first pass:

| Observation ID | Required DME states | Independent external channels |
|---|---|---|
| `issue34-airflow-selector-gates` | `1e8c`, `1fea`, `2ac2`, `2b26`, `2b30` | MAF, RPM, throttle |
| `issue34-filling-candidate-selection` | `6b5a`, `6b6a`, `6b6e`, `6b9a` | absolute MAP, ambient pressure, post-intercooler IAT, RPM |
| `issue34-modeled-load-selection` | `2b30`, `6b5a`, `6b9a`, `6b6a`, `6b68`, `6abe`, `6ade`, `6b62`, `6ab6` | absolute MAP, MAF, RPM, throttle, wideband lambda |

The external-channel union is therefore independent MAF, absolute MAP, ambient
pressure, post-intercooler IAT, RPM, throttle, and wideband lambda. Pedal, raw
stock-MAF voltage, coolant, battery, Valvetronic/VANOS state, and raw CAN are
recommended context channels but are not substitutes for that union.

The modeled-load observation replaces the obsolete missing-bridge hypothesis.
The conditional `6ac0 -> 6c6e -> 6c6a -> 6ac2 -> 6b9a` route is now recovered,
but canonical `CW_EISY=0` leaves its bit-1 airflow input inactive. The stock
baseline instead binds `S_EVNN_AUS=0`, `CW_EISY=0`, `S_RFST_ON=0` and
`K_RFMIN_EISY=600` to the normalized/corrected `6b9a -> 6ab6` route. Changing
firmware to exercise the alternative is outside this capture protocol.

### Transport decision order

The retained host inputs now define the first probe path, but they do not waive
the live acceptance gate:

1. run the catalog-derived `IDENT_AIF` job and retain every ECU-identification
   and historical AIF result set;
2. require `SERIENNUMMER_LESEN.SERIENNUMMER == locally_recorded_expected_serial` and
   `DATEN_REFERENZ_LESEN.DATEN_REFERENZ == 0049PP0L921S0AC1S`; stop on either
   mismatch, and retain `ZIF_LESEN` plus `HARDWARE_REFERENZ_LESEN` for
   program/hardware correlation;
3. only after the DREF matches, run `SPEICHER_LESEN_ASCII` with
   `SEGMENT=LAR`, `ADRESSE=0x840008`, and `ANZAHL=8`; require
   `DATEN_ASCII == 9PP92L00`;
4. through `STATUS_MESSWERTBLOCK_LESEN`, define/read the eight documented
   common IDs `0x4807`, `0x4200`, `0x4201`, `0x4202`, `0x4203`, `0x4205`,
   `0x4600`, and `0x5A0C`, then reuse/read the same F0 block while retaining
   raw request/response telegrams and host brackets;
5. only after that passes, compare one harmless known RAM value through
   `SPEICHER_LESEN`; and
6. pursue CCP DAQ only after a matching firmware description or live
   compatibility proof, endpoint/routing confirmation, and the required
   security implementation are retained.

The generated runbook derives each identity job from the PRG's concrete result
signature and fails generation if any referenced job or required field is
absent. `INFO` is PRG build metadata, not an executable job.

Static inspection rejects direct use of the A2L addresses. Its EPK is
`9PP73000`; the canonical external/LAR image contains `9PP92L00` at runtime
`0x840008`. The file is still a high-value semantic donor: it contains 13,380
measurements, 8,072 characteristics, CCP 2.1 DAQ, ETK, and KWP2000 metadata.
Exactly 7,992 characteristic names crosswalk to the canonical XDF, with 7,865
of 7,867 comparable linear conversions and 7,864 units agreeing. Names,
descriptions, scaling, and function grouping are evidence; its RAM and
calibration addresses are not.

The PRG is self-contained: all 331 jobs decode, all are documented, and its
`USES` count is zero. It contains 82 tables, 465 `MESSWERTETAB` rows, a dynamic
F0 measurement job using `$2C F0 04`, `$2C F0 02`, and `$21 F0`, plus generic
`$23 ReadMemoryByAddress` jobs for 1--254 contiguous bytes. No companion `.GRP`
is required for these direct jobs. Defining F0 changes ephemeral
diagnostic-session state; it does not persist calibration, adaptation, or
flash state, but it is not a zero-state-change read.

The focused set is 31 bytes and statically fits five seven-byte CCP ODT
payloads; the A2L's 10 ms DAQ list permits ten ODTs. That makes CCP the best
event-driven candidate, not a usable endpoint: the EPK mismatch, unproven live
CAN routing, and missing `internal.dll` seed/key and `CRC16.DLL` checksum
implementations remain blockers. ETK metadata requires compatible ETK
hardware. The PRG and A2L analyses, complete searchable catalogs, full BEST/1
disassembly, and exact derived hashes are retained under
`analysis/derived/msv70-host-inputs/` (supporting artifact withheld from public edition).

### Read-only transport acceptance probe

A transport passes the focused probe only when retained raw output shows:

- all 14 values returned at their exact widths without memory writes, flashing,
  reset, adaptation changes, or diagnostic suppression;
- one ECU event/DAQ sample for the group, or a measured maximum intra-sample
  skew small enough for the target observation;
- monotonically increasing sample index or transport sequence, with drops and
  reordering detectable;
- measured sample period, update period, latency/bracketing uncertainty, and
  maximum intra-group skew; and
- stable repetition during a stationary warm idle window.

Sequential per-address KWP polling may establish steady-state values. It cannot
prove within-cycle selector or branch ordering unless the complete polling
span is measured and is below the observation's accepted error bound. A
vehicle DME patched solely to add logging also violates this protocol's
stock-firmware capture boundary.

## Address, width, and per-access signedness

For every RAM state, the compiler records:

```text
absolute address = 0x007f8000 + r13 displacement
```

Every available #27 fact is projected exactly once with its function,
instruction, source CSV row, evidence kind, access kind, width, mnemonic, and
instruction text. Signedness is **per access**:

- `lha` / `lwa` reads are signed;
- `lbz` / `lhz` / `lwz` reads are unsigned or zero-extended at that access;
- stores have no consumer read interpretation; and
- an unclassified read mnemonic remains `explicit_unknown`.

The state-level row lists observed widths but never turns them into a universal
signedness. Update rate is `not_measured` for every state until a real capture
establishes sample/update behavior.

## Predeclared outcomes

Each observation has four allowed outcomes:

1. `support` — the exact predeclared relationship is compatible with the
   captured branch, timing, scale, and independent instrument uncertainty;
2. `contradict` — the original dossier falsification condition is met;
3. `unresolved` — branch coverage, calibration, timebase, producer identity,
   excitation, or uncertainty cannot distinguish the hypotheses; or
4. `not_evaluated` — no accepted capture window exists.

The decision rule is fixed before collection. **Correlation cannot create** a
`DERIVES_FROM` edge or replace instruction evidence. A confidence change must
cite exact raw and analysis artifact hashes, while the capture manifest's
relationship guard must show that correlation analysis did not modify the
reviewed relationship manifest.

The filling observation binds the captured firmware identity and the fixed
`S_EVNN_AUS` calibration byte: canonical zero selects `6b9a -> 6b6a`.
Support requires distinguishable candidate values and the separate scaled
output `6b6e = sat_u16((6b6a << 10) / 0x0ccd)`. The unchanged image need not
exercise both calibration values; the alternate branch remains untested.
`CW_EISY=0` selects the modeled load topology, not the direct-MAF branch.
The [MAF envelope analyzer](msv70-measured-maf-envelope.md) retains that
architecture boundary and tests runtime flags as zero/nonzero predicates,
using `1e8c` and changing selected-airflow evidence only when `1fea == 0`.

## Host-bracketed affine timebase

The planned alignment method is **host-bracketed affine** mapping, not an
assumption that USB, CAN, DME, and ESP32 timestamps already share a clock:

Both streams preserve their raw timestamps before fitting or resampling.

1. the external logger preserves its monotonic device timestamp and emits
   sequence-numbered synchronization records;
2. the host brackets receipt of those records with `CLOCK_MONOTONIC_RAW`;
3. DME samples retain their host monotonic timestamp, original sample index,
   and transport order;
4. anchors cover the start, periodic intervals, and end of the capture;
5. a weighted affine logger-to-host clock model records offset, scale, drift,
   median/95th/max residual, and a conservative alignment-error bound; and
6. the bound must be no greater than one quarter of the shortest relevant
   measured event. If event duration or state update rate is unknown, the
   observation cannot pass.

At least 20 anchors are required by the current contract. That number is a
preflight minimum, not proof of synchronization. Transport asymmetry, receive
bracketing, clock quantization, DME sample latency, drift, dropped records, and
reordering remain explicit uncertainty terms. The actual DME logger and ESP32
integration must demonstrate this method; current validation status is
`not_run`.

### Alignment implementation

For each logger synchronization exchange, retain
`sequence`, device time `d_i`, host `h_before_i`, host `h_after_i`, and status.
Use `h_i = (h_before_i + h_after_i) / 2` as the host anchor and at least half
the bracket span as its uncertainty. Fit the weighted model
`h = offset + scale * d`; do not rewrite either raw clock. Retain fit inputs,
weights, coefficients, drift in ppm, residual distribution, rejected-anchor
rule, and rejected rows. Start, periodic, and end anchors must total at least
20 after rejection.

Stamp each DME group with host `CLOCK_MONOTONIC_RAW` before and after the
transport read or receive operation, plus ECU/transport sequence when
available. The conservative cross-stream error bound must include logger-anchor
bracket, affine-fit residual, DME bracket/latency, clock quantization, and
measured intra-group skew. Resampling is an analysis product; the original
records remain immutable and independently ordered.

Linux documents `CLOCK_MONOTONIC_RAW` as raw hardware-based time not subject to
frequency adjustment, while excluding suspended time. The retained man-page
source supports that clock choice. The affine fit, weighting, residual policy,
and quarter-event acceptance rule are project requirements, not Linux or ASAM
claims.

## Channel provenance, calibration, and faults

The protocol resolves all spelling variants in the 24 observations to 25
canonical channel IDs. Every channel requires raw field provenance and a
fault/status record. Explicit independent measurement channels—MAF, absolute
MAP, ambient pressure, wideband lambda, fuel pressure, post-intercooler IAT,
pre-throttle pressure, sensor voltage, torque, knock acquisition, and both-bank
EGT—also require a passing bench calibration artifact with valid range and
uncertainty before a capture can pass.

RPM, pedal, throttle, battery, coolant, injector-command, ignition, Valvetronic,
and VANOS channels require exact source/interface traceability. Their factory
or service-tool provenance is not mislabeled as independent bench calibration.
Engine/ignition state and soak time are context channels but still retain the
raw source and clock.

The related hardware remains tracked in issue
#22 (supporting artifact withheld from public edition) and sensor issues
#11 (supporting artifact withheld from public edition),
#12 (supporting artifact withheld from public edition),
#13 (supporting artifact withheld from public edition), and
#15 (supporting artifact withheld from public edition).
In plain tracker terms, capture dependencies are issue #22 and sensor issues
#11, #12, #13, and #15.

## Allowed perturbations and safety boundary

The machine protocol defines observe-only, key-on engine-off, idle/warmup,
bounded RPM, small throttle, bounded naturally aspirated load, bench pressure,
natural battery-voltage, stock service-state, and stop-on-adverse sustained-load
procedures. Each requires operator authorization, validated channels and
timebase, and predeclared abort thresholds.

The [pre-swap CAN runbook](msv70-can-baseline-capture-runbook.md) intentionally
uses a smaller first pass: key-state/bus wake, KOEO pedal input, cold start,
warm idle accessory loads, and warm stationary RPM holds. A hot restart and
one gentle road/load recording are recommended but may be skipped. Raw CAN can
discover externally broadcast candidates and timing; it cannot bind anonymous
RAM, prove the MAF/filling bridge, or select internal pressure, torque, fuel,
lambda, or table branches. Those outcomes still require synchronized DME and
independent channels.

Use a staged stock-NA capture:

1. **Transport/timebase proof:** bench or stationary warm idle; verify the
   17-state grouped snapshot, raw ordering, drop detection, and at least 20
   clock anchors before evaluating a firmware relationship.
2. **`NA-05` stationary excitation:** start with a quiet window; hold 1200,
   1800, and 2500 rpm twice; add smooth blips below 2500 rpm; finish with a
   quiet window.
3. **`NA-10` optional moderate load:** road or dyno only after `NA-05` passes;
   operate between 1500 and 3000 rpm at roughly 20-40% pedal, then include
   coastdown. No WOT.

Do not flash or reset adaptations between these stages. If the selector or gate
branch required by an observation never occurs, report `unresolved`; do not
reinterpret steady correlation as branch evidence. Fault injection stays
deferred until a clean baseline exists and the chosen protocol is documented
to support it safely.

For a selected observation subset, `state_measurements` must exactly cover the
union of its required states and match `ram_stream.state_ids`; required
channels are likewise the exact selected-observation union. This permits the
focused three-observation capture without pretending the other 21 contracts
were executed.

No boosted run is authorized. Firmware mutation, diagnostic suppression, and
actuator bypass are all false in both the protocol and capture schema. Stop on
unexpected knock, unsafe lambda, fuel/oil pressure warnings, instrument faults,
lost synchronization, or adverse EGT/component-protection behavior. Numeric
abort thresholds cannot be filled honestly until the selected instruments are
calibrated and the stock-safe procedure is approved.

## Commands

Generate to a fresh path and compare before replacing tracked bytes:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_dynamic_validation.py compile \
  --repo-root . \
  --output ./output/msv70-dynamic-validation-worksheet.NEW.jsonl

python3 scripts/msv70_dynamic_validation.py verify-worksheet \
  --repo-root .
```

Prepare a new, non-accepting manifest template:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_dynamic_validation.py prepare-capture \
  --repo-root . \
  --output ./output/issue39-capture-template.json
```

Structural validation permits a `not_run` template. Execution acceptance does
not:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python3 scripts/msv70_dynamic_validation.py validate-capture \
  --repo-root . \
  --capture analysis/validation/msv70-capture-manifest.example.json

python3 scripts/msv70_dynamic_validation.py validate-capture \
  --repo-root . \
  --capture evidence/captures/issue-39/CAPTURE/capture.json \
  --require-executed
```

All commands emit JSONL and refuse to overwrite compiler/template outputs.
Run focused and full tests through `scripts/stream_jsonl.py` (supporting artifact withheld from public edition) with fresh raw-log
paths so the complete console stream remains traceable.

## Remaining issue-#39 acceptance work

The static protocol and deterministic worksheet are ready. Issue #39 must stay
open until all of the following exist:

- a passing canonical-identity gate plus retained raw telegrams for the reviewed
  eight-ID F0 compatibility/timing probe; static decoding did not execute an
  ECU job;
- a passing read-only grouped/event-driven probe for the focused 17-state set,
  followed before the full suite by issue-#22 logger configuration artifacts
  and measured update rates for the 73 reviewed indirect/mixed-width views;
- calibrated instruments and pre-capture fault/status records;
- a measured, passing DME/logger timebase with retained alignment artifacts;
- retained baseline and controlled naturally aspirated capture files with full
  timestamps and metadata;
- capture-bound support/contradict/unresolved outcomes; and
- any confidence changes tied to exact hashes without correlation-derived
  causal edges.

Static tooling cannot substitute for those measurements.
