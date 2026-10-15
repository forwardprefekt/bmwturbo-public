# Research backlog

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Priority A — required before meaningful boost

- [x] Reproduce a disposable Ghidra project from canonical firmware/XDF inputs
  and verify it against the pinned baseline (#25 (supporting artifact withheld from public edition)).
- [x] Recreate/export the current pressure signal trace with code addresses and
  consumers (`msv70-rom-map.md` and `msv70-boost-load-downstream-trace.md`).
- [x] Trace final pressure-corrected cylinder filling into the proven spark,
  MDIOP, and lambda indexing paths (#27 (supporting artifact withheld from public edition)).
- [ ] Capture the canonical modeled route `6b9a -> 6b6a -> 6abe -> 6ade -> 6ab6`
  and the separate measured-MAF candidate in the same unchanged baseline.
  `S_RFST_ON=0` resolves the nested stock selector. The bounded
  `2b30 -> 6b5a/6b9a` search is not a prerequisite for either load architecture
  (#34 (supporting artifact withheld from public edition)).
- [ ] Complete the exhaustive axis/limiter and active-selector inventory above
  naturally aspirated load/filling; the 25-table focused catalog is not the
  whole program.
- [x] Complete the bounded `Ps`/`Pvdk` pressure inventory with explicit unknown
  plausibility, diagnostics, adaptation, and protection boundaries
  (#35 (supporting artifact withheld from public edition)); dynamic execution remains #39.
- [ ] Characterize the X60005-31/32/33 pressure-sensor electrical input stage and fault bounds.
- [ ] Choose a boost-capable manifold absolute pressure sensor only after the electrical characterization.
- [ ] For the native modeled route, define a truthful absolute-`Ps` interface
  instead of injecting absolute values into unsigned differential states.
- [ ] Prove a truthful `Pvdk` producer and charge-temperature provenance.
  `S_EISY_TURBO` changes request normalization; actual modeled filling already
  depends on pressure through `6ac2`, with an ambient-derived integrator bound.
- [x] Compare factory MSV70 and MSD80 pressure-sensor connector assignments;
  document the existing 31/32/33 `Ps` route and candidate 9/10/11/17
  `Pvdk`/IAT route
  ([wiring note](msv70-msd80-pressure-sensor-wiring.md)).
- [ ] Compare spare MSV70/MSD80 PCB population and continuity behind X60005
  pins 9/10/11/17; prove supply, filtering/protection, NTC pull-up, ADC routing,
  and raw software acquisition before adding harness terminals.
- [x] Trace injector flow intent, dead-time, voltage compensation, wall film,
  and pulse constraints through the final per-cylinder main-pulse software
  values (#36 (supporting artifact withheld from public edition)); maximum
  pulse, physical duty margin, and hardware/runtime acceptance remain #39.
- [ ] Verify fuel-system capability using measured rail pressure minus manifold pressure under load.

## Priority B — integration quality

- [x] Audit `KF_MDIOP`, `KL_MDMAX`, torque monitors, and software
  intervention boundaries (#37 (supporting artifact withheld from public edition));
  runtime selectors, actuator response, and boosted acceptance remain #39.
- [ ] Audit pressure-domain maps currently ending around ~900–924 hPa
  (#35 (supporting artifact withheld from public edition)).
- [ ] Investigate lambda/O2 transport-delay parameters for post-turbine sensor placement.
- [ ] Validate Valvetronic high-load behavior before optimizing lift for boost.
- [ ] Retune VANOS only after fueling and ignition are stable.
- [ ] Validate throttle as a torque/overboost intervention layer.
- [x] Trace the static knock, lambda/component-protection, modeled
  exhaust-temperature, and diagnostic/fail-safe boundaries
  (#38 (supporting artifact withheld from public edition)); physical
  margins, active selectors, and runtime consequences remain #39.

## Hardware / validation

- [x] Compile the issue-#34 through issue-#38 handoffs into a deterministic
  issue-#39 worksheet, predeclared outcome/channel/timebase contracts, and a
  fail-closed capture validator. This completes static protocol preparation,
  not sensor execution.
- [ ] Execute the synchronized MSV70 RAM and independent-instrumentation
  protocol (#39 (supporting artifact withheld from public edition)).
- [x] Define the minimal pre-manifold CAN baseline as five stationary scenario
  files, with an optional hot restart and one optional gentle road/load file;
  record a machine-readable catalog, operator sheet, CAN-only evidence
  boundary, and post-capture analysis flow in the separate private archive.
- [x] Resolve 72 indirect state views and the one mixed-width observation
  view with exact manifest-bound byte/halfword contracts; preserve their
  reviewed evidence class rather than calling them direct facts.
- [ ] Implement and retain the issue-#22 logger configuration for those 73
  reviewed views and measure their actual sample/update rates.
- [ ] Measure and retain a passing host-bracketed affine DME/logger timebase.
- [ ] Retain baseline and bounded naturally aspirated captures with complete
  raw timestamps, calibrations, fault/status records, and artifact hashes.
- [ ] Independent manifold MAP logging.
- [ ] Fuel-pressure transducer.
- [ ] Independent wideband sanity check.
- [ ] Post-intercooler IAT.
- [ ] Build a protected X60005 breakout and execute the pressure-input voltage
  sweep plus temperature-resistance substitution protocol.
- [ ] Boost-safe PCV/CCV with one-way isolation from manifold boost.
- [ ] Lowest practical wastegate spring for first startup.
- [ ] Pressure-test intake manifold / DISA / charge system before boost.

## Evidence infrastructure completed

- [x] Manifest-driven focused targeting (#26 (supporting artifact withheld from public edition)).
- [x] Structured function/call/RAM/calibration facts (#27 (supporting artifact withheld from public edition)).
- [x] Disposable SQLite/FTS5 search index (#28 (supporting artifact withheld from public edition)).
- [x] Evidence-backed, cycle-safe relationship traversal (#29 (supporting artifact withheld from public edition)); it preserves the unproven airflow/filling gap.
