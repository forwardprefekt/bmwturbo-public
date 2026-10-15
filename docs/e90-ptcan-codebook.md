# E90 PT-CAN codebook and Check Control findings

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Run it

The archived notebook uses Python 3.12 and the private archive’s CAN/notebook
dependencies. Host-specific setup notes are withheld. The examples below
require separately obtained tooling and inputs; they do not run from this
public documentation companion alone.

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
source .venv/bin/activate
jupyter lab analysis/notebooks/e90_ptcan_codebook.ipynb
```

The reusable tables can be rebuilt and byte-checked without opening Jupyter:

> Private-archive example only: separately obtain the referenced tools and
> rights-cleared inputs. They are withheld from this public edition; this block
> is not a runnable public-package workflow.

```bash
python scripts/export_e90_ptcan_codebook.py
python scripts/export_e90_ptcan_codebook.py --check
```

For source lookup without reopening the research clones, begin at
`references/can/e90-source-quick-reference.md` (supporting artifact withheld from public edition). The hash-bound local inventory
is `references/can/upstream/sources.jsonl` (supporting artifact withheld from public edition); all currently retrievable supporting
sources and their licenses/handling notices live below
`references/can/upstream/` (supporting artifact withheld from public edition). No-license and copyrighted snapshots remain private
research inputs, and the invalid mega-DBC remains inert rather than loadable.

## What the retained trace establishes

The immutable BLF contains 488,526 standard frames across 61 arbitration IDs.
All 61 IDs now have a message-level identity. That is not an assertion that
every signal is decoded:

- 11 IDs have exact-DLC community signal profiles, all labeled
  `community_unverified`;
- 43 canonical-DBC messages have an identity but no signal layout;
- 3 DBC messages have no exact-DLC frames in this capture;
- 4 IDs absent from the canonical DBC now have independently sourced message
  identities only.

The four prior DBC gaps are:

| ID | Evidence-backed identity | Current signal status |
|---|---|---|
| `0x49C` | LDM network management | message identity only |
| `0x4F2` | KBM network management | message identity only |
| `0x59C` | LDM services | message identity only |
| `0x5F8` | IHKA services | message identity only |

## The observed `0x592` warnings

An independent E90 cluster implementation sends `0x592` Check Control display
requests with a little-endian ID in bytes 1-2 and `0x31`/`0x30` in byte 3 for
active/inactive. The retained payloads fit that layout exactly. BMW's E90
driver-information table resolves both observed IDs:

| Payload subset | Check Control ID | State | Meaning |
|---|---:|---|---|
| `40 1C 00 31 ...` | 28 | active | Engine oil below minimum; top up 1 litre |
| `40 1C 00 30 ...` | 28 | inactive | same display request cleared |
| `40 22 00 31 ...` | 34 | active | Fixed yellow engine-shaped warning lamp; no text listed |
| `40 22 00 30 ...` | 34 | inactive | same lamp request cleared |

The first ID-28 active frame occurs at relative 20.1171 s and its first
inactive frame at 284.3934 s. ID 34 first becomes active at 283.8937 s and
inactive at 320.2497 s. Timing proximity is retained as an observation, not a
claim that one warning caused the other.

These are cluster display requests, not stored DME diagnostic trouble codes.
The trace contains no standard `0x7DF`/`0x7E0..0x7EF` OBD/UDS frames and none
of the pinned BMW `0x6xx` diagnostic IDs. The underlying root-cause fault
memory therefore cannot be read from this BLF.

## Stronger signal candidates

Source-backed formulas live separately from the canonical DBC profiles:

- Three independent implementations agree that `0x0AA` bytes 4-5 are
  little-endian engine speed with factor `0.25 rpm/bit`. The retained trace is
  0-2907 rpm with a 667 rpm median. This is substantially more plausible than
  the canonical DBC's 0-11628 result, but still needs a tach-referenced marked
  capture before project-car confirmation.
- Two independent implementations agree that `0x1D0` byte 0 minus 48 is
  coolant temperature. The observed candidate range is 29-69 degC.
- One implementation proposes `0x0AA` byte 3 times 0.3937 as accelerator
  position. It remains single-source and unvalidated.
- Two sources agree that `0x3B4` carries battery voltage but disagree on the
  exact scale. The repeated `A5 F3` payload is 13.995 V using low-12-bit
  factor 0.015 and 13.721 V using `(u16le - 0xF000) / 68`. Both remain in the
  output as `source_conflict_scale`; a synchronized multimeter value should
  decide between them.

## Generated outputs

`analysis/outputs/e90_ptcan_codebook/` (supporting artifact withheld from public edition) contains:

- `message_codebook.csv`: one evidence/status row per captured ID;
- `community_signal_profiles.csv`: observed ranges from exact-DLC canonical
  DBC decodes;
- `corroborated_signal_candidates.csv`: independently sourced candidate
  formulas, agreement/conflict, and observed ranges;
- `raw_byte_activity.csv`: entropy and change rate per byte;
- `rolling_counter_candidates.csv`: raw increment-pattern candidates only;
- `service_payloads.csv`: unique opaque `Services*` payload inventory;
- `check_control_events.csv`: every decoded `0x592` display-request frame;
- `diagnostic_summary.jsonl`: machine-readable diagnostic boundary/result.

## OEM diagnostic software boundary

The earlier filename-only search found only the retained `S7623406.0da`
calibration. The source-host upload now adds exact `MSV70.prg`, `EDIABAS.INI`,
and `OBD.INI` inputs, not a complete EDIABAS/INPA/ISTA corpus. Static decoding
recovers all 331 direct MSV70 jobs, including identity, grouped F0 measurement,
and generic memory-read contracts; see the
`host-input summary` (supporting artifact withheld from public edition).
Those definitions support a reviewed DME diagnostic probe but are not a
broadcast-CAN signal database and do not identify the transport frames in the
existing passive capture.

For the next vehicle session, run the canonical identity gate and reviewed
stationary F0 probe, then capture the diagnostic bus and the tool's raw
read-only report together before clearing faults, coding modules, or
transmitting other exploratory frames.
