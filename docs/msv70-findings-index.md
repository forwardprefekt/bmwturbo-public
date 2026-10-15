# MSV70 findings index

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Target: 2006 E90 330i, N52, MSV70 `0049PP0L921S0AC1S`

This is the navigation and confidence boundary for the current MSV70
reverse-engineering work. It links to the detailed dossiers and generated
facts rather than repeating them. This synthesis changes no firmware or
calibration bytes. Nothing here is a flash recommendation.

## Latest offline closures and owner handoff

The 2026-09-05 continuation adds five source-bound reports without a new DME
pull, tests, numerical test scenarios or firmware/hardware changes:

| Report | Newly settled offline | Remaining real-world boundary |
|---|---|---|
| [MAF acquisition](../analysis/derived/review-2026-09-04-offline-maf-closure/findings.json) | Acquired versus feedback producer, availability/selector conditions, Q6/Q5 boundary and exact match to an untested public local-range proposal | Actual MAF/housing/placement characteristic and current behavior |
| [Pressure interface](../analysis/derived/review-2026-09-04-pressure-interface-closure/findings.json) | ADC descriptor, conversion, model-feedback substitute, bit-5 behavior and bounded offset/voltage diagnostics | Board/sensor electrical facts and truthful boosted interface |
| Charge temperature (supporting artifact withheld from public edition) | Diagnosed HFM IAT/coolant/model lineage, alternate factor modes and nonneutral MAF correction | Actual sensor location, heat/pressure response and independent post-IC temperature |
| [Injector equations](../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json) | Binary scales, modeled rail pressure, ADC-derived voltage, pointer families, biased offset and uniform minimum uplift | Supplier definitions, pressure-dependent latency/short-pulse data and actual operating conditions |
| [Lambda topology](../analysis/derived/review-2026-09-05-lambda-topology-closure/findings.json) | Separate bank sensor/controller histories and lag/delay maps; modeled exhaust-pressure compensation | Runner/merge/bung geometry, sensor environment and deliberate bank architecture |

The multilingual source corpus (supporting artifact withheld from public edition)
separates public proposals, commercial claims, patents and measured/physical
evidence. Owner actions (supporting artifact withheld from public edition) now begin with
hardware records, supplier definitions and fuel/layout decisions, not another
image pull. Existing runtime/capture contracts and curated graph relationships
remain unchanged; these reports do not convert static evidence into a passed
runtime observation.

## Canonical dataset and address conventions

- Full BDM read: 2,621,440 bytes, SHA-256
  `757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`.
- 9PPL921S XDF: SHA-256
  `61726421333a4dd02438b867dc665f10ba858cf449baa9c87913c05e17aa2ed4`.
- Exact generated-input hashes and loader slices:
  `analysis/ghidra/input/provenance.json` (supporting artifact withheld from public edition).
- ROM regions, checksum coverage, and public-program comparison:
  [`msv70-rom-map.md`](msv70-rom-map.md).
- Dataset ID:
  `msv70:sha256:757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`;
  structured-fact logical SHA-256
  `7fd2b355353235228a61d45ba86ce6ccce88a1e66a561f61984cbc773db90d9c`.

| Address form | Convention |
|---|---|
| XDF parameter offset | `x` in the 128 KiB calibration block |
| Full-read file offset | `0x40000 + x` |
| Runtime calibration alias | `0x840000 + x` |
| External/LAR CPU address | full-read offset in the mapped external regions; loader base `0x000000` |
| Internal-flash CPU address | `0x400000 + (file offset - 0x200000)`; loader base `0x400000` |
| Calibration small-data use | configured baseline `r2 = 0x847ff0`; retain the signed `r2` displacement |
| RAM small-data use | configured baseline `r13 = 0x7f8000`; retain the raw `r13` displacement |

Example: XDF offset `0x12528`, full-read offset `0x52528`, and runtime
address `0x852528` are the same `KF_ZW_VT98` Z component. The configured `r2`
base is also checked by the exact `c_conf_maf` relationship
`0x847ff0 - 0x365a = 0x844996`. These are versioned baseline conventions and
direct code uses, not a claim that an A2L independently named every register
state.

## Evidence authority

The pinned reference is Ghidra 12.0.4 PUBLIC build `20260303`. The disposable
workflow, processor, block layout, analyzer profile, JVM, and comparison
commands are in the Ghidra decompilation playbook (supporting artifact withheld from public edition).
The official archive SHA-256 is
`c3b458661d69e26e203d739c0c82d143cc8a4a29d9e571f099c2cf4bda62a120`.
The manifest-generated focused evidence is indexed by
`analysis/derived/focused/generation.json` (supporting artifact withheld from public edition),
which binds every artifact to the manifest, canonical input hashes, Ghidra
release, and JVM. Its address-named C files live under
`analysis/derived/focused/decompiled/` (supporting artifact withheld from public edition).

The 2026-09-03 fresh-eyes review adds an independently retained
decompilation corpus at
`analysis/derived/review-2026-09-03/` (supporting artifact withheld from public edition)
with its own provenance contract (same canonical inputs, same pinned Ghidra
release). Its review report is
`reviews/2026-09-03-fresh-eyes-eisy-sensor-path-and-boost-reading-review.md` (supporting artifact withheld from public edition).
It re-verified the tracked instruction-level claims and recovered the
conditioned issue-#69 raw-state route: `2b30 -> 2b26` when `1f62 == 0`,
then `2b26 -> 6980 -> 6ac0 -> 6ab8`, with `CW_EISY` bit 0 selecting
`6ab8 -> 6ab6`. This does not bind `2b26` to physical measured MAF or prove
runtime branch activation. The review also instruction-proved
ambient-minus-differential for absolute manifold pressure (#70), decoded the
latent `S_EISY_TURBO` doubled-Pvdk/P_std normalization and its
`K_PVDKPNRFV_MX` cap (#71), recorded the full EISY model structure (#72), and
exposed an absolute-address/pointer-table census coverage gap (#73) with a
consolidated boost-reading accuracy map (#74).

Whole-program observations are in the structured fact contract (supporting artifact withheld from public edition).
The SQLite/FTS5 guide (supporting artifact withheld from public edition) describes a disposable search
projection; the tracked CSV/JSON and focused decompilations remain authoritative.
Nearest-address XDF candidates are discovery aids, never exact symbol matches,
and shared descriptors/axes retain every alias instead of selecting one name.

Evidence labels are intentionally separate from safety: **Direct** means an
exact listing/reference/p-code, address, or component record;
**Decompiled-direct** means version/hash-bound rendered C; **Derived** means
reproducible arithmetic or unit conversion; **Semantic** means a reviewed
medium-confidence name that retains its raw ID; and **Unproven** means the
relationship must not be generated or relied upon. High confidence never
means a boosted value is safe.

## Scalable analysis pattern and lessons

The scalable path is already present: a pinned disposable Ghidra rebuild
(#25 (supporting artifact withheld from public edition)), one reviewable
focused manifest (#26 (supporting artifact withheld from public edition)),
deterministic listing/reference/p-code facts
(#27 (supporting artifact withheld from public edition)), and a disposable
relational/FTS projection (#28 (supporting artifact withheld from public edition)).
Do not add a graph server, another decompilation tree, or a C-text equivalence
normalizer. Issue #29 (supporting artifact withheld from public edition)
should project direct edges from the structured facts and keep reviewed
semantic annotations separate. Rendered-code regex matches remain labeled
supplemental discovery evidence.

The key lessons are to freeze the memory map before comparing counts, keep
XDF object/component aliases many-to-one rather than choosing a convenient
name, distinguish initializer writes from runtime producers, and preserve an
unknown boundary instead of converting narrative proximity into causality.

## Findings and confidence

| Domain | Finding | Raw identity | Evidence | Confidence | Follow-up |
|---|---|---|---|---|---|
| Provenance | The external/LAR and internal-flash programs are loaded separately and the calibration block has a runtime alias. | external `0x000000..0x17ffff`; internal `0x400000..0x47ffff`; calibration `0x840000..0x85ffff` | [`msv70-rom-map.md`](msv70-rom-map.md); `generation.json` above | High: hashes and block metadata | Baseline #25 (supporting artifact withheld from public edition) |
| Small data | The analysis contract uses `r2 = 0x847ff0` for calibration references and `r13 = 0x7f8000` for RAM-state displacements. | `r2-0x365a -> 0x844996`; `ram:r13:*` | playbook (supporting artifact withheld from public edition); fact summary (supporting artifact withheld from public edition) | High for configured bases and observed operands; semantic RAM names vary | Facts #27 (supporting artifact withheld from public edition) |
| Measured MAF | ADC logical channel `0x1b` indexes `id_maf_tab`; the unsigned 16-bit, `0.015625 kg/h` output represents at most `1023.984375 kg/h`. Direct decoding shows the monotonic stock table first reaches 65535 at index 233 and remains saturated through 255. The raw `2b26` state lies on a conditioned path to adjusted load, but its physical measured-MAF identity remains runtime-contingent. | `FUN_00432fc8`; `FUN_00435048`; `r13+0x2b26`; runtime `0x849ca4`; file `0x49ca4` | [airflow dossier](msv70-maf-to-load-decompilation.md); focused C (supporting artifact withheld from public edition); canonical binary (supporting artifact withheld from public edition) | High for instructions, decoded cells, scale, and conditioned raw dataflow; unproven physical identity/runtime branch | Runtime protocol #39 (supporting artifact withheld from public edition); conditioned path #69 (supporting artifact withheld from public edition) |
| Airflow selection | `2ac2 -> 2b30` requires `1fea == 0` and `1e8c == 0`; `2b26 -> 2b30` requires `1fea == 0` and `1e8c != 0`; a blocked update retains the prior value. When `1f62 == 0`, `2b30 -> 2b26`, creating a conditioned raw-state cycle rather than proving either candidate's physical identity. | `FUN_00435180`; `FUN_00435048`; `r13+0x2ac2`; `r13+0x2b26`; `r13+0x2b30`; gates `r13+0x1fea`, `r13+0x1e8c`, `r13+0x1f62` | [consumer/boundary dossier](msv70-airflow-filling-boundary.md); focused selector (supporting artifact withheld from public edition); conditioned copy (supporting artifact withheld from public edition) | High for conditioned branch/copy; unknown for physical names and active runtime producer | Runtime protocol #39 (supporting artifact withheld from public edition) |
| Conditioned airflow routes | `2b30 -> 2b26 -> 6980 -> 6ac0 -> 6ab8 -> 6ab6` requires `1f62 == 0` and final-load `CW_EISY` bit 0 set. A separate recovered route continues `6ac0 -> 6c6e -> 6c6a -> 6ac2 -> 6b9a` with bit 1 set and bit 2 clear. Canonical `CW_EISY=0` leaves the airflow-integrator input inactive; its modeled-pressure ceiling remains. | `r13+0x2b30`; `r13+0x2b26`; `r13+0x6980`; `r13+0x6ac0`; `r13+0x6c6e`; `r13+0x6c6a`; `r13+0x6ac2`; `r13+0x6b9a`; `cal:xdf:1e000:value` | [airflow dossier](msv70-maf-to-load-decompilation.md); [source-bound continuation](../analysis/derived/review-2026-09-04-load-continuation/findings.json) | High for conditioned software dataflow; physical MAF identity, runtime activation and boosted validity remain unproven | Runtime protocol #39 (supporting artifact withheld from public edition); graph/context pack #82 (supporting artifact withheld from public edition) |
| MAF range propagation (#86) | Acquisition converts table Q6 to downstream Q5, whose u16 range is 2047.96875 kg/h. Direct measured load normalizes to signed relative load `6ab8`, then mixed-access shared `6ab6`; unsigned shared charge `2ada` reaches `ffff` at load raw 21485. The optional bit-1 integrator separately clamps summed Q5 inflow `6c72` at 1023.96875 kg/h. | `00435048`; `00442f7c`; `00434c98`; `00447004`; `6ac0`, `6ab6`, `2ada`, `6c72` | [typed trace, range experiment and consumer matrix](msv70-maf-to-load-decompilation.md#issue-86-range-propagation-conclusion); hash-bound evidence (supporting artifact withheld from public edition) | High for instructions and integer arithmetic; physical units are source-consistent, runtime unobserved | #86 (supporting artifact withheld from public edition); no global-rescale or flash approval |
| Filling selection | Canonical `S_EVNN_AUS=0` selects `6b9a` into `6b6a`; nonzero selects `6b5a`. `6b6e = sat_u16((6b6a << 10) / 0x0ccd)` is a scaled output, not a selector. The mismatched A2L's 38-anchor `EISY_EV` projection supplies candidate names: `6b9a` neural-network filling (`Rfv_ist_nn`), `6b5a` lift fallback (`Rfv_ist_hub`), and `6b6a` selected modeled-unthrottled relative filling (`Rfv_ist`). | `FUN_004439b4`; `S_EVNN_AUS` at `0x85e00b`; `r13+0x6b5a`; `r13+0x6b9a`; `r13+0x6b6a`; `r13+0x6b6e` | [airflow dossier](msv70-maf-to-load-decompilation.md); [MAF analysis contract](msv70-measured-maf-envelope.md); focused C (supporting artifact withheld from public edition) | High for source-bound active selection/arithmetic; Medium for A2L-informed names; alternate calibration remains untested | Capture the active branch without changing firmware; runtime protocol #39 (supporting artifact withheld from public edition) |
| Pressure acquisition and selector | Stock modeled `Pvdk` is ambient-derived `2b38 -> 698c -> 63c6`; its filtered value bounds integrated pressure `6c6a`, selected into `6ac2` by canonical `CW_EISY` bit 2 clear. Actual NN filling uses `max(6ac2-6b76,0)` with coefficient/temperature factors. The differential conversion cannot represent a negative differential. `S_EISY_TURBO` changes request normalizer `67b0`, whose cap is not a cap on density ratio `01802fc8`. | `FUN_004353d0`; `FUN_001038e4`; `FUN_00447004`; `FUN_004439b4`; `FUN_00434c48` | [pressure dossier](msv70-pressure-ps-pvdk-dossier.md); [fresh findings](../analysis/derived/review-2026-09-04-pressure-continuation/findings.json) | High for executable equations and canonical bytes; physical pressure truth/model accuracy remain unverified | Truthful pressure-interface and baseline capture; runtime protocol #39 (supporting artifact withheld from public edition) |
| Factory turbo comparison | Four stock N54/MSD80 references expose separate pre-/post-throttle pressure architecture, a 0.2--2.5 bar absolute transfer, pressure-ratio boost control, and fuel/spark/torque load axes reaching 220/180/220%. BMW WDS also places N54 manifold pressure on X60005 pins 31/32/33 and charge pressure/temperature on 9/10/11/17; MSV70 shares the former assignments and marks the latter unused. This supports a factory-derived wiring candidate but does not prove MSV70 PCB population, bind anonymous states, or authorize direct table copying. | `I8A0S`; `IJE0S`; `IKM0S`; `INA0S`; X60005 | MSD80 comparison (supporting artifact withheld from public edition); [pressure wiring](msv70-msd80-pressure-sensor-wiring.md); corpus manifest (supporting artifact withheld from public edition); decoded values (supporting artifact withheld from public edition) | High for OEM topology, connector assignments, hashes, bytes, and decoded XDF-defined domains; unproven for MSV70 empty-pin circuitry, community names, and cross-DME function equivalence | Bench-compare X60005 input population, then perform code-level pressure/filling alignment |
| Adjusted load | Canonical `CW_EISY=0`, `S_EVNN_AUS=0`, and `S_RFST_ON=0` select `6b9a -> 6b6a -> 6abe -> 6ade`, then lower-bounded `s16(6ade)-s16(6b62) -> 6ab6`. The zero start-enable byte disables both the startup-map and transition-filter alternatives. Bit 0 set instead selects `6ab8 -> 6ab6`; neither topology requires a MAF-to-NN bridge. | `FUN_00442f7c`; `cal:xdf:1e000:value`; `cal:xdf:1e005:value`; `ram:r13:6ab6` | [airflow dossier](msv70-maf-to-load-decompilation.md); [source-byte and instruction proof](../analysis/derived/review-2026-09-04-load-continuation/findings.json) | High for source-bound branch/dataflow; physical identity and runtime behavior require capture | Canonical modeled baseline versus separately reviewed measured-MAF alternative |
| Cylinder mass | Saturating rounded multiply/divide combines adjusted load and `c_maf_ref`; `c_maf_ref` decodes to `646.511085 mg/stk` at the 100% reference. | `FUN_00434c98 -> FUN_0040b46c`; `r13+0x6ab6`; `r13+0x2ada`; `c_maf_ref` | [airflow dossier](msv70-maf-to-load-decompilation.md); focused C (supporting artifact withheld from public edition) | High for formula/constants; physical state still needs RAM validation | Injection dossier #36 (supporting artifact withheld from public edition); runtime protocol #39 (supporting artifact withheld from public edition) |
| Injector main pulse | A bounded path combines cylinder mass, bank lambda/enrichment factors, requested mass, standard-flow/wall-film, injector-pressure, and voltage/dead-time corrections, then applies minimum/manual branches into six main-pulse calculation outputs and converts/dispatches them as per-cylinder timer commands. `0x3238` is only a maximum preliminary value. | `FUN_00457f14`; `FUN_00457f68`; `FUN_00459738`; `FUN_0043cc84`; `FUN_0043cce0`; `FUN_0043d02c`; `FUN_0043d760`; `FUN_004271c8`; `r13+0x30f6..0x3100`; timer counts `r13+0x4d54..0x4ee4` | [injector dossier](msv70-injector-pulse-path.md); classifications (supporting artifact withheld from public edition) | High for reviewed software dataflow; explicit unknown for maximum pulse, physical duty, runtime-active pointer tables, and hardware result | Runtime protocol #39 (supporting artifact withheld from public edition) |
| Torque authority | Functional driver/cruise selection conditionally participates in a multi-input request arbitration; MDIOP model and MDMAX ceiling remain distinct. Independent monitor levels produce their own difference/threshold latches, while separate loop-carried limits can conditionally influence the bounded throttle-controller request. | `r13+0x2f02 -> 0x2f04 -> 0x2d14 -> 0x6258 -> 0x66d6`; `r13+0x2180/0x218c -> 0x5790/0x5791 -> 0x2182`; `r13+0x2f30/0x2f32`; `r13+0x2e2a` | [torque dossier](msv70-torque-authority-dossier.md); classifications (supporting artifact withheld from public edition) | High for reviewed conditional software paths; explicit unknown for monitor-fault causality, runtime selectors, physical actuation, measured torque, and boosted values | Runtime protocol #39 (supporting artifact withheld from public edition); protection #38 (supporting artifact withheld from public edition) |
| Knock, lambda, and component protection | Threshold bytes feed conditional per-cylinder feedback and dispatch. Canonical initialization resolves six channel parameter objects and CPU-side service requests; acquisition-word writer `0041c3f0` is recovered. The scalar `6356` join remains separate, not a prerequisite for this path. Component-protection states still reach the bank lambda combiner. | threshold `2049..204e`; acquisition `3bf2`; dispatch `1feb..1ff0 -> 00704102+16*i`; service `0070401a`; separate scalar `6356` | [protection dossier](msv70-knock-lambda-protection-dossier.md); [dispatch findings](../analysis/derived/review-2026-09-04-dispatch-continuation/findings.json) | High for canonical setup and CPU-side publication; physical acquisition, timer program, delivered spark/lambda and diagnostic consequences remain unverified | Runtime protocol #39 (supporting artifact withheld from public edition); retain protection authority |
| Dynamic validation | The 24-observation worksheet retains 242 RAM states, 2,344 direct RAM-access records, five globals with 43 accesses, 25 channels, and 73 reviewed indirect/mixed-width views. The focused first pass is three issue-#34 observations, 17 direct states and seven external channels. It now validates canonical modeled-load normalization/correction instead of an obsolete missing-bridge hypothesis; recovered pressure candidates and knock acquisition/dispatch remain planned, not measured. | `analysis/validation/msv70-dynamic-validation-{protocol.json,worksheet.jsonl}` (supporting artifact withheld from public edition); focused `ram:r13:{1e8c,1f62,1fea,2ac2,2ada,2b26,2b30,6984,6ab6,6abe,6ade,6b5a,6b62,6b68,6b6a,6b6e,6b9a}` | [protocol and implementation gate](msv70-dynamic-validation-protocol.md); host-input summary (supporting artifact withheld from public edition); worksheet (supporting artifact withheld from public edition) | High for address/width and static branch contracts; **not run** for project-car transport, timing, and captures | Prove canonical identity, grouped/event timing and affine synchronization before evaluating the active source-bound branch |
| Consumers | Adjusted load indexes `KF_ZW_*` and `KF_MDIOP_*`; converted cylinder mass indexes `ip_lamb_bas_*`. | `FUN_000b0424`; `FUN_000bca44`; `FUN_0046e3e8`; `r13+0x6ab6`; `r13+0x2ada` | [load trace](msv70-boost-load-downstream-trace.md); focused C for spark (supporting artifact withheld from public edition), torque (supporting artifact withheld from public edition), and lambda (supporting artifact withheld from public edition) | High for call arguments and table addresses; active selector states need runtime proof | Injection #36 (supporting artifact withheld from public edition); torque #37 (supporting artifact withheld from public edition); protection #38 (supporting artifact withheld from public edition) |
| Table layout | The three traced consumers reference 25 constant-address tables across 38 call sites; dimensions, widths, offsets, and shared axes are recorded. | runtime descriptors and XDF `x/y/z` offsets | [table layout](msv70-interpolated-table-layout.md); calibration-component CSV | High for constant call sites/layout; aliases are preserved | Manifest #26 (supporting artifact withheld from public edition) |
| Interpolation | Recovered helpers hold their endpoint cells outside the breakpoint range; they do not extrapolate. | `FUN_00408ab4`; `FUN_00414978`; `FUN_00415070` | [airflow dossier](msv70-maf-to-load-decompilation.md); focused helpers 00408ab4 (supporting artifact withheld from public edition), 00414978 (supporting artifact withheld from public edition), and 00415070 (supporting artifact withheld from public edition) | High for inspected helper branches | Graph/query work #29 (supporting artifact withheld from public edition) |

## Limits are not interchangeable

| Limit class | Current evidence | Consequence |
|---|---|---|
| Datatype / representation | `id_maf_tab` is Q6 u16; averaging converts to Q5 u16, not a global Q6 range. Shared `6ab6` is mixed-access relative load; `2ada` is unsigned cylinder mass. | A local table/divisor redesign differs from globally changing airflow units. Neither recovers already-clipped measurements or authorizes calibration changes. |
| Arithmetic | Direct measured normalization caps at 327.67% before correction; shared charge reaches `ffff` at load raw 21485. Optional integrator `6c72` has a separate signed-Q5 1023.96875 kg/h summed-inflow cap. | Load, mass and airflow ceilings are different. The optional integrator cap is not on the direct bit-0 measured-load route; `ffff` charge is not a proven usable value. |
| Calibration axis | pressure corrections end near 900--924 hPa, primary spark at 108.36%, MDIOP at 115%, base lambda at 500/650 mg/stk | Endpoint behavior is helper-specific; raising one clamp leaves other domains pinned. |
| Sensor / electrical | The stock manifold input uses unsigned differential conversion followed by recovered ambient subtraction; physical transfer, electrical fault bounds and boosted validity remain unmeasured | A generic absolute MAP transfer-table swap is unsupported. |
| Diagnostic / plausibility | Pressure enable gates, thresholds, adaptation candidates, and diagnostic state machines are focused, but final DTC/fallback/protection consequences remain explicit unknowns | Disabling declarations or monitors is not a substitute for extending their valid domain. |
| Physical safety | Boosted cell values, injector margin, rail-pressure behavior, and protection response are not established | Bench and controlled engine validation with independent instruments remain mandatory. |
| Injector pulse / duty | Minimum-time and manual-override branches are recovered; no maximum main-pulse calibration or safe physical duty ceiling is proven | Representation width, phase scheduling, table axes, and arithmetic saturation must not be substituted for measured injector/driver margin. |
| Torque authority / monitoring | MDIOP endpoints, MDMAX, request arbitration, monitor thresholds, software throttle bounds, and the VANOS eligibility threshold are distinct mechanisms | Do not raise model tables while bypassing independent monitors, infer monitor-fault causality, or treat a bounded request as measured actuator position or delivered torque. |
| Knock / lambda / component protection | Knock-map axes, raw arithmetic clamps, modeled temperature/lambda targets, diagnostic thresholds, and physical detonation/thermal limits are separate classes | Keep stock safety authority intact; static table endpoints and modeled targets do not establish sensor electrical limits, delivered lambda/spark, or physical safety margin. |

## Runtime validation boundary

The focused first capture groups `r13+0x1e8c`, `0x1f62`, `0x1fea`, `0x2ac2`,
`0x2ada`, `0x2b26`, `0x2b30`, `0x6984`, `0x6ab6`, `0x6abe`, `0x6ade`,
`0x6b5a`, `0x6b62`, `0x6b68`, `0x6b6a`, `0x6b6e`, and `0x6b9a`.
The denominator `6b68`, intermediate `6ade`, and correction `6b62` are required
to evaluate the canonical modeled-load observation. Synchronize independent
MAF, absolute MAP, ambient pressure, post-intercooler IAT, RPM, throttle, and
wideband lambda.
Pedal, raw stock-MAF voltage, coolant, battery, Valvetronic/VANOS, and raw CAN
are recommended context.

Before the `NA-05` run, prove an event-driven or grouped stock-DME transport and
the host-bracketed affine timebase described in the
[dynamic protocol](msv70-dynamic-validation-protocol.md). A controlled
perturbation can bind a physical interpretation and identify the active
branches; correlation alone does not prove physical MAF provenance or runtime
activation. The conditioned `2b30 -> ... -> 6b9a` software route is already
recovered with `1f62 == 0`, `CW_EISY` bit 1 set, and bit 2 clear. Canonical
`CW_EISY=0` disables that optional airflow input, and its modeled-pressure
ceiling remains. A complete alternate derivation into `6b5a` is not asserted.

For torque-authority validation, also capture `r13+0x2f02`, `0x2f04`,
`0x2d14`, `0x6258`, `0x66d6`, monitor states `0x2180`, `0x218c`, `0x5790`,
`0x5791`, `0x2182`, intervention limits `0x2f30` / `0x2f32`, bounded
throttle request `0x2e2a`, Valvetronic setpoint `0x694c`, and the
`0x65ac` / `0x6e14` VANOS gate. Keep stock monitoring enabled and synchronize
pedal, measured throttle, Valvetronic lift, intake/exhaust VANOS position, and
an independent torque measurement. Correlation cannot select a branch or prove
monitor-fault causality.

For protection validation, synchronize actual threshold bytes `r13+0x2049..0x204e`
and scheduler `0x2023`, keeping window halfwords `2d54..2d6a` separate,
ignition result `0x6356`, lambda/protection staging `0x2b5a..0x2b60`, full-load
target `0x2b9e`, final raw outputs `0x2ba8/0x2baa/0x2bac`, and the exact
selector states declared in the protection domain. Retain all diagnostic and
fallback behavior. Use independent knock acquisition, commanded ignition
timing, wideband lambda, both-bank EGT, absolute MAP, post-intercooler IAT, and
RPM; correlation cannot establish a missing static join or physical safety.

## Work ownership

- Parent program and dossier/dynamic definition of done: #24 (supporting artifact withheld from public edition).
- Frozen reproducible baseline: #25 (supporting artifact withheld from public edition).
- Manifest-driven focused evidence: #26 (supporting artifact withheld from public edition).
- Structured whole-program facts: #27 (supporting artifact withheld from public edition).
- Disposable SQLite/FTS5 search index: #28 (supporting artifact withheld from public edition).
- Bounded relationship graph; preserve conditional routes and runtime/physical evidence gaps: #29 (supporting artifact withheld from public edition).
- This synthesis and PR #23 merge gate: #30 (supporting artifact withheld from public edition).
- Selected-airflow consumers and conditioned filling routes: #34 (supporting artifact withheld from public edition).
- `Ps`/`Pvdk`, pressure plausibility, diagnostics, and protection: #35 (supporting artifact withheld from public edition).
- Cylinder mass through final injector pulse width: #36 (supporting artifact withheld from public edition).
- Torque request, monitors, and interventions: #37 (supporting artifact withheld from public edition).
- Knock, lambda, component protection, and diagnostic fail-safes: #38 (supporting artifact withheld from public edition).
- Synchronized RAM and independent-instrumentation protocol: #39 (supporting artifact withheld from public edition).
- Fresh-eyes review: measured-MAF to load bridge and `CW_EISY` bit 0 producer selection: #69 (supporting artifact withheld from public edition).
- Fresh-eyes review: ambient-minus-differential `Ps` proof and acquisition chains: #70 (supporting artifact withheld from public edition).
- Fresh-eyes review: latent turbo normalization topology and Pvdk/P_std cap: #71 (supporting artifact withheld from public edition).
- Fresh-eyes review: EISY model structure reference and dossier updates: #72 (supporting artifact withheld from public edition).
- Fact coverage: absolute-address and pointer-table census union: #73 (supporting artifact withheld from public edition).
- Boost-reading accuracy sensor/lookup extension map: #74 (supporting artifact withheld from public edition).

The static dossiers and validation compiler are present. Actual dynamic
validation remains owned follow-up work and is not a reason to invent facts in
this index or broaden correlation into causality.
