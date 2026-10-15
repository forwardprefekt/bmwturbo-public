# Ignition, lambda, and protection

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page summarizes the canonical MSV70 software paths for spark selection, knock processing, lambda targeting and feedback, modeled component protection, and diagnostics. These are distinct software lanes, not a proven single safety chain. Static code does not establish delivered spark or lambda, physical knock/temperature, sensor adequacy, or safe behavior on modified hardware.

## Evidence boundary

The primary source is the [knock, lambda, and component-protection dossier](../msv70-knock-lambda-protection-dossier.md), including its [single-turbo lambda topology closure](../../analysis/derived/review-2026-09-05-lambda-topology-closure/findings.json), knock continuation (supporting artifact withheld from public edition), and [dispatch/acquisition continuation](../../analysis/derived/review-2026-09-04-dispatch-continuation/findings.json). These sources document instruction-reviewed software behavior and calibration intent. Actual running branches, physical sensor signals, actuator transfer, and engine response require separate observation. See [evidence rules](evidence-and-tools.md).

## Keep the lanes separate

The stock review identifies separate spark-selection, knock-processing, lambda-target/feedback, modeled component-protection, and diagnostic/fail-safe paths. They share operating context and may share data, but the reviewed evidence does not justify flattening them into an invented cause-and-effect protection chain. A threshold, calibration endpoint, modeled target, software dispatch or diagnostic report is not proof of physical safety or a specific downstream intervention. See the [dossier's five software lanes](../msv70-knock-lambda-protection-dossier.md#five-separate-software-lanes) and [limits distinction](../msv70-knock-lambda-protection-dossier.md#limits-are-not-interchangeable).

## Ignition selection

`func:000b0424` consumes adjusted-load state `ram:r13:6ab6` and separate normal, safe, and full-load calibration families, including runtime descriptors `0x8524de`, `0x852668`, `0x852b68`, and `0x852bea`. Its selector/blending logic publishes a signed raw software result in `ram:r13:6356`. `func:0046d15c` reads that state and converts it into a downstream raw software value. These are software values and consumers, not proof of coil actuation, final delivered spark angle, or a universal knock-response connection. Active old/new-mode populations and physical consequences are unobserved. See the [ignition lane](../msv70-knock-lambda-protection-dossier.md#1-ignition-selection) and the [load reference](airflow-and-load.md).

The per-cylinder knock processing described below has a separately recovered software dispatch route, but its exact join to scalar result `6356` is unresolved. Do not equate a recovered dispatch route with a proven modification of commanded ignition timing or physical retard. The dossier explicitly records `software_dispatch_recovered_6356_join_unresolved`; see its [knock continuation](../msv70-knock-lambda-protection-dossier.md#fresh-consumer-continuation-2026-09-04).

## Knock: windows, thresholds, and dispatch

`func:00439ef8` performs two different computations. Measurement-window scheduling uses selector `21d2` to choose beginning and ending calibration families for six cylinders. The starts are 16-bit words `2d54..2d5e`, ends `2d60..2d6a`; when an end is not greater than its beginning, the selected pair becomes raw `0x20/0x90`. These window words are not threshold-factor outputs.

Threshold processing uses a separate selector (`2023`, copied from `21d5`) to choose among six `ip_fac_thd_knk_*` families. It uses shared airflow/RPM storage and distinct per-family calibration surfaces. The resulting threshold values are bytes at `r13+0x2049+cylinder` (`007fa049..007fa04e`), not the window words. The selected filtered word, `c_nl_min`, adjustment curve, optional `c_fac_tra_knk` multiplication, and `c_knk_thd_max` contribute to the result. Canonical raw constants include 10, 33, and 225; they are software constants, not physical knock limits. Runtime selector/cylinder populations and behavior outside reviewed selector range 0..5 are not established. See the dossier's [knock processing](../msv70-knock-lambda-protection-dossier.md#2-knock-processing) and [fresh consumer continuation](../msv70-knock-lambda-protection-dossier.md#fresh-consumer-continuation-2026-09-04).

The continuation reviews a conditional comparison against a previously stored selected threshold, gated by operating states and countdown. It updates per-cylinder words, means, output-byte arrays and downstream corrections on distinct paths; disabled/countdown handling differs, and unchanged summaries do not imply no per-cylinder work. Overrides and transitions can select other sources. This is software behavior, not an unconditional threshold response. See the [fresh consumer continuation](../msv70-knock-lambda-protection-dossier.md#fresh-consumer-continuation-2026-09-04).

A separate continuation resolves canonical software parameter publication: descriptor `004049c4` enumerates six channels, initializes pointer targets at `00704100+0x10*i`, and the reviewed indirect halfword destinations are `00704102` through `00704152`. Service requests select `0070401a`; acquisition words come through a descriptor-selected block (`00704800`/`00704c00`), not that dispatch table. This resolves software addresses and dispatch, not silicon-register meaning, timer program, pins, drivers, coils, or delivered ignition. Do not add MMIO reads to a RAM logger based on address familiarity. See the [dispatch/acquisition evidence](../../analysis/derived/review-2026-09-04-dispatch-continuation/findings.json).

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `func:00439014`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

Each invocation processes three indices starting at byte `21da`, wrapping at six. Record bit 7 selects the callee; both branches convert the unsigned `1feb+cylinder` byte to `byte*16-0x5f0`, then pass a sign-extended 16-bit result. This recovers a software dispatch choice, not the unresolved join to `6356` or physical ignition timing.

## Lambda targets and single-turbo topology

`func:0046e3e8` uses raw cylinder-mass coordinate `2ada` and six base-lambda map families, publishing intermediate states including `2b96`, `2b98`, `2b9a`, and `2b9c`. `func:0046e7b0` chooses a fixed fallback or curve-derived full-load software target and writes `2b9e`. `func:0046ea20` combines these inputs with protection staging and publishes summary outputs `2ba8`, `2baa`, and `2bac`. The maps, clamps and stores are reviewed software behavior; active runtime populations, delivered mixture, fuel capacity/quality and suitable target mixture are unknown. See the dossier's [lambda target selection](../msv70-knock-lambda-protection-dossier.md#3-lambda-target-selection) and [fuel and injection](fuel-and-injection.md).

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `func:0046e7b0`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

Either zero gate byte (`1fe2` or `1fd0`) selects the fixed raw value `0x5800`; only two nonzero gates reach the curve helpers. Both routes publish `2b9e`. The raw fallback is not a measured mixture or a recommended lambda target.

The offline [lambda topology closure](../../analysis/derived/review-2026-09-05-lambda-topology-closure/findings.json) recovers separate sensor records with `0x2d8` stride and controller records with `0x14c` stride. Bank paths `4438/4710` feed controller histories, then `41e6/4332`, copied on the injector request path to `3dda/3f2a`. Pointer `00407410` selects distinct normal/aged sensor-lag and exhaust-delay maps for each bank. This is a static software topology, not proof that the vehicle's physical sensors or exhaust preserve two-bank observability. See [single-turbo lambda topology](../msv70-knock-lambda-protection-dossier.md#single-turbo-lambda-topology-offline-closure) and the [injector path](../msv70-injector-pulse-path.md#reviewed-main-pulse-path).

Both modeled sensor conversions use common modeled exhaust pressure `2b34`, formed from `ip_prs_ex_inc(2ac6) + 2ccc`; it is not a turbine-inlet pressure measurement. Its approximate 600–1600 hPa correction domain is not a sensor rating or validated pre-turbine operating range. Two probes after full exhaust mixing cannot recreate independently observable three-cylinder-bank feedback. Bank-preserving samples must be representative and taken before the first mixing junction; two bungs or an externally divided-looking flange do not prove internal separation. Pre-turbine sampling needs part-specific pressure, temperature, pulsation and transport characterization. See the [topology closure](../../analysis/derived/review-2026-09-05-lambda-topology-closure/findings.json) and [dossier topology findings](../msv70-knock-lambda-protection-dossier.md#single-turbo-lambda-topology-offline-closure).

The dossier reports that 22RPD advertises a single-bank code patch with one post-turbo sensor, without disclosing AC1S implementation, revision support, or diagnostic/adaptation handling. ESS's N52 header warning identifies feedback timing as an application dependency. These are vendor statements, not proof of compatibility or permission to parallel sensors, disable a bank, or remove monitoring. Physical runner/bank/first-merge/bung topology and exact sensor identity remain unresolved; an exterior manifold photo does not show internal separation. See [topology findings](../msv70-knock-lambda-protection-dossier.md#single-turbo-lambda-topology-offline-closure).

## Modeled component protection

`func:0010fcd4` uses `KF_TA_VKSOLL`, lambda limit `KF_BS_MX`, policy selector `CW_ATMOD`, calibration offsets and diagnostic service states to publish globals `01802fdc` and `01802fde`. The term “temperature” reflects calibration intent; the reviewed path is modeled/target calculation, not proof of a physical EGT sensor. Those globals are shifted into `7478/747a`, conditionally staged (or replaced by fixed alternatives) into `2b5a`, `2b5c`, `2b5e`, and `2b60`, and consumed with full-load input by `0046ea20` before lambda-related summaries are published. This connected software handoff does not demonstrate physical temperature, delivered lambda, thermal durability, or live activation. See the dossier's [modeled component protection](../msv70-knock-lambda-protection-dossier.md#4-modeled-component-protection).

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `func:0046ea20`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

The archive interprets the selected staged inputs as unsigned 16-bit values: the first bank reads `2b5a/2b5e`, the second `2b5c/2b60`. The later comparison block (supporting artifact withheld from public edition) compares full-load state `2b9e` and the selected `2b5e`/`2b60` input after shifting each left four bits. This is evidence of bank-specific staged inputs entering lambda arithmetic, not evidence that a physical temperature threshold activated protection.

## Diagnostics and fail-safe policy

`func:0043acf0` gates knock-sensor diagnostic behavior from raw operating states and calibration thresholds, then issues service-report calls for raw results `0x29` and `0x2a`. The reviewed report mechanics do not identify final DTC ownership or establish downstream ignition, lambda, torque, fueling, throttle, or shutdown effects. Pressure, airflow/load, fueling and torque retain their own monitor/fallback boundaries; no universal cross-domain diagnostic consequence is inferred. Safeguard policy remains `required_not_disable` in the dossier. See [diagnostic and fail-safe policy](../msv70-knock-lambda-protection-dossier.md#5-diagnostic-and-fail-safe-policy) and [torque and controls](torque-and-controls.md).

## Limits and validation boundary

Keep calibration axes, helper behavior, operating limits, arithmetic/storage saturation, diagnostics/debounce, modeled targets, sensor/electrical limits and physical safety as separate categories. A map maximum, integer clamp, diagnostic threshold, or modeled target is not a physical safety limit. The dossier's issue-#39 rows for knock feedback/dispatch, ignition selectors, lambda selection, component-protection thermal behavior and diagnostic consequence remain `execution_status:not_run`. Suitable synchronized validation requires raw state plus independent measurements appropriate to the claim; correlation cannot create a static dataflow edge, prove causality, or authorize calibration changes. See [limits are not interchangeable](../msv70-knock-lambda-protection-dossier.md#limits-are-not-interchangeable) and [issue #39 validation transfer](../msv70-knock-lambda-protection-dossier.md#issue-39-validation-transfer).

## Current unknowns

- Live spark selector/mode populations, actual delivered timing, coil-channel mapping and physical actuator transfer.
- Knock sensor conditioning/units, runtime threshold/window populations, actual knock response, exact join to scalar ignition result, and physical dispatch/register semantics.
- Sensor identity, bank sampling positions, internal exhaust separation, feedback timing, actual wideband/lambda delivery, and valid boosted sensor compensation.
- Whether modeled component-protection paths activate under operating conditions, any physical EGT/thermal state, and actual durability effect.
- Diagnostic ownership and downstream fallback/intervention behavior for the reviewed knock results.
- Suitable physical operating limits and safety margins for any modified hardware or calibration.

## Related pages

- [Airflow and load](airflow-and-load.md), [pressure model](pressure-model.md), and [fuel and injection](fuel-and-injection.md).
- [Torque and controls](torque-and-controls.md), [calibration and limits](calibration-and-limits.md), and [logging and validation](logging-and-validation.md).
- [Evidence rules](evidence-and-tools.md), [hardware and recovery](hardware-and-recovery.md), and [open questions](open-questions.md).
