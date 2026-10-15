# MSV70 naturally aspirated CAN baseline capture runbook

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Issue #39 (supporting artifact withheld from public edition) owns the
synchronized runtime validation. This runbook turns its broad perturbation
classes into a small recording session that can be completed before the stock
manifold is removed. The machine-readable source is
`analysis/validation/msv70-can-capture-scenarios.json` (supporting artifact withheld from public edition),
and the fill-in operator sheet is
`analysis/validation/msv70-can-session-log.example.csv` (supporting artifact withheld from public edition).

All RPM, pedal, time, and duration values below are **procedure defaults**, not
proven MSV70 thresholds, firmware units, or newly declared safety limits.
Adjust or skip a step when normal vehicle condition requires it. The purpose is
repeatable excitation, not stress testing.

## Short answer

Most useful pre-turbo work can be done stationary. The minimum worthwhile set
is five files:

| ID | Situation | Approximate recording time | Why keep it before the swap |
|---|---|---:|---|
| `NA-01` | Key off -> KOEO -> key off | 70 s | Bus population, wake/state transitions, pressure-at-ambient context |
| `NA-02` | KOEO pedal sweep and steps | 110 s | Clean pedal/driver-request stimulus without RPM or engine-load confounders |
| `NA-03` | Cold start, no pedal, first two minutes | 150 s | Stock start, airflow/filling, voltage, VANOS/Valvetronic, and lambda context |
| `NA-04` | Warm idle, accessory loads on/off | 130 s | Stationary torque/load compensation without a driver request |
| `NA-05` | Warm 1200/1800/2500 RPM holds and small blips | 170 s | Repeatable airflow, pressure, injection, and controller transients |

That is about eleven minutes of raw recording plus whatever time the car needs
to warm normally. `NA-06` hot restart is useful but optional. `NA-10` combines
all worthwhile moving tests into one roughly ten-minute drive and is optional
until the muffler/exhaust is roadworthy. There is no need to perform a WOT pull
or approach the stationary limiter.

## What raw CAN can and cannot validate

Raw CAN is valuable for discovering frames and testing whether candidate
fields change in the right direction, at the right time, and repeatably under
a known stimulus. For example, the KOEO pedal sequence can isolate a broadcast
pedal/request field if one exists, and the warm accessory steps can expose a
broadcast torque/load-compensation response without vehicle motion.

Raw CAN alone cannot prove the engineering meaning of anonymous MSV70 RAM. It
also cannot establish physical airflow provenance or activate a recovered
conditional airflow/filling route,
identify the active `Ps`/`Pvdk` producer, prove `S_EISY_TURBO` branch selection,
distinguish `MDIOP` from `MDMAX`, or show which lambda/injector/table-index path
ran. A signal may not be broadcast at all. Therefore:

- matching timing/direction makes a CAN decode a **candidate**;
- a missing CAN response is **unresolved**, not proof that the DME state did
  not change;
- correlation never creates a firmware `DERIVES_FROM` relationship; and
- final semantic outcomes remain `support`, `contradict`, or `unresolved`
  under the synchronized protocol, with exact capture hashes.

The decisive future dataset aligns three layers:

1. raw CAN for vehicle/bus context;
2. exact DME RAM/service states from the issue-39 worksheet; and
3. calibrated independent pressure, MAF, lambda, fuel-pressure, temperature,
   and other channels where required.

The first raw-CAN session is still worth doing before those other layers are
ready because it preserves the unmodified naturally aspirated behavior and
gives us the frame-discovery dataset.

## Do this before removing the stock manifold

Keep this list short:

1. Do `NA-01` through `NA-05`. If a stationary exhaust problem makes `NA-05`
   unreasonable, capture `NA-01` through `NA-04` and record why it was skipped.
2. If the muffler is repaired and one normal drive is practical, do `NA-10`.
   It is the only listed test that separates real road load from stationary
   free-rev behavior, so it is high value, but it is not required to begin CAN
   decoding.
3. Save a DTC snapshot without clearing codes or adaptations. Record the exact
   DME software/tune/full-read identity and hash, fuel grade, ambient
   conditions, and any non-stock hardware.
4. Photograph and label the stock MAF, rear manifold pressure sensor,
   throttle, connectors, wire colors, hose/orifice routing, and part numbers.
5. Keep the removed stock manifold pressure sensor, its sealing hardware, and
   ideally a serviceable pigtail/manifold pressure port. Do not discard them;
   they allow the stock differential transfer to be exercised on a protected
   bench later.

There is no need to install every future sensor, intentionally unplug the MAF,
induce a pressure-sensor fault, add EGT hardware, or perform a high-load run
just to complete this baseline. MAF and pressure-sensor disconnect tests are
deferred until synchronized internal logging can make the resulting DTC and
fallback behavior answer a specific question.

Do not reset adaptations or flash a different calibration between the
configuration snapshot and these captures. If either is unavoidable, treat
the post-change data as a new session rather than mixing it with the baseline.

## Recording rules that save work later

- Use one native raw file per scenario. Do not pre-convert, rescale, filter, or
  delete frames.
- Use the adapter's timestamps and retain the original export. Record adapter,
  software version, CAN bus/channel, configured bitrate, and whether hardware
  or host timestamps were used.
- Prefer names such as
  `20260827_NA-02_koeo-pedal_r1.asc`. If the tool can only create one long
  recording, use bookmarks or write down recorder elapsed times for each
  scenario.
- Leave controls alone for 10 seconds at the beginning and end of each file.
  These quiet windows make counters, multiplexing, and baseline traffic easier
  to separate from the intended event.
- Write down deviations, not a perfect transcript. A gear change, missed RPM
  hold, accessory that did not engage, warning, or recorder restart matters;
  ordinary small timing error does not.
- Keep the raw file plus the completed operator CSV together. When the files
  arrive, repository intake will calculate SHA-256 hashes and create the full
  issue-39 capture manifest.

Native `candump`, ASC/BLF, PCAP/PCAPNG, SavvyCAN CSV, or another lossless
timestamped adapter export is acceptable. The exact parser can be selected
after the adapter/tool format is known; preserving the original bytes is more
important than converting in advance.

## Scenario cards

### `NA-01` — key state and bus wake

Vehicle stationary. Begin with ignition off and no control inputs for 20
seconds. Switch ignition on without starting and wait untouched for 30 seconds.
Switch ignition off and wait untouched for 20 seconds.

This is also the recording shakedown. Before proceeding, confirm the file has
timestamps and nonzero CAN traffic and that the recorder stayed connected
through the state changes.

### `NA-02` — engine off, pedal input

KOEO, engine definitely not running. Slowly move the accelerator pedal through
approximately 0/25/50/75/100 percent, pausing about three seconds at each point,
then return slowly to zero. Wait 10 seconds. Make three distinct steps to about
25/50/100 percent with a full return between them. Wait 10 seconds and repeat
once.

This moves the **accelerator pedal**; it does not assume the electronic
throttle plate will mirror it with the engine off. That distinction is useful
evidence rather than a test failure.

### `NA-03` — cold start

Use the first ordinary start after a documented cold soak. Record approximate
soak duration and ambient temperature. Capture 10 seconds KOEO, start without
pedal input, then leave pedal and accessories unchanged for 120 seconds.

This is the only scenario that benefits from a separate cold start. It does
not require idling all the way to full temperature; warm-idle scenarios can be
recorded whenever the car becomes warm normally.

### `NA-04` — warm idle accessory loads

At stable warm idle, record 20 seconds with climate control, lamps, blower, and
rear defogger off. Turn on one documented electrical bundle (for example,
headlamps + blower + rear defogger) for 20 seconds, then off for 20 seconds. If
A/C operates normally, turn it on for 20 seconds and off for 20 seconds. Record
what actually engaged. Do not add artificial steering-at-lock or stall loads.

This helps distinguish a DME torque/load response from a direct pedal request
without requiring movement.

### `NA-05` — stationary RPM holds and small transients

Fully warm, securely stationary, transmission in Park or Neutral. Hold about
1200 RPM for five seconds, idle 10 seconds; hold 1800 for five, idle 10; hold
2500 for five, idle 10. Repeat the ladder once, then make three small smooth
blips that remain below about 2500 RPM. Do not approach the free-rev limiter.

### `NA-06` — optional hot restart

Record 20 seconds of warm idle, shut down for 60 seconds untouched, switch
ignition on for 10 seconds, restart without pedal input, and record 60 seconds
of idle. Skip this if another start is undesirable because of exhaust or
battery condition.

### `NA-10` — optional single road/load recording

Do this only after the muffler/exhaust and car are roadworthy, on a safe legal
route or controlled dyno. In one recording:

- hold two or three ordinary steady cruise conditions for about 20 seconds;
- make two smooth moderate accelerations from roughly 1500 to 3000 RPM with
  about 20--40 percent pedal when traffic and transmission behavior permit;
- include one ordinary closed-pedal coast-down; and
- record transmission mode and unexpected shifts.

This is intentionally a normal-driving comparison, not WOT or a maximum-load
test. It adds the moving/load dimension that stationary captures cannot.

## How the captures will be analyzed

When the files arrive:

1. archive the native recordings unchanged, calculate hashes, and record tool,
   bus, timestamp, vehicle, tune, DTC, and scenario provenance;
2. normalize timestamp, bus, arbitration ID, DLC, and raw data bytes without
   assigning physical meaning;
3. check timestamp precision, gaps, dropped/reordered records, changing bus
   population, and counter/multiplex patterns;
4. segment by scenario and quiet/event windows;
5. rank candidate bytes/bits by change timing, direction, entropy, repeatability,
   and correlation to the deliberately changed input;
6. compare candidates across scenarios to reject false decodes—for example, a
   true pedal candidate should repeat in `NA-02`, while a speed candidate must
   remain zero there and change in `NA-10`;
7. once DME RAM/service and independent logger streams exist, fit the issue-39
   host-bracketed affine timebase and overlay the exact reviewed states; and
8. evaluate each hypothesis using its predeclared support, contradiction, or
   unresolved rule and cite the exact raw/analysis hashes.

Useful outputs will include per-ID activity timelines, byte/bit change heatmaps,
candidate-versus-reference overlays, lag plots, counter/multiplex diagnostics,
and state-transition tables. A promising correlation remains a candidate until
the firmware and synchronized evidence support the same interpretation.

## Which speculation each situation addresses

| Question | Best scenario(s) | CAN alone | What closes the question |
|---|---|---|---|
| Which fields represent ignition, crank/run, RPM, or counters? | `NA-01`, `NA-03`, `NA-06` | Strong candidate identification | Cross-scenario repeatability and traced source |
| Is pedal/driver request broadcast? | `NA-02`, then `NA-05` | Candidate direction, scale, and lag | DME pedal/request and arbitration RAM |
| Does torque compensation occur without pedal? | `NA-04` | Candidate broadcast torque/load response | `MDIOP`/`MDMAX`/request states plus independent torque if claimed |
| How do throttle and Valvetronic share authority? | `NA-04`, `NA-05`, `NA-10` | Only if their positions/requests are broadcast | Synchronized throttle, lift, selectors, and controller RAM |
| Do `Ps` and `Pvdk` behave like post-/pre-throttle pressure? | `NA-01`, `NA-03`, `NA-05`, `NA-10` | Context only | Independent absolute pressures plus instruction-bound DME candidates |
| Does selected MAF track filling/load? | `NA-03`, `NA-05`, `NA-10` | Correlation candidate only | Exact selected-airflow/filling states; no causal edge from correlation |
| Does injector correction follow rail minus manifold pressure? | `NA-04`, `NA-05`, `NA-10` | No | Calibrated rail/MAP, injector command, and pressure-correction RAM |
| Which lambda/ignition/torque table branch is active or at an endpoint? | `NA-05`, especially `NA-10` | No | Exact selectors, table coordinates/indexes, delivered lambda/spark/torque |
| What does `S_EISY_TURBO` change? | None of the raw-CAN baseline alone | No | Non-driving branch-aware DME capture; no firmware mutation is authorized |

## After the turbo conversion

Repeat the harmless identity/commissioning portions—KOEO pressure sanity,
first idle, warm idle loads, and low-RPM stationary holds—under the later
commissioning plan so the NA and modified states share scenario shapes. Do not
treat this document as authorization for a boosted pull. Boosted pressure/load
domain, lambda, fuel-pressure, knock, thermal, and torque plausibility testing
belongs in a separate staged commissioning protocol with calibrated sensors and
explicit stop limits.

Related architecture and static evidence are documented in
[`msv70-msd80-pressure-sensor-wiring.md`](msv70-msd80-pressure-sensor-wiring.md),
[`msv70-pressure-ps-pvdk-dossier.md`](msv70-pressure-ps-pvdk-dossier.md),
[`msv70-airflow-filling-boundary.md`](msv70-airflow-filling-boundary.md), and
[`msv70-torque-authority-dossier.md`](msv70-torque-authority-dossier.md).
