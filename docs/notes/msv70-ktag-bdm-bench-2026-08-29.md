# MSV70 KTAG / BDM bench notes — 2026-08-29

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Status: **bench preparation only; no ECU write performed**.

This note records the physical ECU opening, BDM-pad identification, adapter-kit
identification, continuity observation, and the source-backed procedure we want
to follow before using KTAG on the donor MSV70.

## Goal

Use the donor MSV70 as a recoverable development ECU and obtain a verified,
immutable low-level backup before any cloning, identity work, or executable-code
patch testing.

The desired first result is not a tune. It is a reproducible set of original
reads with tool/mode metadata and hashes.

## Project hardware observed on the bench

### ECU

- Siemens/VDO MSV70 case opened from the PCB side.
- The exposed board layout visually matches the MSV70 family shown in the EVC
  BDM documentation.
- A 2 x 5 array of ten elongated solder pads was located near the center of the
  board. EVC documents these as the required MSV70 BDM pads.
- The small two-pad BDM-enable jumper immediately adjacent to that region was
  identified by comparing the board to the EVC MSV70 connection diagram.
- **No programming operation has been performed as part of these notes.**

### KTAG / KSuite setup

Previously inventoried on the BMW tooling machine:

- KSuite 2.80.
- Alientech USB driver package present.
- KSuite contains BMW/MSV70 protocol/help material.

Physically confirmed during this bench session:

- KTAG hardware described by the owner as a Master unit.
- Positioning-frame adapter kit present.
- Large adapter boards include both JTAG/Nexus and BDM Motorola MPC5xx families.
- The relevant base-board family is the **BDM Motorola MPC5xx** group, not the
  similarly shaped JTAG/Nexus MPC5xx board.
- Small pull-out tip marked **`16`** is present. Its PCB also carries an explicit
  center-pad **pin-1 (`1`) marking**.

### Adapter identification

Alientech's K-TAG manual lists the BDM Motorola MPC5xx accessory family as
`14KTMA0001` and identifies:

- `14AM00TBAS` — multifunction positioning-frame adapter for 1.27 mm pull-out
  tips;
- `14AM00TB01` — Siemens pull-out tip;
- `14AM00TB02` — Marelli pull-out tip;
- `14AM00TB03` — Bosch EDC7 pull-out tip.

The project kit's tip marked `16` is being treated as the Siemens-positioning
probe candidate because its physical form and markings match the Siemens
pull-out-tip arrangement used with the Motorola BDM base. **Before ECU power is
applied, KSuite's selected-family connection sheet should be used as the final
adapter/orientation authority.**

The rainbow direct-wire cables remain useful for other configurations, but we
should not bypass the positioning-frame adapter merely because a direct cable
is available.

## Authoritative MSV70 BDM facts

EVC's BDM100 documentation explicitly lists Siemens **MSV70 / MSS70** with:

- processor: **MPC563**;
- external flash: **AM29BDD160**;
- serial EEPROM: **ST95320**;
- probe: **BDM141**;
- external power required.

For MSV70 specifically, EVC states that:

1. ECU power is supplied through the **main ECU connector**.
2. Two PCB solder pads must be bridged with a solder drop or a **0-ohm
   resistor** so the BDM tool can recognize the ECU.
3. The BDM141 spring contacts are centered on the ten BDM pads.
4. At least roughly **1 mm spring travel** is required for reliable contact.
5. The positioning-frame arm must be lowered carefully to avoid shorts against
   components on the exposed board.

This is the best source currently in the project for the physical MSV70 BDM
connection because it names the ECU family and shows the board/jumper directly.

## Bench observation: continuity / ground check

With the ECU **unpowered**, the owner continuity-tested the ten elongated BDM
pads to ground.

Using the temporary pad numbering established during the chat session, pads
**6 and 8** showed continuity to ground.

Important limitation: this `6` / `8` numbering was a **temporary visual grid**,
not a factory or EVC pin designation. Preserve the measurement, but do not turn
it into an authoritative BDM pin map until the pad orientation has been checked
against the KSuite/EVC connection drawing or an electrical pinout from a primary
source.

The Siemens tip itself has an explicit `1` mark beside one corner of its center
2 x 5 contact pattern. The final mounting orientation must make that probe-side
pin-1 reference agree with the ECU-side pin-1 orientation shown by the selected
MSV70 family documentation.

## Correction log from the bench conversation

A few adapter guesses were made from partial photos before the full kit was
visible. Keep the corrections here so an early chat statement cannot become a
false project fact:

1. The `144300T104` rainbow cable was initially suggested as the likely direct
   path. It is not the preferred path when the proper positioning-frame BDM
   hardware is available.
2. A board labeled `5,6,14 JTAG NEXUS MPC5XX ADAPTER` was briefly mistaken for
   the desired Siemens board. **That was wrong.** It is the JTAG/Nexus family,
   not the BDM Motorola MPC5xx base needed for this workflow.
3. The current intended hardware family is **BDM Motorola MPC5xx base + Siemens
   pull-out tip**, subject to confirmation in KSuite's MSV70 family help before
   power is applied.

## First-read procedure

Alientech's K-TAG manual explicitly recommends making a **Backup before reading
or writing individual components**. It also notes that after a backup KSuite can
save the individual component files separately.

Our project procedure should be stricter than the minimum tool workflow:

1. Photograph ECU label, board, jumper state, adapter orientation, and bench
   power connections.
2. Select the exact BMW / Siemens MSV70 family in KSuite and save a screenshot
   of the connection/help page with the read set.
3. Verify the positioning adapter and pin-1 orientation against that help page.
4. Verify the EVC-required BDM jumper.
5. Use a current-limited, stabilized bench supply and the exact MSV70 main-
   connector power/ground pins from the selected connection documentation.
6. Perform **Backup / full read first**. Do not write.
7. Save every separate component KSuite offers (for example MICRO, EXTFLASH,
   EEPROM, if exposed by the selected family).
8. Power-cycle the setup cleanly and perform a second independent read.
9. Compute SHA-256 for the two full backups and each corresponding component.
10. Do not trust a read merely because KSuite reports success. Require the two
    independent reads to match byte-for-byte unless the tool format itself
    contains volatile metadata; if it does, extract and compare the actual ECU
    payloads separately.
11. Preserve the raw first reads read-only under a dated donor-ECU directory.
12. Only after the backup set is verified should we investigate cloning,
    CAS/ISN identity, software-family conversion, or code patches.

## Proposed repository intake for the first donor read

Private-archive intake example only; no read files or photographs are included
in this public edition. Suggested layout once files exist:

```text
reads/private/donor-msv70/<date>/
  README.md
  ktag-backup-1.*
  ktag-backup-2.*
  micro-1.*
  micro-2.*
  extflash-1.*
  extflash-2.*
  eeprom-1.*
  eeprom-2.*
  SHA256SUMS
  photos/
```

`README.md` should record:

- ECU BMW/Siemens part numbers and label identifiers;
- donor vehicle if known;
- KSuite version;
- KTAG hardware/version as reported by KSuite;
- selected KSuite family/protocol;
- adapter/base/tip used;
- bench supply voltage and current limit;
- exact power/ground pins;
- jumper state;
- read timestamps;
- file sizes;
- hashes;
- any communication errors or retries.

## Why this matters for the turbo work

Routine calibration changes can continue to use the existing MSV70 flashing
workflow. The bench BDM path is valuable because the project is now exploring
areas that may eventually require executable-code changes, including pressure
handling and Siemens' existing turbo-calculation branches.

A separately recoverable donor ECU is therefore a useful development target,
but only if its original state is captured correctly first. BDM is a recovery
and cloning capability; it is not a substitute for understanding what each
memory region contains.

## Open items before power

- [ ] Open the exact KSuite MSV70 family/help page and archive it.
- [ ] Confirm base board + Siemens tip selection from that KSuite family page.
- [ ] Confirm probe pin-1 orientation from the MSV70 connection drawing.
- [ ] Confirm exact main-connector +12 V and GND pins from a primary/source
      drawing rather than memory or forum material.
- [ ] Confirm jumper bridge visually after soldering, with no adjacent short.
- [ ] Decide bench supply voltage/current limit and record it.
- [ ] Inspect probe spring pins for free movement and adequate compression.
- [ ] Verify no metal frame/adapter part can contact nearby components.
- [ ] Make first read only; no write until duplicate-read hashes are reviewed.

## Sources

### EVC — primary physical BDM reference

- EVC BDM100 product/support list:
  https://www.evc.de/en/product/bdm/Default.asp
- EVC BDM100 manual (English), especially **Interconnections for Siemens MSV70
  ECUs**, manual page 20:
  https://www.evc.de/ftp/winols/BDM100-en.pdf

The EVC material directly supports MPC563 / AM29BDD160 / ST95320 / BDM141,
external ECU power, the required MSV70 jumper, and the positioning/contact
procedure.

### Alientech / K-TAG documentation

- K-TAG Master manual mirror used for accessory and backup-workflow reference:
  https://www.tuningtools.com/mwdownloads/download/link/id/6/

Relevant sections identify the `BDM Motorola MPC5xx (14KTMA0001)` accessory
family, `14AM00TBAS`, Siemens pull-out tip `14AM00TB01`, and the recommendation
to create a complete Backup before component-level read/write operations.

## Confidence boundary

**High confidence / source-backed:** MSV70 uses MPC563 BDM; EVC BDM141 physical
family; MSV70 requires a jumper; external main-connector power is required;
AM29BDD160 and ST95320 are listed memory devices; Alientech has a Motorola
MPC5xx BDM accessory group with a Siemens pull-out tip.

**Direct bench observation:** the actual project ECU is open; the ten elongated
pads and adjacent jumper region are visible; the project owns the relevant
adapter families; the small tip is marked `16` and has an explicit pin-1 mark;
temporary pads 6 and 8 measured as ground.

**Not yet promoted to fact:** exact ECU-side BDM pin numbering/orientation,
exact KTAG/KSuite family number for this particular hardware/software build,
exact bench power pins, and the exact contents/sizes KSuite will expose on this
MSV70. Resolve these before applying power or writing.
