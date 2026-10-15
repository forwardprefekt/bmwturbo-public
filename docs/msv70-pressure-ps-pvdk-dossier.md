# MSV70 Ps/Pvdk pressure-domain dossier

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Scope and evidence contract

This issue-#35 dossier is a manifest-bound static review of the declared
pressure inventory. It separates direct instructions, reviewed decompiler
control/dataflow, exact XDF storage, and unresolved engineering semantics. The
canonical input and Ghidra 12.0.4 baseline are inherited from the generation
metadata in `analysis/derived/focused/generation.json` (supporting artifact withheld from public edition).
The reviewed manifest SHA-256 is
`1475903d8b481c3453bd7a96f1f1b4122e20dd23c70edb8253011251957cddaf`.

The machine-readable judgments are in
`msv70-pressure-domain.jsonl` (supporting artifact withheld from public edition).
The inventory covers 35 exact function targets, 115 exact XDF components, direct
raw-state anchors for the acquisition/selector/diagnostic spine, eight distinct
limit classes, and four policy classes. It does not claim that name searches
found every pressure behavior in the firmware.

**No sensor purchase or firmware patch** is approved by this dossier. Static
firmware evidence cannot establish connector impedance, analog protection,
fault-voltage bounds, a safe pressure range, or safe boosted operation.

## Pressure stages that must remain separate

### Stock differential electrical acquisition

X60005 pins 31/32/33 are documented as supply, ground, and signal for the rear
manifold differential-pressure sensor. That connector description does not
prove the input-stage impedance, clamping, diagnostic voltage window, or a safe
replacement sensor. Those electrical properties remain outside the firmware
trace and explicitly unknown.

### Converted differential pressure

The exact XDF object `ip_map_dip_mes_bas__v_map` uses
`cal:xdf:0110c:x` for two voltage points (about 0.737 and 4.502 V) and
`cal:xdf:037d8:z` for two decoded differential-pressure points (about 0 and
800 hPa). These are configured transfer points, not an absolute MAP transfer
and not proof of electrical headroom.

Two manifest-bound acquisition candidates use the descriptor/value pair:

- `func:0013a260` selects raw accumulator/input pairs, writes raw intermediate
  `ram:r13:2b40`, invokes the transfer helpers, then writes `ram:r13:2b3c` and
  `ram:r13:2b3e` (focused C (supporting artifact withheld from public edition)).
- `func:004576f0` alternates the same raw pairs, updates accumulators, and either
  applies the two-point conversion or an alternate calculation/substitution branch before writing
  the same raw outputs (focused C (supporting artifact withheld from public edition)).

The direct RAM accesses and widths are proven. The engineering alias, physical
scale at every raw access, and exact scheduling frequency are not.

The continuation below identifies raw reconstructed, integrated, selected, and
modeled pre-throttle pressure candidates and their software signs/conditions.
Exact physical aliases, calibrated sensor transfer, electrical fault bounds,
update timing, and boosted validity remain unproven; raw identity is not a
physical measurement.

### Ambient-minus-differential conversion

The contiguous raw path is now instruction-proven:
`2b3c/2b3e -> 694e/6950`, then `6c48 = sat_u16(01802da6 - s16(694e))`
on canonical `CW_EISY` bit 5 clear. `00434c48` uses **LHZ**, not LHA, before
the positive `257/121` scaling and signed saturation in `0040b38c`.
Consequently the recovered conversion cannot produce a negative differential
value. Encoding a negative two's-complement input or inserting an absolute
pressure in these differential slots does not yield truthful boosted `Ps`.
An interface/domain change is required; a two-point transfer edit is not enough.

### Internal absolute `Ps` state

The sensor-reconstructed pressure candidate is `6c48`; the integrated
pressure candidate is `6c6a`. `004439b4` selects them into `6ac2` using
`CW_EISY` bit 2: the canonical zero byte chooses the integrated candidate.
`00447004` bounds that integrator with `63c6 - K_DPS_INTMN` (`51` raw)
and applies `K_PSMIN` (`2560` raw). The upper-bound source is filtered modeled
pre-throttle pressure, not a boost-capable measurement.

`004353d0` produces `2b38` as ambient `2ce8` minus an unsigned air-cleaner
decrement when `1fea == 0`, otherwise retaining it. Canonical decrement cells
are all zero. `000df4e8` converts it into `698c`; `000b2534` filters it into
`63c6`. Thus the native model has pressure-aware mathematics but not truthful
boosted pressure inputs. Transient filter lag is not new pressure authority.

These are executable raw-state identities. Physical transfers, accuracy,
electrical ranges and complete scheduling still require source-bound capture.

| Quantity | Static result | Remaining boundary |
|---|---|---|
| Sensor voltage/differential transfer | Exact XDF points and focused conversion calls | Electrical limits and fault voltage window unknown |
| Converted differential candidate | Raw writes `2b3c`/`2b3e` in two focused functions | Alias, per-access units/sign, cadence not fully proven |
| Ambient pressure | `2ce8`, converted into `01802da6`; ratio copied into `01802d96` | Physical calibration, update timing and fault behavior require capture |
| `Ps` candidates | Sensor-reconstructed `6c48`, integrated `6c6a`; selected `6ac2` | Neither recovered stock producer establishes truthful boosted pressure |
| Residual pressure | `6b76` is subtracted from `6ac2` for actual NN filling | Cross-revision engineering name and physical model accuracy remain conditional |
| `Pvdk` candidates | Modeled `2b38 -> 698c -> 63c6` | No measured pre-throttle pressure source is established |

## `S_EISY_TURBO` branches

`S_EISY_TURBO` is exact component `cal:xdf:1e00a:value`, stock raw value zero.
`func:000c4100` performs a direct one-byte read at `insn:000c420c`.

- With `S_EISY_TURBO = 0`, the function copies `global:01802d96` into raw
  `ram:r13:67b0` at `insn:000c426c`.
- With `S_EISY_TURBO = 1`, it caps `global:01802fc8`, doubles it, saturates the
  result to `0xffff`, and stores `ram:r13:67b0` at `insn:000c425c`.

The cap is **local to the request/control normalizer `67b0`**. Producer
`001038e4` forms `01802fc8 = sat_u16((P << 14) / 0x654d)` without
`K_PVDKPNRFV_MX`. Stock `S_PVDKPN_F=0` selects unfiltered `698c` for `P`.
Independently, stock `S_FTPVDKPN=1` uses this uncapped ratio in density factor
`01802fca`, which feeds actual throttle-inflow estimate `6b4c`.

Actual NN filling instead uses `max(6ac2 - 6b76, 0)`, model coefficient
`6b66`, and temperature factor `6b70`, producing `6b9a`. Canonical selectors
then publish `6b9a -> 6b6a -> 6abe -> 6ade -> 6ab6`. Enabling
`S_EISY_TURBO` alone changes request/control normalization, not the source of
that actual-filling pressure. It is not a complete boost implementation.

Fresh provenance, 14 full function/instruction exports, canonical bytes and
synthetic arithmetic probes:
[`pressure-continuation/findings.json`](../analysis/derived/review-2026-09-04-pressure-continuation/findings.json).
The static native-model research route is real; pressure acquisition,
temperature provenance, calibration domains, plausibility and physical
validation remain required. `P/Pstandard` is dimensionless normalization,
not evidence by itself of an altitude bug. Preserve raw arithmetic:
`0x654d = 25933` (nominally `1013.0078125 hPa` at `0.0390625/count`);
the executable `257/121` conversion is not exactly the ratio of rounded XDF scales.

### Acquisition/fallback closure, 2026-09-05

The [offline interface closure](../analysis/derived/review-2026-09-04-pressure-interface-closure/findings.json)
recovers logical ADC 7 through descriptor `00403d48` to bank `00704800`,
encoded channel `0x31`, result slot 8 (`00704a90`). The descriptor and
conversion program are software facts, not proof of the board net, connector
impedance or protection circuitry.

The alternate `004576f0` input is
`max(ambient 2ccc - modeled-pressure 2aca, 0)`. Diagnostic instance `0xe9`
publishes its selection flag indirectly at `r13:233c`. This fallback recycles
the existing integrated pressure; it is not another absolute-pressure sensor.
`CW_EISY` bit 5 selects/averages current and previous differential samples
before ambient subtraction and does not change their nonnegative domain.

Raw voltage thresholds, debounce/recovery and stopped-engine offset adaptation
are now source-bound. The latter delays 300 qualifying invocations, averages
100 acquisitions, accepts a bounded signed error and clamps the learned offset
to ±20 raw counts. These are invocation/count limits, **not measured seconds**.
Exact qualifications, temperature-dependent acceptance/gain, reset and
retention boundaries are in the closure; hardware fault tolerance is not
established by software thresholds.

The reviewed calibration-only interface extension is contradicted by these
sign/source constraints. This is not an exhaustive claim about every dormant
subsystem in the binary, or a rejection of the separate direct-airflow route.
No numerical scenario, test, new DME pull or firmware change was performed.

## Pressure correction and endpoint inventory

- `func:000d8668` calls the bounded interpolation family with
  `cal:xdf:08a48:y` and `cal:xdf:08c20:z`; the decoded pressure axis ends near
  900 hPa.
- `func:0043b118` uses `cal:xdf:0092e:x` and `cal:xdf:027c2:z`; that decoded
  pressure axis ends near 924 hPa.
- `func:004439b4` contains the already reviewed filling computation and exact
  `KL_PIRG_MX` storage (`cal:xdf:144a6:x`, `cal:xdf:144b2:z`).

Endpoint behavior is classified per helper; no family-wide assumption is
allowed. `func:00409680`, `func:00409d80`, `func:004165c0`, and
`func:00416658` select boundary cells outside their axes. `func:00415904`
delegates to additional helpers, so its complete multi-axis boundary behavior
remains explicitly unknown. These software contracts do not establish that the
engine, sensor, arithmetic, or diagnostic domain is safe beyond an axis.

| Limit class | Reviewed finding |
|---|---|
| Sensor / electrical | Transfer points are known; impedance, protection, and fault-voltage bounds are explicitly unknown. |
| Datatype / representation | Raw `67b0` is 16-bit; its numeric range establishes neither pressure range nor safety. |
| Arithmetic / saturation | The enabled turbo branch caps, doubles, and saturates to `0xffff`. |
| Calibration-axis endpoint hold | Pressure corrections end near 900/924 hPa; each actual helper is classified separately, with delegated behavior left unknown. |
| Plausibility / diagnostics / adaptation | Acquisition voltage/offset diagnostics, immediate fallback and stopped-engine adaptation are recovered in the offline closure; named plausibility instance `0x130`, bulk/NVM lifetime and physical behavior remain separately bounded. |
| Protection response | A diagnostic fallback is concrete; a pressure-triggered protective torque/fuel/shutdown action under boost is not established. |
| Physical safety | Safe boost, sensor suitability, fuel-pressure margin, and mechanical limits require independent instrumentation and controlled validation. |

## Plausibility, diagnostics, adaptation, and protection

The declared inventory includes focused consumers of `lc_ldm_ena_plaus_err`,
`c_map_dip_sp_mmv_diagcps`, `S_MSZYLDIAG_MOD`, `S_PREGSOL`, `K_PSMIN`,
`c_map_max_mis`, and `c_t_map_dif_dly_mis`. This represents each required
policy category without bypassing it:

- **plausibility:** enable-gated consumers are visible; the named `0x130`
  indirect-callback reachability remains a bounded static question, not proof
  of a pressure-triggered safety response;
- **diagnostics:** raw voltage and offset-shift qualification, debounce,
  recovery and immediate model-feedback substitution are recovered;
- **stopped-engine adaptation:** delay/sample counters, error/gain limits,
  direct reset and update stores are recovered; complete bulk/NVM lifetime
  and real-time physical behavior remain separate;
- **protection:** no pressure-specific torque, throttle, fuel, or shutdown
  response was recovered. Misfire pressure-transient qualification is
  diagnostic context, not proof of a protection command; the response remains
  explicitly unknown.

Zeroing or disabling these policies is not a valid substitute for extending and
validating their operating domain.

## Issue #39 handoff

Six planned rows were appended to the existing synchronized worksheet
`msv70-dynamic-observations.jsonl` (supporting artifact withheld from public edition).
They cover the stock differential input, ambient-minus-differential hypothesis,
internal Ps candidate, Pvdk candidate, both selector branches, and
stopped-engine adaptation. All remain `execution_status: not_run`, use no
invented RAM aliases, require independent absolute MAP and post-intercooler IAT,
and state falsification criteria. Correlation cannot create a static dataflow
edge.
