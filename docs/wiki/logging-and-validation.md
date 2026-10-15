# Separate logs from accepted validation

[Wiki home](README.md) | [Evidence rules](evidence-and-tools.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page explains what the retained CAN capture establishes, what internal DME logging still needs to prove, and how physical measurements must relate to both. A decoded message, a successful transport call, and an accepted synchronized experiment are different results.

## Current validation status

The [dynamic-validation protocol](../msv70-dynamic-validation-protocol.md) defines 24 planned observations. Their source observation records (supporting artifact withheld from public edition) remain `not_run`; the generated worksheet and example manifests are not vehicle-validation evidence.

| Evidence layer | Current boundary |
|---|---|
| Native-code analysis | Establishes reviewed operations, selectors, widths, and conditional paths |
| Retained passive CAN recording | Establishes captured frames and source-qualified interpretations |
| Diagnostic job definitions | Establishes available static job descriptions, not successful live ECU access |
| Read-only RAM transport | Requires retained identity, raw results, timing, and focused-state coverage |
| Synchronized experiment | Requires calibrated independent instruments, measured time alignment, and capture-bound outcomes |
| Boosted operating envelope | Not established by the preceding layers or by a passing software check |

Use the [memory-capture runbook](../msv70-memory-capture-runbook.md) for the operator sequence. This wiki does not replace that procedure with a shorter, less constrained one.

## What the passive CAN capture contains

The [E90 PT-CAN codebook](../e90-ptcan-codebook.md) describes the retained 500 kbit/s baseline recording and its decoding limits:

| Retained result | Interpretation |
|---|---|
| 488,526 standard frames across 61 arbitration IDs | Observed traffic in that recording, not every message the vehicle can transmit |
| Message identities for all 61 IDs | Message identification does not imply a verified signal layout |
| Exact-DLC community signal profiles for 11 IDs | Profiles remain `community_unverified` |
| `0x592` Check Control display requests | Cluster warning requests, not stored DME fault codes |
| No reviewed diagnostic request/response traffic in the capture | The BLF cannot reveal the underlying DME fault memory |

RPM, coolant, accelerator, and battery-voltage candidates have different corroboration levels. The battery-voltage sources disagree on scaling. Retain that conflict until a synchronized reference measurement resolves it; do not choose the more plausible-looking number.

The [CAN baseline runbook](../msv70-can-baseline-capture-runbook.md) defines stationary recordings and metadata. Keep passive observation separate from transmitting diagnostic or exploratory traffic.

### Read the decoder’s acceptance boundary

This **repository-authored analysis-tool Python excerpt (not firmware source)** from `decode_matching_frames`, lines 405–426 (supporting artifact withheld from public edition), requires both a known arbitration ID and the expected payload length:

```python
def decode_matching_frames(frames: pd.DataFrame, database: Any) -> pd.DataFrame:
    """Decode only exact ID/DLC matches and retain an unverified status."""
    messages = {int(message.frame_id): message for message in database.messages}
    rows = []
    for frame in frames.itertuples(index=False):
        message = messages.get(int(frame.arbitration_id))
        if message is None or int(frame.dlc) != int(message.length):
            continue
        decoded = message.decode(
            bytes.fromhex(frame.data_hex), decode_choices=False, scaling=True
        )
        rows.append(
            {
                "session_time_s": float(frame.session_time_s),
                "arbitration_id": int(frame.arbitration_id),
                "id_hex": frame.id_hex,
                "dbc_message": message.name,
                "decode_status": "community_unverified",
                **decoded,
            }
        )
    return pd.DataFrame(rows)
```

The function applies the database’s scaling but labels decoded rows `community_unverified`. Rejecting a mismatched length prevents use of that message layout; accepting a length does not validate its units or physical meaning. This is analysis-tool source, not DME firmware or a live capture result.

## First internal-state capture

The immediate firmware question requires a grouped or event-driven capture with known timing. Reading states serially without timing evidence can combine values from different updates.

The [17-state acquisition gate](../msv70-dynamic-validation-protocol.md#dme-acquisition-implementation-gate) groups the following `r13` displacements:

| Group | Required states |
|---|---|
| Availability, eligibility, and update gate | `1e8c`, `1f62`, `1fea` |
| Airflow candidates and selected state | `2ac2`, `2b26`, `2b30` |
| RPM coordinate | `6984` |
| Charge, load, intermediate, correction, denominator | `2ada`, `6ab6`, `6abe`, `6ade`, `6b62`, `6b68` |
| Modeled filling candidates and conversion | `6b5a`, `6b6a`, `6b6e`, `6b9a` |

The protocol contains exact absolute addresses and access widths. The unit/type registry (supporting artifact withheld from public edition) preserves consumer-dependent signedness. Do not interpret every halfword using one global signed type.

The denominator and correction states are necessary to evaluate the load equation. The selector and update gates are necessary to distinguish newly acquired values from feedback, substitutes, or retained state.

## Establish time and channel provenance

The intended clock contract is host-bracketed affine alignment, not an assumption that DME, CAN, host, and sensor-logger timestamps already agree. Preserve the synchronization exchanges, raw device clocks, host brackets, fitted mapping, residuals, uncertainty, and gaps described in the [timebase protocol](../msv70-dynamic-validation-protocol.md#host-bracketed-affine-timebase).

Each accepted channel needs its source field, units, calibration, timestamp basis, and fault/status history. Independent measurements include absolute manifold pressure, ambient pressure, airflow, post-intercooler intake-air temperature, RPM, and wideband lambda. Injector validation also needs rail pressure and injector-terminal voltage; use compatible pressure references before subtracting rail and manifold values.

The [ESP32 logger design](../esp32-sensor-logger.md) is an instrumentation plan. Proposed channel rates are not measured performance, and the logger is not the sole safety system.

## Decide an observation’s outcome

The [protocol's predeclared outcomes](../msv70-dynamic-validation-protocol.md#predeclared-outcomes) are `support`, `contradict`, `unresolved`, and `not_evaluated`. The separate execution status `not_run` records that an observation has not been performed. Do not substitute execution status for an evaluated verdict.

Before accepting support, review the raw files as well as tool output. Confirm identity, complete state/channel coverage, finite usable samples, calibration, valid fault status, and time alignment. Missing or contradictory evidence must remain visible; a validator’s success status alone does not establish physical correctness.

Keep naturally aspirated baseline observations, protected bench characterization, and any proposed altered-operating-envelope experiment separate. The runbooks’ setpoints are procedure defaults, not newly discovered safe engine limits. No page here authorizes fault injection, protection suppression, or boosted testing.

## Current unknowns

The project still needs a demonstrated read-only transport, measured update rates, complete logger configuration, accepted clock alignment, and retained synchronized observations. The static dossiers cannot determine live branch histories or bind every RAM candidate to a physical measurement.

Related pages: [Airflow and load](airflow-and-load.md), [fuel and injection](fuel-and-injection.md), [hardware and recovery](hardware-and-recovery.md), and [open questions](open-questions.md).
