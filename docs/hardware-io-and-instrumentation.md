# MSV70 hardware I/O and instrumentation plan

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Recorded turbo-relevant MSV70 wiring leads

Verify against exact WDS/build date before altering wiring.

- X60005 pins 9/10/11/17 — unused in the reviewed MSV70 WDS; factory N54/MSD80
  uses these positions for pre-throttle charge pressure/temperature
- X60005 pin 25 — MAF supply
- X60005 pin 26 — MAF signal
- X60005 pin 27 — MAF ground
- X60005 pin 28 — intake-air temperature
- X60005 pin 31 — manifold differential-pressure sensor supply
- X60005 pin 32 — manifold differential-pressure sensor ground
- X60005 pin 33 — manifold differential-pressure sensor signal

The N54 connector alignment is a strong bench-test lead, not proof that the
MSV70 PCB has the corresponding input stages populated. Exact WDS mappings,
Bosch PST 3 function-to-pin wiring, electrical unknowns, and validation gates
are in
[`msv70-msd80-pressure-sensor-wiring.md`](msv70-msd80-pressure-sensor-wiring.md).

## Required independent instrumentation

1. **Manifold absolute pressure** — independent of whatever signal the modified DME ultimately uses.
2. **Fuel rail pressure** — log injector differential pressure as `rail pressure - manifold pressure`.
3. **Independent wideband lambda** — sanity check against the DME being modified.
4. **Post-intercooler IAT** — charge temperature entering the engine, not compressor-inlet temperature.

## Pressure sensor work still required

Do not buy a generic 3-bar MAP merely because it is 0–5 V.

Before choosing the DME-connected `Ps` sensor:

- characterize sensor supply
- characterize ADC/input voltage range
- determine fault/plausibility thresholds
- prove input protection behavior
- define exact pressure transfer function
- update linearization and all relevant diagnostics

For a second, pre-throttle `Pvdk`/IAT path, additionally prove the PCB
population, 5 V supply, pressure ADC, NTC pull-up, filtering, protection, and
software acquisition behind X60005 pins 9/10/11/17. Do not delete the stock MAF
to obtain this input; the WDS-derived candidate uses separate cavities.

## PCV/CCV requirement

The boosted engine must never allow positive manifold pressure into the crankcase through the vacuum-side ventilation route. Maintain a controlled evacuation path under boost, typically toward pre-compressor suction, with suitable one-way isolation and oil separation.
