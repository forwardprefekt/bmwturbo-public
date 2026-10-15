# MSV70/MSD80 pressure-sensor wiring comparison

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Date: 2026-08-27

Scope: connector-level comparison of the project MSV70/N52 wiring with the
factory MSD80/N54 two-pressure topology, plus a bounded integration plan for a
Bosch PST 3 sensor. This is a wiring research note, not approval to connect a
sensor to an uncharacterized DME input or to road-test modified firmware.

## Executive result

There is one established MSV70 pressure-input route and one unusually strong
factory-derived candidate for a second route:

```text
stock MAF                        -> X60005 pins 25/26/27/28 (retain)
post-throttle manifold Ps       -> X60005 pins 31/32/33 (existing route)
pre-throttle charge Pvdk + IAT  -> X60005 pins 9/10/11/17 (candidate route)
```

BMW WDS material assigns the N54/MSD80 charge-pressure/temperature sensor to
X60005 pins 9/10/11/17 and its post-throttle manifold-pressure sensor to pins
31/32/33. The reviewed MSV70 WDS export assigns pins 31/32/33 to the manifold
pressure sensor, assigns pins 25--28 to the MAF/IAT, and marks pins 9--11 and
17 unused.

That connector alignment is direct wiring evidence. It does **not** prove that
the project MSV70 PCB has the N54 input protection, filtering, 5 V supply,
temperature pull-up, ADC routing, or software acquisition populated behind the
unused cavities. The second input is a bench-test candidate, not a confirmed
plug-in feature.

## Evidence and provenance

| Source | Directly supports | Evidence boundary |
|---|---|---|
| BMW WDS A6000 MSV70/N52 connector export ([mirror](https://www.e90post.com/forums/attachment.php?attachmentid=1566814&d=1486137638)) | X60005 pins 9--11 and 17 unused; MAF/IAT on 25--28; manifold-pressure supply/ground/signal on 31/32/33 | Harness/connector assignment only; no PCB input-stage specification |
| BMW WDS A6000 N54 module 5/6 ([direct SVG](https://wds.psjr.org/en/e90/svg/SP0000024783.svg)) | Exact `PNDK` and `PVDK/TVDK` DME pins, sensor connector pins, wire colors, and 0.5 mm2 conductors | Production MSD80/N54 wiring; not proof of MSV70 population |
| BMW, *2007 Engine Management* ([mirror](https://www.e90post.com/forums/attachment.php?attachmentid=50388&d=1165592709)) | Separate pressure/temperature before throttle and manifold pressure after throttle; throttle is the physical boundary | OEM architectural description, not a cross-DME porting instruction |
| [Bosch Motorsport PST 3](https://www.bosch-motorsport.com/products/sensors/pressure-and-temperature/pressure-sensor-combined-pst-3/) | Order number, range, supply, pressure transfer, NTC table, connector pinout, mounting notes | Manufacturer specification for Bosch `0 261 230 280`; not the N54 production sensor transfer |
| [`msv70-pressure-ps-pvdk-dossier.md`](msv70-pressure-ps-pvdk-dossier.md) | MSV70 pressure acquisition, conversion components, diagnostics leads, and explicit unknowns | Static firmware evidence; no electrical input-stage characterization |
| `msd80-n54-factory-turbo-reference.md` (supporting artifact withheld from public edition) | Four stock MSD80 binaries, exact provenance, pressure transfer, and boosted domains | Factory comparison evidence; not directly reusable MSV70 calibration |

Retrieved-source fingerprints are recorded so a later researcher can detect a
changed mirror response:

| Retrieved source | Size | SHA-256 |
|---|---:|---|
| MSV70 WDS PDF | 678,348 bytes | `f4005c4e9531ff43fffa8ecf6ceb02eecf666851754de23f7ca95c54c09bac24` |
| N54 WDS module 5/6 SVG | 77,684 bytes | `73188ec1cc948fea3aa78e42adbbf5478e9d273dd9d7258e76df26b5b67c20a1` |

## Direct WDS connector findings

### MSV70/N52

The reviewed MSV70 WDS export gives these X60005 assignments:

| DME pin | WDS function |
|---:|---|
| 9, 10, 11 | Not used |
| 17 | Not used |
| 25 | MAF supply |
| 26 | MAF signal |
| 27 | MAF ground |
| 28 | MAF intake-air-temperature signal |
| 31 | Manifold-pressure sensor supply |
| 32 | Manifold-pressure sensor ground |
| 33 | Manifold-pressure sensor signal |

The project MSV70 training reference describes the production sensor at the
rear of the intake manifold as a differential-pressure sensor. Connector names
such as "manifold pressure" therefore do not prove that stock firmware already
converts this channel as a boost-capable absolute MAP.

### MSD80/N54

The BMW N54 WDS SVG labels the post-throttle channel `PNDK` and the
pre-throttle charge channel `PVDK`; `TVDK` is charge-air temperature.

| Physical state | DME X60005 pin | WDS signal | N54 sensor connector pin |
|---|---:|---|---:|
| Post-throttle sensor supply | 31 | `U_PNDK` | 2 |
| Post-throttle sensor ground | 32 | `M_PNDK` | 4 |
| Post-throttle pressure signal | 33 | `A_PNDK` | 1 |
| Pre-throttle sensor ground | 9 | `M_PVDK` | 4 |
| Pre-throttle pressure signal | 10 | `A_PVDK` | 1 |
| Pre-throttle sensor supply | 11 | `U_PVDK` | 2 |
| Pre-throttle temperature signal | 17 | `A_TVDK` | 3 |

The N54 diagram specifies 0.5 mm2 conductors. Its sensor connector numbering is
specific to the production N54 sensor and must not be copied onto a different
sensor by matching pin number rather than electrical function.

## Bosch PST 3 candidate

Bosch order number `0 261 230 280` is a combined absolute pressure/temperature
sensor with these manufacturer-published properties:

- pressure range: 0.2--3.0 bar absolute;
- pressure output: 0.4--4.65 V at a 5 V supply;
- supply: 4.75--5.25 V, 9 mA nominal;
- temperature element: passive NTC, -40--130 degrees C;
- sensor pins: 1 ground, 2 temperature, 3 supply, 4 pressure;
- mounting: 12.05 mm fitting, O-ring seal, M6 retainer, within 60 degrees of
  vertical to limit condensate damage.

Using Bosch's published sensitivity and offset, the illustrative pressure
conversion is:

```text
P_abs_bar = (V_signal - 0.09643 V) / 1.518 V/bar
```

At 1.0 bar absolute, the expected pressure signal is approximately 1.614 V.
The 3.0 bar absolute full scale corresponds to approximately 2.0 bar / 29 psi
gauge at sea level; required transient headroom must be checked against the
actual boost target and ambient pressure.

This is **not** the same transfer as the 0.2--2.5 bar class conversion recovered
from the stock MSD80 binaries. The PST 3 requires its own linearization and
diagnostic thresholds.

## Purchased N54 manifold-sensor candidate

A used OEM N54 manifold-pressure sensor is an unverified hardware candidate.
Private transaction details are omitted. The recorded application does not
establish its exact BMW or Bosch part number. A reusable public listing suggests
BMW `13627551429` / short number `7551429`; these remain candidates until read
from the actual sensor marking.

If the delivered identity matches, the factory WDS alignment makes it a strong
**bench-only candidate** for the post-throttle `Ps` route on X60005 pins
31/32/33. The four stock MSD80 binaries also provide a reproducible
0.2--2.5-bar-absolute transfer candidate. Neither fact authorizes applying that
transfer to the purchased part before a pressure/voltage sweep, or connecting
the part to a running-engine MSV70 before the DME supply, ground, ADC, input
protection, and diagnostic behavior are characterized.

Historical boundary: this wiring note originally treated ambient-minus-
differential as an unproven software hypothesis. The later pressure dossier
recovers that software relationship and its unsigned-domain limitation. This
does not establish a truthful boosted interface: replacing two stock conversion
points alone remains insufficient.

| Use | Current decision | Next gate |
|---|---|---|
| Protected external logger | Candidate | Confirm delivered marking and connector; bench-measure supply current and pressure/voltage transfer |
| MSV70 post-throttle `Ps`, pins 31/32/33 | Bench only | Characterize DME supply, sensor ground, ADC range/protection, raw acquisition, diagnostics, and substitution behavior |
| MSV70 pre-throttle `Pvdk`, pins 9/10/11/17 | Not this sensor role | Prove PCB population and acquisition path, then use a separately characterized pressure/temperature sensor |

Public reusable-listing reference:
https://www.ebay.com/itm/227213953181

## Candidate wiring by electrical function

### Established MSV70 route for post-throttle `Ps`

If the PST 3 is used as the DME's manifold absolute-pressure sensor, it must be
mounted after the throttle and wired by function:

| PST 3 pin | Function | MSV70 X60005 pin |
|---:|---|---:|
| 1 | Sensor ground | 32 |
| 2 | Temperature NTC | Not available on the three-wire route; isolate or route only to a separately characterized input |
| 3 | 5 V supply | 31 |
| 4 | Pressure signal | 33 |

This route reuses an existing MSV70 pressure acquisition, but it is not a
calibration-only swap. The stock differential transfer, conversion semantics,
fault window, plausibility checks, and downstream consumers must be traced and
modified for absolute pressure before a running-engine test.

### Candidate N54-style route for pre-throttle `Pvdk` and IAT

If bench characterization proves that the MSV70 input hardware and software
path exist behind its unused cavities, the function-correct PST 3 wiring would
be:

| PST 3 pin | Function | Candidate MSV70 X60005 pin |
|---:|---|---:|
| 1 | Sensor ground | 9 |
| 2 | Temperature NTC | 17 |
| 3 | 5 V supply | 11 |
| 4 | Pressure signal | 10 |

The sensor belongs in the cold charge path after the intercooler and before the
throttle for this role. A pre-throttle sensor must not be connected to the
post-throttle `Ps` input: the two pressures diverge whenever the throttle
restricts flow or closes.

The stock MAF remains on pins 25--28. The candidate charge-sensor route does
not require deleting, electrically spoofing, or replacing the MAF.

## Required bench validation

Do not connect the sensor to pins 9/10/11/17 until all relevant gates below are
passed on a spare DME or protected breakout setup.

1. Confirm the exact project-car WDS assignment by VIN/build date and verify
   connector cavity numbering from the shell, not viewing orientation.
2. Compare a spare MSV70 with an MSD80 around X60005 pins 9/10/11/17. Record
   continuity and the population of series resistors, RC filtering, clamps,
   temperature pull-up, supply switching, and MCU/ADC destinations.
3. With power removed, check for shorts to ground, battery, and adjacent pins.
4. On a current-limited bench supply, measure X60005 pin 11 relative to sensor
   ground pin 9. Do not assume it is 5 V merely because the N54 uses that pin.
5. If the pressure input stage is confirmed, sweep approximately 0.4--4.65 V
   into pin 10 through a protective series resistance and identify the raw ADC,
   filtering, scaling, diagnostics, and update cadence in RAM.
6. Test pin 17 with a resistance substitution box spanning Bosch's published
   NTC values. Do not drive the suspected passive-temperature input directly
   with a voltage source.
7. Separately characterize pins 31/32/33: supply tolerance/current capability,
   ADC input impedance and clamps, raw range, fault-voltage window, diagnostic
   consequences, and software conversion.
8. Implement explicit pressure/temperature transfers and safe low/high fault
   thresholds. Do not reuse stock N54 values without binding their exact
   consumers.
9. Cross-check DME pressure and temperature against an independent MAP and IAT
   logger through vacuum, key-on ambient, throttle transients, and only then
   conservative positive pressure.

Use proper repair terminals and sealed sensor connectors. Route signal with
its DME sensor ground along the factory harness, away from ignition and
high-current/PWM wiring. Do not use chassis ground for the sensor, backfeed an
external 5 V supply into an uncharacterized DME rail, or share an analog output
with another controller until both input impedances and ground offsets are
known.

## Decision boundary

| Bench result | Integration decision |
|---|---|
| Pins 9/10/11/17 have the required populated input stages and a traceable acquisition path | Populate the unused harness cavities and implement the N54-style pre-throttle `Pvdk`/IAT path, while retaining pins 31/32/33 for post-throttle `Ps` |
| Connector pads exist but conditioning/pull-up parts are unpopulated | Do not copy components from an MSD80 without a reviewed schematic/BOM-level comparison; prefer a separate conditioned input design |
| No credible MSV70 input path exists | Use pins 31/32/33 for absolute manifold `Ps`; keep pre-throttle pressure on independent instrumentation until a separately protected interface and software acquisition are designed |

One sensor cannot represent both physical states across the throttle. A full
factory-style model ultimately needs `Ps` after the throttle and `Pvdk` before
it, plus independent instrumentation for validation and fail-safe decisions.
