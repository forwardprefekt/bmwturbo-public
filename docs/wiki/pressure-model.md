# Pressure model

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page separates the MSV70's acquired differential-pressure signal, ambient-derived modeled pre-throttle pressure, integrated manifold-pressure state, request/control normalization, and modeled filling. The canonical software has pressure-aware calculations, but its stock inputs do not establish a truthful above-ambient manifold-pressure measurement. Pressure arithmetic and branch selection are known more precisely than the sensors, wiring, timing, or physical behavior.

## Canonical scope and selectors

The scope is the canonical 2006 E90 330i N52 MSV70 `0049PP0L921S0AC1S`, matching XDF `9PPL921S`, image SHA-256 `757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495`. RAM displacements are relative to `r13=0x007f8000`; a runtime address, XDF offset and file offset are different address spaces. See [identity and memory](identity-and-memory.md) and [evidence rules](evidence-and-tools.md).

Relevant canonical bytes are `CW_EISY=0` at runtime `0x85e000`, `S_EISY_TURBO=0` at `0x85e00a`, `S_EVNN_AUS=0` at `0x85e00b`, `S_PVDKPN_F=0` at `0x85e39c`, and `S_FTPVDKPN=1` at `0x85e39d`. The [pressure continuation](../../analysis/derived/review-2026-09-04-pressure-continuation/findings.json) binds these values to source bytes. `CW_EISY` bit 1 chooses modeled throttle inflow when clear and filtered airflow input when set; bit 2 chooses integrated versus sensor-reconstructed pressure; bit 5 chooses the differential-sample handling. They are not interchangeable boost switches.

## Differential-pressure acquisition

The native differential input uses a two-point XDF transfer, not an absolute manifold-pressure transfer: voltage points `cal:xdf:0110c:x` are about 0.737 and 4.502 V; differential outputs `cal:xdf:037d8:z` are about 0 and 800 hPa. At `00432fc8` the acquisition function adds unsigned ADC logical-channel-7 samples into alternating `u32` banks (`4010`/`4014`) and increments `u16` counters (`2c84`/`2c86`) according to byte `1f47`. `0013a260` consumes the selected bank during initialization; `004576f0` periodically combines the completed bank with the previous-bank sample, updates the saved sums/counts, and clears the consumed bank. Exact scheduling order is recovered, but wall-clock frequency, interrupt phase and conversion latency are not.

The initial path computes rounded `u32/u16` mean, subtracts signed learned offset `2b46`, clamps the nonnegative result at 1024 (`2b40`), then invokes the transfer helpers for `2b3c`; it copies that result to `2b3e`. Periodically, with `u8(233c)==0`, the latest mean is transferred to `2b3c` and the combined mean to `2b3e`. When `233c!=0`, both are replaced by `max(u16(2ccc)-u16(2aca),0)`. This fallback recycles ambient pressure minus existing integrated pressure; it is model feedback, not another absolute sensor. The combined mean in `2b3e` is not an independent ADC channel.

The source-bound acquisition closure resolves ADC logical 7 through descriptor `00403d48` (encoded channel `0x31`) into result slot 8 at `0x00704a90`, selected through the `0x00704800`/`0x00704c00` bank. This identifies the software acquisition route only. It does not bind the MCU channel to the connector or board net, or establish input impedance, clamp protection, reference, settling, or safe fault voltage. Connector X60005 pins 31/32/33 are documented as sensor supply/ground/signal, but analog interface properties remain unknown.

`2b40` is a bounded offset-corrected mean, not a pressure quantity until passed through the configured curve. `2b3c` and `2b3e` are `u16` transfer outputs; transfer helper endpoint behavior holds the final cell. The raw accumulators use ordinary addition and rely on scheduler/sample-count bounds for overflow avoidance, which have not been measured. The learned-offset closure has invocation-count delay/sample limits (300 delay calls and 100 samples), temperature conditions, and ±20 raw-count clamp; these are not measured seconds or proof of sensor drift behavior.

## Differential scaling and absolute-pressure candidates

`00434c48` reads `2b3e` with `LHZ` and converts it to `694e`; it separately reads `2b3c` and converts it to `6950`. Both multiply by 257 and call signed-saturating `0040b38c` for rounded division by 121. Across the full `u16` input range these callsites produce only `0..32767`. Supplying a two's-complement negative bit pattern instead enters as a large unsigned input and yields positive saturation; it cannot make this route represent above-ambient pressure.

**Withheld firmware-code evidence:** The PowerPC disassembly excerpt is omitted pending rights review. Archived source locator: `FUN_00434c48`, `00434c60–00434c84`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

Both `LHZ` instructions zero-extend their samples before the signed-saturating helper sees them. The constants `0x101` and `0x79` are 257 and 121; changing sample bits does not change that unsigned interpretation.

Ambient `2ce8` is converted to `01802da6` using `sat_u16(floor((u16(2ce8)*257+60)/121))`. The denominator is `0x79`; the addition of 60 implements rounding for this positive conversion. The sensor-reconstructed absolute candidate is `6c48=sat_u16(u16(01802da6)-s16(selected differential))`. Canonical `CW_EISY` bit 5 clear selects `694e`. If bit 5 is set, `00446994` selects or averages `6950` with its retained sample according to `K_NPK`. After coherent producer updates the differential remains nonnegative, constraining this candidate to no greater than ambient-derived `01802da6`. Bit 5 handles temporal/pulsation effects, not boost authority.

The other pressure candidate is integrated `6c6a`. `004439b4` selects `6c48` or `6c6a` into `6ac2` using `CW_EISY` bit 2; canonical zero selects the integrated state. The external measurement alternative is not a direct absolute MAP signal: it is ambient minus the converted nonnegative differential. A two-point curve edit alone cannot change that signal domain.

## Ambient and modeled pre-throttle pressure

Ambient state `2ce8` is acquired/filtered via a separate transfer and fault-gated path (`000e49d0`); its initialization is also explicit. The ambient result is converted to `01802da6`; the ambient ratio state is `01802d96`, which can be forced to zero under the recovered state predicate. Physical ambient calibration, fault behavior and update timing still need runtime binding.

`004353d0` derives modeled pre-throttle pressure candidates `2b38` and `2b3a` from `2ce8` less an interpolated air-cleaner decrement when `1fea==0`, retaining existing values otherwise. Canonical decrement cells are all zero, so an executed update copies ambient into both candidates. `000df4e8` converts `2b38` to `698c` with the same `257/121` scaling; `000b2534` filters `698c` into `63c6` using the PT1 state at `0x007fdefc` and gain `K_ZKLF=0x051f`. These are modeled pre-throttle pressure states, not a measured pressure before the throttle. Filter lag or ambient-derived initialization does not add pressure information.

## Integrated manifold-pressure model

`00447004` integrates manifold-pressure state `6c6a`. Canonical bit 1 clear selects modeled throttle inflow `6b4c` for `6c6e`; bit 1 set selects filtered airflow state `6ac0`. The latter is an optional conditional bridge, not the canonical inflow. The integrator combines inflow and auxiliary/purge terms into `6c72`, subtracts modeled outflow `6b6a` with signed saturation to form `6c70`, then applies its integration/damping term to the prior `6c6a` to produce candidate `6c76`. The retained instructions (supporting artifact withheld from public edition) are the archive’s basis for the following **authored mathematical/pseudocode summary, not firmware source**:

```text
ceiling = max(u16(63c6) - 51, 0)
6c6a = max(2560, min(candidate, ceiling))
```

The 51-count margin is `K_DPS_INTMN`; the 2560-count floor is `K_PSMIN`. Since the lower floor is applied last, it can exceed the computed ceiling when the ceiling is below 2560. This ordering is an arithmetic fact, not an authority to treat the integrated value as valid above ambient or to infer safety. The ceiling itself comes from modeled `Pvdk` (`63c6`), not a boost-capable absolute sensor. The optional airflow inflow also has a signed-Q5 upper clamp `6c72<=32767`, equivalent to 1023.96875 kg/h total inflow; selection does not bypass it.

**Withheld firmware-code evidence:** The Ghidra-reconstructed C (not original BMW source) excerpt is omitted pending rights review. Archived source locator: `FUN_00447004`. The supporting artifact is withheld from the public edition; the explanation below reports the archive’s interpretation rather than independently inspectable code proof.

`007fe3c6` is modeled pre-throttle pressure `63c6`; `0085d8dc` and `0085d8de` supply the margin and floor. The reported operation bounds candidate `6c76` by ceiling `6c7a`, then applies the floor to intermediate `6c7c`. The `6eb6` bit records the ceiling comparison before that final floor; it is not evidence of a physical overpressure response.

`004439b4`'s candidate filling path uses positive pressure residual `max(6ac2-6b76,0)`, coefficient `6b66`, and temperature factor `6b70` to produce `6b9a`. With canonical `CW_EISY=0`, integrated `6c6a` flows through `6ac2` if that selector update executes. `S_EVNN_AUS=0` selects `6b9a` into `6b6a`; `S_RFST_ON=0` then selects direct normalized modeled load `6abe -> 6ade`, followed by the final corrected load. These pressure-aware model relations are static code paths and do not demonstrate truthfully measured boosted manifold pressure or validated model accuracy. See [airflow and load](airflow-and-load.md) for the separate load and charge scales.

## Request/control normalization is a different lane

`001038e4` forms `01802fc8=sat_u16(floor((P<<14)/0x654d))` from `698c` when canonical `S_PVDKPN_F=0` selects unfiltered modeled pre-throttle pressure; the alternate setting selects `63c6`. This is a Q14-style dimensionless normalization (denominator `0x654d=25933`, nominally 1013.0078125 hPa at 0.0390625/count). The loaded input is `u16`; the reachable quotient maximum is 41403, not the nominal full Q14 unsigned capacity. There is no `K_PVDKPNRFV_MX` read in this producer.

Canonical `S_FTPVDKPN=1` selects `01802fc8` in the density factor `01802fca`; zero selects ambient-ratio variant `6c98`. The factor feeds both actual throttle-inflow estimation and inverse/request calculations. Separately, `000c4100` reads `S_EISY_TURBO`: zero copies ambient ratio `01802d96` to `67b0`; nonzero computes `sat_u16(2×min(01802fc8,K_PVDKPNRFV_MX))`, with canonical maximum `0x4000`, limiting that request/control value to `0x8000` (1.0 in Q15). This cap is local to `67b0`: it does not modify `01802fc8`, acquire manifold pressure, or alter the upstream source of actual filling. Enabling `S_EISY_TURBO` alone is not a complete boost implementation.

## Limits, evidence and unresolved boundaries

Pressure corrections and downstream maps have independent axis endpoints; for example the inspected pressure correction axes end near 900 and 924 hPa and hold boundary cells rather than extrapolating. Stock base spark load axes end at 108.36%, torque axes at 115%, and fuel/lambda families have their own mg/stk limits. These are calibration-domain boundaries, not safe operating limits. No pressure-specific torque, fuel, throttle or shutdown protection response under boost was established in the reviewed inventory. Do not disable diagnostics or protection as a substitute for extending and validating the operating domain.

The primary references are the [Ps/Pvdk dossier](../msv70-pressure-ps-pvdk-dossier.md), [pressure-corrected load trace](../msv70-boost-load-downstream-trace.md), [airflow/filling boundary](../msv70-airflow-filling-boundary.md), [pressure acquisition/interface closure](../../analysis/derived/review-2026-09-04-pressure-interface-closure/findings.json), and [pressure continuation findings](../../analysis/derived/review-2026-09-04-pressure-continuation/findings.json).

Unresolved boundaries include physical ADC-to-wire mapping and analog protection, actual sensor transfer/calibration, live fault and selector states, real invocation interval and sensor latency, ambient/MAP accuracy, temperature source and model, real pressure behavior under load, complete diagnostic/protection response, and the validity of modeled filling beyond stock calibration domains. Independent absolute manifold and ambient pressure, post-intercooler IAT, airflow, lambda and fuel-pressure instrumentation are needed for any controlled validation. No numeric representation or model output establishes mechanical safety or authorization for boosted operation.

## Related pages

- [Airflow and load](airflow-and-load.md)
- [Fuel and injection](fuel-and-injection.md)
- [Torque and controls](torque-and-controls.md)
- [Calibration and limits](calibration-and-limits.md)
- [Logging and validation](logging-and-validation.md)
- [Open questions](open-questions.md)
