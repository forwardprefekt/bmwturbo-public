# ESP32 Sensor Logger / Drift DAQ

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Parent tracker: issue #22

## Goal

Build a cheap, independent sensor logger for the N52 turbo drift car using an ESP32 rather than a commercial motorsport DAQ.

The logger should record the physical quantities that can hurt the engine even if MSV70 diagnostic logging is unavailable:

- engine oil pressure
- fuel rail pressure
- independent manifold pressure / boost
- post-intercooler IAT
- crankcase pressure
- optional coolant pressure
- BMW CAN data such as RPM / pedal / throttle / coolant temperature where decoded

The logger is **not** the sole safety system. Catastrophic oil-pressure warning and boost fallback should remain independently actionable.

## Current architecture decision

**Do not design a custom PCB for Rev 0 or Rev 1.**

Start with commodity modules and a proper harness:

```text
switched 12 V
    |
    +-- fuse / input protection
    |
    +-- 12 V -> 5 V supply --------------------+
    |                                           |
    |                                      5 V sensors
    |                                           |
    +-- ESP32 DevKit                            |
          |                                     |
          +-- SPI -> microSD                    |
          +-- TWAI -> CAN transceiver -> BMW CAN (listen-only initially)
          |
          +-- ADC channels <- divider/filter <- sensor signals
          |
          +-- warning output -> relay/transistor -> lamp/buzzer
```

Rev 2 may become a small custom PCB after channel count, connector choices, filtering, and power behavior are proven in the car.

## Suggested channel plan

| Channel | Signal | Expected sensor type | Initial log rate |
|---|---|---|---:|
| ADC1 | Engine oil pressure | 0.5-4.5 V, ~0-150 psi | 50 Hz |
| ADC2 | Fuel rail pressure | 0.5-4.5 V, ~0-150 psi | 50 Hz |
| ADC3 | Independent manifold MAP | 0.5-4.5 V absolute MAP | 100 Hz |
| ADC4 | Crankcase pressure | low-range compound/differential | 25-50 Hz |
| ADC5 | Coolant pressure (optional) | low-range pressure | 25 Hz |
| ADC6 | 5 V sensor supply monitor | divider from sensor 5 V rail | 25 Hz |
| ADC7 / temp | Post-IC IAT | NTC or conditioned analog | 10 Hz |
| CAN | RPM, pedal, throttle, temperatures, etc. | BMW PT-CAN | message-native |

Use ADC1-capable pins on the classic ESP32 if Wi-Fi is enabled; ADC2 conflicts with Wi-Fi on that platform.

## Rev 0 — bench prototype

### Order / gather

- [ ] ESP32 DevKit / ESP32-WROOM-32 development board
- [ ] 3.3 V CAN transceiver breakout (SN65HVD230-class is fine for bench/prototype)
- [ ] microSD SPI breakout that is compatible with 3.3 V logic
- [ ] 16-32 GB high-endurance microSD card
- [ ] 12 V -> 5 V buck converter
- [ ] Inline fuse holder + ~1 A fuse
- [ ] Breadboard/perfboard, headers, hookup wire
- [ ] Resistor assortment (1% preferred)
- [ ] 0.1 uF ceramic capacitors for ADC filtering
- [ ] 2x identical 0.5-4.5 V / 0-150 psi pressure sensors for oil + fuel
- [ ] One independent absolute MAP sensor sized for expected boost
- [ ] Known mechanical pressure gauge / regulated air source for calibration

### Sensor input prototype

For a typical 0.5-4.5 V sensor, start with a divider/filter such as:

```text
sensor signal --- 12k ---+--- ESP32 ADC
                         |
                        10k
                         |
                        GND

ADC node -------- 0.1 uF -------- GND
```

This keeps a 4.5 V sensor output safely below the ESP32 input range and provides basic low-pass filtering.

- [ ] Verify actual divider values with a multimeter before connecting ESP32
- [ ] Measure 0 V, ~0.5 V, 2.5 V and ~4.5 V test points
- [ ] Confirm ADC never sees an unsafe voltage
- [ ] Implement ESP-IDF ADC calibration
- [ ] Oversample raw ADC and average/filter in software
- [ ] Log the 5 V sensor supply so ratiometric sensors can be compensated

### Bench calibration

- [ ] Calibrate oil-pressure sensor against known gauge at several pressures
- [ ] Calibrate fuel-pressure sensor separately even if same model
- [ ] Record slope, intercept, supply voltage, sensor serial/markings
- [ ] Plot residual/error in a notebook
- [ ] Reject sensors with bad linearity, hysteresis, or unstable output
- [ ] Verify MAP sensor against atmospheric pressure / vacuum / regulated pressure if available

## Rev 1 — in-car prototype

### Power

A bare hobby buck converter is acceptable for bench work but should not be trusted indefinitely on vehicle power.

- [ ] Use switched/fused 12 V source
- [ ] Add reverse-polarity protection
- [ ] Add a TVS/transient clamp on the vehicle input
- [ ] Add local bulk + ceramic decoupling
- [ ] Verify 5 V rail during cranking
- [ ] Verify ESP32 survives starter/alternator electrical noise
- [ ] Log supply/brownout events

### CAN

- [ ] Connect CAN transceiver to ESP32 TWAI peripheral
- [ ] Start in listen-only mode
- [ ] Confirm BMW bus bitrate before connection
- [ ] Do **not** add another 120 ohm termination resistor to an already terminated in-car bus
- [ ] Capture raw BMW CAN frames to SD
- [ ] Decode RPM first as a common synchronization channel
- [ ] Decode pedal, throttle, coolant temperature and other useful broadcast values as found
- [ ] Keep aftermarket transmission disabled until there is a deliberate reason to transmit

### SD logging

Prefer a compact binary record while driving; convert to CSV/Parquet later.

Suggested record fields:

```text
timestamp_us
rpm
oil_pressure_psi
fuel_pressure_psi
map_kpa_abs
iat_c
crankcase_pressure
coolant_pressure
sensor_5v
pedal_pct
throttle_pct
flags
```

- [ ] Preallocate / rotate log files so one corrupt file does not lose a whole event
- [ ] Include configuration/calibration version in every file header
- [ ] Flush safely without calling sync on every sample
- [ ] Detect and flag SD write overruns
- [ ] Maintain sensor sampling even if SD writes momentarily stall
- [ ] Create Python converter: binary -> Parquet/CSV
- [ ] Create notebook loader/plotter

## Safety / alarms

### Oil pressure

The logger should eventually use an RPM-dependent warning envelope rather than one fixed pressure threshold.

- [ ] Baseline healthy hot-idle oil pressure
- [ ] Baseline pressure vs RPM with hot oil
- [ ] Define minimum acceptable pressure curve
- [ ] Implement low-pressure timer/debounce
- [ ] Drive a large red warning lamp / buzzer
- [ ] Prefer a separate hardwired catastrophic oil-pressure switch/alert as backup

### Fuel pressure

Derived injector differential pressure:

```text
DeltaP_injector = fuel_rail_absolute_or_gauge_adjusted - manifold_pressure
```

- [ ] Confirm exact rail-pressure sensor reference convention
- [ ] Calculate differential pressure correctly
- [ ] Alarm on unexpected loss of differential pressure under boost

### Boost / lambda

- [ ] Log independent MAP even if MSV70 later receives its own boost-capable MAP
- [ ] Integrate independent wideband later (issue #11)
- [ ] Add lean-under-boost detection once lambda data is available
- [ ] Do not make the ESP32 logger the sole overboost protection path

## Packaging / harness

- [ ] Use sealed connectors at engine-bay sensor branches
- [ ] Return analog sensor grounds to a common logger/sensor-ground scheme
- [ ] Avoid grounding each sensor at random chassis points
- [ ] Keep analog wiring away from coil primary wiring, starter cable, and alternator B+
- [ ] Add strain relief and abrasion protection
- [ ] Mount ESP32/SD electronics inside cabin or a protected enclosure, not beside the turbo
- [ ] Label every connector and channel
- [ ] Maintain a pinout in this document

## Wideband integration

Issue #11 tracks the independent wideband.

Two reasonable paths:

1. read a wideband controller's 0-5 V output through a protected ADC input; or
2. decode its CAN stream and avoid another analog channel.

CAN is preferred if the selected controller documents the protocol.

## Related project issues

- #11 — independent wideband O2
- #12 — fuel-pressure sensor and logging
- #13 — post-intercooler IAT
- #15 — MSV70 boost-capable manifold pressure sensor (separate from logger MAP)
- #21 — drift oil-control / baffled pan / oil-pressure logging

## Rev 2 — custom PCB gate

**Do not start this until Rev 1 has meaningful in-car logs.**

A custom PCB becomes worthwhile if the prototype proves useful but wiring/module reliability or packaging is poor.

Possible Rev 2 board:

- ESP32 module
- protected 12 V automotive input
- regulated 5 V sensor rail
- 3.3 V MCU rail
- CAN transceiver
- microSD
- 6-8 protected analog divider/filter channels
- thermistor channel(s)
- sensor-ground distribution
- warning-lamp / buzzer low-side output
- test points
- sealed harness connector(s)

- [ ] Freeze channel list from real Rev 1 usage
- [ ] Freeze connector family
- [ ] Measure actual sensor current requirements
- [ ] Capture cranking/load-dump/noise behavior
- [ ] Decide whether ESP32 ADC is good enough or add external ADC
- [ ] Only then draw schematic/PCB

## External ADC decision

Do not order one preemptively.

- [ ] Test ESP32 ADC after calibration + filtering + oversampling
- [ ] Quantify noise and nonlinearity in notebook
- [ ] If inadequate, evaluate MCP3008/ADS1115-class external ADC based on required sample rate
- [ ] Prefer SPI ADC for fast pressure/MAP channels if an external ADC becomes necessary

## Initial budget target

Excluding sensors, Rev 0/1 electronics should stay roughly in the **$40-$80** class rather than turning into a commercial $600+ DAQ.

The sensors, wiring and connectors remain useful if a custom PCB is built later.

## Definition of done

Rev 1 is successful when the car can produce one synchronized log containing at least:

- RPM
- oil pressure
- fuel pressure
- independent MAP
- post-IC IAT
- sensor supply voltage

and that log can be loaded reproducibly into the project notebook tooling, with an oil-pressure warning working independently of the analysis laptop.