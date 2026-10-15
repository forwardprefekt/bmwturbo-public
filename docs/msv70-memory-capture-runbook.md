# MSV70 read-only memory capture runbook

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

## Decision

The next vehicle session can test the diagnostic path through a real, bounded
sequence:

1. exact ECU identity;
2. the documented grouped F0 measurement job;
3. one repeated two-byte `LAR` read from canonical SRAM;
4. all 17 focused states as exact-width reads and as seven compact reads; and
5. aggregate throughput at stable warm idle.

The source-bound plan is
`analysis/validation/msv70-memory-probe-plan.json` (supporting artifact withheld from public edition).
The executable runner is
`scripts/msv70_memory_probe.py` (supporting artifact withheld from public edition).
Both are testable without a vehicle; live success remains `not_run` until the
vehicle session occurs.

This first session is a **transport probe**, not an accepted issue-#39 capture.
Unmodified `EdiabasTest.exe` executes multiple jobs in one persistent EDIABAS
session, but its output does not bracket each `ExecuteJob` call with a host
monotonic timestamp. The runner brackets each complete process invocation. That
is enough to prove access and aggregate rate; it is not enough to claim
per-sample skew, update rate, or cross-device clock alignment.

## What is already exact

### Source binding

- Diagnostic file: `references/ediabas/ECU/MSV70.prg` (supporting artifact withheld from public edition)
- SHA-256:
  `3772fdc18259939ef732c1ce42b5b2cecc56a6d5ea65746a9b503cb68bd5212a`
- Static catalog: 331 jobs, 82 tables, 465 `MESSWERTETAB` rows, and no `USES`
  dependency on a companion `.GRP`.
- The installed configuration is `STD:OBD`; retained `OBD.INI` names `COM1`.
  That assignment is only a host configuration fact. Use the port of the
  verified connected diagnostic cable.

### Mandatory identity gate

Run these first, in order:

| Order | Job | Required result |
|---:|---|---|
| 1 | `IDENT_AIF` | retain all current/historical AIF identity sets |
| 2 | `SERIENNUMMER_LESEN` | `SERIENNUMMER == locally_recorded_expected_serial` |
| 3 | `DATEN_REFERENZ_LESEN` | `DATEN_REFERENZ == 0049PP0L921S0AC1S` |
| 4 | `ZIF_LESEN` | `ZIF_PROGRAMM_REFERENZ == 0049PP0L921S` |
| 5 | `HARDWARE_REFERENZ_LESEN` | retain the hardware reference |
| 6 | `SPEICHER_LESEN_ASCII` | only after orders 1–5 pass; `LAR;0x840008;8`, requiring `DATEN_ASCII == 9PP92L00` |

Any mismatch ends the session. A matching filename is not an identity result.
The runner does not execute order 6 after a serial, DREF, program-reference, or
required-field failure.

### Standard grouped probe

The exact first grouped job is:

```text
STATUS_MESSWERTBLOCK_LESEN
```

Define and read F0 once:

```text
3;0x4807;0x4200;0x4201;0x4202;0x4203;0x4205;0x4600;0x5A0C
```

Reuse and read the defined block:

```text
2;0x4807;0x4200;0x4201;0x4202;0x4203;0x4205;0x4600;0x5A0C
```

The eight documented channels are RPM, intake temperature, ambient pressure,
manifold pressure, HFM mass flow, manifold/boost pressure 1, throttle angle,
and MAF voltage. Defining F0 clears and defines an ephemeral diagnostic-session
local identifier. It does not write calibration, adaptation, or flash, but it
is not a zero-state-change operation. Do not represent it as one.

### Exact memory-read contract

`SPEICHER_LESEN` is the PRG's default-mode KWP2000 `$23
ReadMemoryByAddress` job:

```text
SEGMENT;ADRESSE;ANZAHL
```

- `SEGMENT`: a row from `SpeicherSegment`;
- `ADRESSE`: `0x000000..0xFFFFFF`;
- `ANZAHL`: `1..254` bytes;
- results: `DATEN`, `JOB_STATUS`, `_TEL_AUFTRAG`, `_TEL_ANTWORT`.

The first live SRAM attempt uses `LAR` and the absolute 24-bit address. This is
the only justified first attempt: the canonical EPK proof already uses `LAR`
with an absolute address, and `LAR` is the table's linear-address-range entry.
Do not cycle through `RAMIS`, `RAMXX`, or other segment names until one appears
to work. If `LAR;0x7FE984;2` is rejected, retain the exact job status and raw
negative response, then stop.

PowerPC storage is big-endian. Preserve `DATEN` unchanged. The probe output
exposes both signed and unsigned big-endian integers for inspection, but it does
not assign a global signedness or an engineering unit to shared storage.

## Concrete prerequisites for the next vehicle session

All four physical prerequisites are hard gates:

1. **Known diagnostic interface.** A verified FTDI K+DCAN/K-line-capable cable,
   including its USB hardware ID, chipset, unique serial if available, driver,
   7/8 switch or bridge state, and actual COM port. The inventoried CP210x on
   COM5 is not evidence of K+DCAN compatibility.
2. **Stable vehicle power.** Battery support appropriate for extended KOEO use,
   with voltage observed before, during, and after the probe. This document does
   not invent a numeric voltage threshold; any low or unstable condition is an
   abort.
3. **Exact software.** The canonical PRG must hash exactly. Locate the retained
   EdiabasLib `EdiabasTest.exe` bundle, keep its dependencies together, and hash
   the executable. Do not substitute an untracked download during the session.
4. **Fault baseline.** Identify the DME and read faults before starting without
   clearing faults or adaptations. Save the native output. Any existing fault
   relevant to power, communication, immobilizer, throttle, MAF, pressure, or
   engine operation must be understood before proceeding.

Also record:

- vehicle naturally aspirated configuration and any deviation from the last
  retained baseline;
- fuel grade;
- ambient conditions and approximate coolant state;
- cable, charger, laptop, EDIABAS, and EdiabasTest versions/hashes;
- DME identity and current calibration/full-read hash if available;
- exact start/end UTC and an operator event log; and
- all pre- and post-session DTCs, including an explicit statement that nothing
  was cleared.

Close INPA and Tool32 before invoking EdiabasTest so they do not retain the COM
port. Do not run Tool32 and the runner concurrently.

## Windows preparation

From PowerShell in the repository clone:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
python .\scripts\msv70_memory_probe.py verify-plan --repo-root .
python -m pytest .\tests\test_msv70_memory_probe.py -q
```

Those checks exercise deterministic planning and a synthetic EdiabasTest
process. They do not exercise the cable or DME.

Locate the retained executable and inspect serial ports:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
Get-ChildItem "$env:USERPROFILE\Downloads" -Filter EdiabasTest.exe -File -Recurse |
  Select-Object FullName,Length,LastWriteTime
Get-CimInstance Win32_SerialPort |
  Select-Object DeviceID,Name,PNPDeviceID
```

After selecting the exact files and verified cable port:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
$Probe = (Resolve-Path .\scripts\msv70_memory_probe.py).Path
$Prg = 'C:\EDIABAS\ECU\MSV70.prg'
$EdiabasTest = 'C:\path\to\retained\EdiabasTest.exe'
$Port = 'COM1' # replace with the verified cable port
$Session = 'C:\MSV70-Captures\YYYYMMDD-read-only-probe-r1'

Get-FileHash -Algorithm SHA256 $Prg
Get-FileHash -Algorithm SHA256 $EdiabasTest
```

The runner refuses to create the session if the PRG hash differs. Use a new
session directory for every retry; never delete or overwrite a failed attempt.

## Next-session execution sequence

At every step: vehicle secure and stationary, ventilation safe, no intentional
positive pressure, no programming session, no security unlock, no writes, no
actuator jobs, no adaptation clear, and no fault suppression. Run only the next
stage after the preceding `*-gate.json` says `"status": "passed"`.

### Stage 0 — physical and diagnostic preflight

1. Connect stable battery support.
2. Record cable hardware ID, driver, switch/bridge state, and COM port.
3. Use INPA or Tool32 to identify the MSV70 and read DTCs without clearing.
4. Save the identity/DTC evidence, then close the application.
5. Run `verify-plan` again on the vehicle laptop.

**Pass:** the verified cable communicates, power is stable, the DME is the
expected family, pre-fault evidence is retained, and the exact files are
hash-bound.

**Blocker labels:** `no_verified_interface`, `unknown_com_port`,
`unstable_power`, `dme_not_identified`, `preexisting_fault_unresolved`,
`prg_hash_mismatch`, or `ediabas_test_missing`.

### Stage 1 — exact identity

Ignition on, engine off, stable power:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
python $Probe run --repo-root . --session-dir $Session `
  --ediabas-test $EdiabasTest --prg $Prg --port $Port `
  --stage identity --operator-state KOEO_STABLE_POWER --confirm-read-only
```

Inspect `$Session\identity-gate.json`. Do not continue on an error. The runner
records the EdiabasTest, sibling `EdiabasLib.dll` when present, runner, plan, and
PRG hashes; raw CP-1252 output; UTF-8 renderings; parsed result sets; exact
arguments; process timing; and IFH trace when EdiabasLib produces it.

### Stage 2 — grouped standard channels

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
python $Probe run --repo-root . --session-dir $Session `
  --ediabas-test $EdiabasTest --prg $Prg --port $Port `
  --stage standard --operator-state KOEO_STABLE_POWER --confirm-read-only
```

This defines/reads F0 once and reuses it four times. Every call must return
`JOB_STATUS == OKAY`, all eight documented result fields, and raw request and
response telegrams.

**Stop** for any negative response, missing result, communication retry storm,
instrument-cluster warning, DME reset, or power disturbance.

### Stage 3 — minimal SRAM access proof

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
python $Probe run --repo-root . --session-dir $Session `
  --ediabas-test $EdiabasTest --prg $Prg --port $Port `
  --stage ram-minimal --operator-state KOEO_STABLE_POWER --confirm-read-only
```

This sends five identical jobs in one EdiabasTest session:

```text
SPEICHER_LESEN#LAR;0x7FE984;2
```

Each must return `OKAY`, exactly two `DATEN` bytes, one raw request, and one raw
response. Repetition proves deterministic access to an address-sized result; it
does not yet prove the RPM interpretation or an update rate.

After this stage, close diagnostics cleanly, cycle only as required by the
normal tool procedure, and re-read DTCs without clearing. Any new DME, bus,
immobilizer, throttle, MAF, or communication fault blocks Stage 4.

### Stage 4 — focused individual and compact reads

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
python $Probe run --repo-root . --session-dir $Session `
  --ediabas-test $EdiabasTest --prg $Prg --port $Port `
  --stage ram-focused --operator-state KOEO_STABLE_POWER --confirm-read-only
```

The runner first performs 17 exact-width reads: 17 requests and 31 payload
bytes. It then performs this compact seven-request, 225-byte layout:

| Range | Length | Extracted focused states |
|---|---:|---|
| `0x007f9e8c` | 1 | `r13+1e8c` |
| `0x007f9f62` | 1 | `r13+1f62` |
| `0x007f9fea` | 1 | `r13+1fea` |
| `0x007faac2` | 112 | `2ac2@0`, `2ada@24`, `2b26@100`, `2b30@110` |
| `0x007fe984` | 2 | `6984@0` |
| `0x007feab6` | 42 | `6ab6@0`, `6abe@8`, `6ade@40` |
| `0x007feb5a` | 66 | `6b5a@0`, `6b62@8`, `6b68@14`, `6b6a@16`, `6b6e@20`, `6b9a@64` |

A compact response makes only the values inside that one contiguous range
coherent at the response level. The seven requests are sequential. Do not call
their extracted values one same-cycle snapshot without a measured inter-job
skew bound.

### Stage 5 — warm-idle sanity

Only after a normal warm-up and a stable, mechanically normal idle:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
python $Probe run --repo-root . --session-dir $Session `
  --ediabas-test $EdiabasTest --prg $Prg --port $Port `
  --stage idle-sanity --operator-state WARM_IDLE_STABLE --confirm-read-only
```

This runs three grouped standard reads, three individual focused cycles, and
three compact cycles. It can show whether values are repeatable and respond
plausibly between KOEO and idle. It cannot establish a firmware relationship.
Record every warning, unstable behavior, or operator intervention.

### Stage 6 — aggregate throughput comparison

At the same stable warm idle:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```powershell
python $Probe run --repo-root . --session-dir $Session `
  --ediabas-test $EdiabasTest --prg $Prg --port $Port `
  --stage benchmark --operator-state WARM_IDLE_STABLE --confirm-read-only
```

The three invocations are:

- 25 minimal two-byte requests;
- 10 cycles of 14 individual reads; and
- 10 cycles of seven compact reads.

The stage reports whole-invocation duration and aggregate jobs/s. It does **not**
report a per-job median, p95, maximum, or skew because EdiabasTest supplies no
per-job monotonic brackets. Do not derive those statistics by dividing the
aggregate duration.

Finish with a DTC read, no clearing, and a power/vehicle-state note. Hash the
complete session directory and copy it to separate storage before analysis.

## Day-1 success ladder

| Level | Evidence required | What it permits next |
|---:|---|---|
| 0 | exact files, verified cable/port, stable power, DTC baseline | identity only |
| 1 | exact serial, DREF, ZIF reference, and EPK | grouped standard probe |
| 2 | F0 define/read plus four reuse reads, all raw and `OKAY` | one minimal RAM address |
| 3 | five exact two-byte `LAR` reads at `0x007fe984` | focused exact-width reads |
| 4 | 14 individual and seven compact requests with exact lengths | warm-idle sanity |
| 5 | repeatable aggregate rate with no transport/vehicle fault | implement the persistent timestamped logger |

Level 5 still does not close issue #66 or satisfy issue #39. It proves that the
diagnostic path merits logger implementation.

## Raw evidence contract

A probe session is an immutable directory. The runner creates:

```text
session.json
identity-core/
identity-epk/
identity-gate.json
standard-f0/
standard-gate.json
ram-minimal-rpm/
ram-minimal-gate.json
ram-focused-individual/
ram-focused-compact/
ram-focused-gate.json
idle-standard-f0/
idle-individual/
idle-compact/
idle-sanity-gate.json
benchmark-minimal/
benchmark-individual/
benchmark-compact/
benchmark-gate.json
```

Each invocation directory retains:

- `command.json`: exact argv and ordered job metadata;
- `stdout.bin` and `stderr.bin`: untouched process bytes;
- `stdout.txt` and `stderr.txt`: UTF-8 inspection copies;
- `parsed-results.json`: ordered `JOB`/`DATASET` projection;
- `samples.jsonl` for memory jobs: address, length, raw bytes, raw telegram text,
  extraction offsets, both big-endian integer views, and the shared invocation
  bracket;
- `ifh.trc` when emitted; and
- `invocation.json`: process result, monotonic/UTC bracket, aggregate rate, and
  SHA-256/size for every retained invocation artifact.

Keep separate operator evidence beside the session:

- pre/post DTC native output;
- cable hardware-ID and driver export;
- battery-support and observed-voltage log;
- vehicle/configuration/fuel/ambient notes;
- an event log with UTC and monotonic host markers when available; and
- a final recursive SHA-256 manifest and hash of the archive copied to separate
  storage.

A screenshot may supplement this evidence. It cannot replace raw output,
telegram bytes, or hashes.

## Failure classification and next action

| First failure | Classification | Required action |
|---|---|---|
| Verified cable never appears or INPA cannot identify DME | physical interface/configuration blocker | resolve cable, driver, bridge/switch, port, or power; do not try RAM jobs |
| INPA works but EdiabasTest cannot load/communicate | EdiabasLib packaging/configuration blocker | retain stdout/stderr, executable/DLL hashes, and IFH trace; resolve dependencies or port ownership |
| Identity field differs | identity blocker | stop; do not read EPK or RAM; reconcile exact DME/software |
| F0 define/read fails | grouped-job/session blocker | retain request/response; do not proceed to `$23` RAM |
| `ERROR_SEGMENT`, `ERROR_ADDRESS`, or `ERROR_NUMBER` | PRG argument-contract failure | retain exact argv/output; correct only against static PRG evidence, never by segment guessing |
| `$7F 23` negative response | live memory-service blocker | retain NRC and telegram; stop on unsupported service, conditions, range, or security response; do not unlock |
| Two bytes return but do not behave plausibly across KOEO/idle | address/semantic blocker | preserve contradiction; do not move addresses until static/live evidence explains it |
| Individual reads work, compact range fails | range/length/transport blocker | retain exact first failing range; keep individual mode for characterization |
| Reads work but disturb DME/bus or create faults | safety blocker | stop diagnostic RAM work and retain post-fault evidence |
| Reads are stable but aggregate rate is inadequate | throughput blocker | evaluate observation-specific bundles and PRG-native blocks before any firmware mutation |
| Aggregate rate is good | known timestamp blocker remains | implement the persistent in-process logger below |

### Tool32 fallback

If the verified cable and Tool32 work but `EdiabasTest.exe` cannot be made
available during the session, Tool32 can still provide Levels 1–3 manually:

1. load the exact hash-bound `MSV70.prg`;
2. execute the five identity jobs and save every result set;
3. only after the exact DREF match, run
   `SPEICHER_LESEN_ASCII` with `LAR;0x840008;8`;
4. run the F0 define/read argument once and its reuse argument four times; and
5. run `SPEICHER_LESEN` with `LAR;0x7FE984;2` five times.

Save Tool32 result panes and native trace/output. Manual Tool32 results can prove
access. They cannot supply the automated aggregate benchmark or issue-39 timing.
Do not use the fallback to try guessed segment/address combinations.

## Persistent logger required after a successful probe

Do not extend the wrapper until it resembles a logger. Replace EdiabasTest with
a small in-process EdiabasLib executable that owns one `EdiabasNet` instance and
one resolved, hash-bound PRG for the complete run.

For every `ExecuteJob` call it must record:

- immutable session and sample indices;
- job, segment, address, requested length, and layout/range ID;
- host monotonic request-start and response-end ticks plus clock frequency;
- UTC correlation records, midpoint estimate, and half-bracket uncertainty;
- raw `DATEN`, `_TEL_AUFTRAG`, `_TEL_ANTWORT`, `JOB_STATUS`, and exception/error
  code;
- cycle ordinal, request ordinal in cycle, retry count, and dropped/failed state;
- DME serial, DREF, ZIF, EPK, PRG hash, logger hash, EdiabasLib hash, interface,
  and cable identity; and
- the unmodified IFH trace as secondary transport evidence.

The host bracket around each call—not IFH file write time and not a simultaneous
throttle blip—is the DME-stream timebase observation. A fixed request order must
be retained. A separate forward/reverse-order timing characterization should
measure systematic order skew before selecting the production order.

The logger must compile every job from the checked plan, reject any unlisted job
or range, and have no general pass-through job interface in capture mode.

## Focused acquisition schedule after logger implementation

### Benchmark modes

Measure, rather than assume, all three:

1. **Minimal:** one two-byte `0x007fe984` request.
2. **Individual:** 17 requests / 31 payload bytes.
3. **Compact:** seven requests / 225 payload bytes.

For each mode at KOEO and stable warm idle retain at least enough consecutive
cycles to report median, p95, and maximum call/cycle duration; achieved cycles/s;
failures/retries; maximum request-to-response bracket; and DME/bus symptoms. The
sample count is selected before the executed benchmark and recorded in its
configuration. Do not manufacture latency percentiles from aggregate timing.

### Observation-specific bundles

If seven compact requests cannot meet the measured skew bound, use smaller
bundles tied to one declared question:

| Bundle | Requests | States |
|---|---:|---|
| Airflow selectors | 4 | singleton `1e8c`, `1f62`, `1fea`; airflow block containing `2ac2`, `2ada`, `2b26`, `2b30` |
| Filling selection | 3 | RPM block; load block containing `6ab6`, `6abe`, `6ade`; filling block containing `6b5a`, `6b62`, `6b68`, `6b6a`, `6b6e`, `6b9a` |
| Modeled-load selection (`issue34-modeled-load-selection`) | 4 | airflow block; RPM block; load block containing `6ab6`, `6abe`, `6ade`; filling block containing `6b62`, `6b68` and the filling candidates/outputs |

Use the exact range IDs and extraction widths from the checked plan's
`post_probe_capture_plan.observation_bundles`; dropping `load-block` removes
the normalized/corrected load evidence required by modeled-load selection.

Define F0 once per diagnostic session. Interleave one F0 context read at a
predeclared cadence; record its location in the cycle. Do not silently change
that cadence between repeats.

### Update-rate characterization

Poll rate is not firmware update rate. For each retained state:

1. record a stationary quiet window;
2. record a controlled scenario expected to exercise the state;
3. retain every raw transition and repeated value;
4. estimate observed transition intervals and censor constant/no-transition
   windows as unresolved rather than zero-rate; and
5. calculate the shortest relevant observed event period.

Issue #39 requires total alignment/skew uncertainty no greater than one quarter
of that shortest relevant period. If the bound fails, narrow the bundle or leave
the observation unresolved.

### Vehicle scenarios

Reuse the canonical scenario catalog; do not create another procedure source of
truth:

- `NA-01`: key state and bus wake;
- `NA-02`: KOEO pedal sweep and steps;
- `NA-03`: cold start and untouched idle;
- `NA-04`: warm idle accessory-load steps;
- `NA-05`: repeated 1200/1800/2500-rpm holds and bounded transients; and
- optional `NA-06` hot restart only after the required stationary set passes.

The exact durations, repeats, quiet lead/trail, aborts, and operator defaults are
in
`analysis/validation/msv70-can-capture-scenarios.json` (supporting artifact withheld from public edition).
Run each scenario as separate raw artifacts while the RAM logger, grouped DME
channels, passive PT-CAN, and independent DAQ retain independent clocks. Do not
run optional road/load work until stationary acquisition, alignment, and
instrumentation pass.

### Accepted-capture boundary

Only promote data into issue #39 after all of these exist:

- per-job monotonic brackets from the persistent logger;
- measured transport latency/skew and per-state update behavior;
- at least 20 usable clock anchors and a passing affine timebase fit;
- calibrated independent channels and fault/status records;
- raw DME, PT-CAN, and external-DAQ artifacts with hashes;
- scenario/operator event records;
- a capture manifest that validates through the existing issue-39 schema and
  semantic validator; and
- support, contradiction, or unresolved outcomes computed without creating a
  causal relationship from correlation.

Until then, probe results remain read-only transport evidence. They do not
authorize a flash, altered sensor topology, intentional positive pressure, or a
firmware mirror/logger patch.
