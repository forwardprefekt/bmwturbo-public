# MSV70 direct BDM solder mapping

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Status: **bench-verified pad orientation; T104 color map requires cable-level continuity verification before power**.

This note maps the ten MSV70 BDM pads visible on the project donor ECU to the
standard Motorola/Freescale MPC5xx BDM signals, and records a candidate direct
wiring map for Alientech rainbow cable `144300T104`.

The ECU must remain unpowered while soldering or continuity testing. The BDM
header carries logic-level signals and target-VDD sense; **it is not an ECU
12-V power connector**. EVC documents MSV70 ECU power through the original
main connector.

## 1. Standard MPC5xx BDM pinout

The standard ten-pin MPC5xx/8xx BDM connector is independently documented by
P&E Micro / NXP and EVC:

> Standard-interface pinout table withheld pending third-party redistribution
> review. Consult the linked P&E/NXP and EVC primary sources for the actual
> signal assignments; this public note is not a complete connection sheet.

For MSV70, EVC specifies the `BDM141` probe, which is the original BDM pinout
at **1.27 mm pitch**, with external ECU power required.

## 2. Project ECU pad orientation resolved from continuity

The original bench photograph is withheld. The orientation description below
records the source author’s observation/inference, but readers cannot verify
its viewpoint from this public edition. Do not use it as a standalone wiring
reference; confirm the exact board and the primary connection drawing.

During the 2026-08-29 bench session the ten elongated pads were temporarily
numbered top-to-bottom as:

```text
1   2
3   4
5   6
7   8
9  10
```

With the ECU unpowered, temporary pads **6 and 8** measured continuous to ECU
ground.

The standard BDM interface has adjacent ground pins **3 and 5** on the same
side. Combined with EVC's BDM141/original-pinout documentation, this resolves
the project board orientation as follows when the ECU is held exactly like the
bench photograph with the `TOP` silkscreen toward the lower-right:

```text
PCB photograph orientation

left pad                  right pad
--------------------------------------
10  DSDO                  9  VDD/TVCC
 8  DSDI                  7  HRESET#
 6  VFLS1                 5  GND
 4  DSCK                  3  GND
 2  SRESET#               1  VFLS0
```

So **BDM pin 1 is the lower-right elongated pad** in that photograph
orientation.

This mapping agrees with the bench observation: the right-side middle and
second-from-bottom pads are BDM pins 5 and 3, respectively, and both are
ground.

### Pre-power sanity checks

Before any power is applied:

- BDM pins 3 and 5 must both show continuity to ECU ground;
- there must be no solder bridge between adjacent signal pads;
- BDM pin 9 must never be connected to +12 V;
- the separate MSV70 BDM-enable jumper documented by EVC must be treated as a
  separate connection from this ten-pad field.

After the ECU is powered correctly through its main connector, BDM pin 9 should
be a **logic-level target VDD reference**, not battery voltage. Do not use that
pad to power the ECU.

## 3. Candidate `144300T104` direct-wire map

A third-party reverse-engineering reference for Alientech cable `144300T104`
reports the following MPC5xx BDM color assignment:

> Copied third-party cable-color table withheld pending rights review. The
> project cable has not been continuity-verified; do not infer colors from this
> public edition.

A separate community thread specifically discusses using `144300T104` in place
of Alientech solder adapter `14AS00T03S`; a later participant reports cloning an
MS45.1 successfully using that mapping. This is useful corroboration, but it is
**not primary Alientech documentation** and clone cable color ordering may vary.

The private color mapping remains a **candidate only until the actual project
T104 cable is continuity-mapped**.

The photograph-specific cable-color solder diagram is also withheld pending
third-party rights review. No verified cable-color map is supplied here.

Do **not** solder from any copied color map without verifying the actual T104 cable.

## 4. How to verify the project T104 cable without powering anything

The project already owns the positioning-frame hardware, so it can be used as a
passive reference to map the clone cable instead of trusting wire colors.

1. Disconnect KTAG from USB, power, and ECU.
2. Assemble the known BDM Motorola base plus Siemens `TB01`/tip-16 path with its
   normal ribbon cable, but do not connect it to the ECU.
3. With a multimeter in continuity mode, map each of the ten center probe
   contacts to the corresponding physical socket in that ribbon cable's 26-way
   KTAG-side IDC connector.
4. On `144300T104`, map each colored conductor to the physical socket in its own
   26-way KTAG-side IDC connector.
5. Compare the two keyed 26-way connectors position-for-position. This gives a
   cable-specific mapping from the known Siemens probe contact to each T104
   color without relying on internet color conventions.
6. Label the T104 wires after verification.
7. As a final check after soldering, with the ECU still unpowered, verify that
   the two wires assigned to BDM pins 3 and 5 both have continuity to ECU ground
   and that no adjacent BDM pads are shorted.

This is preferable to assuming that every clone T104 cable follows the same
color sequence.

## 5. Solder-header options

Alientech's K-TAG manual documents `14AS00T03S` as the solder adapter for
Siemens/Marelli BDM Motorola MPC5xx ECUs and states that it uses an **8-pin,
1.27-mm male header**. The official direct-solder workflow therefore is not
simply a generic 2x5 ten-pin header wired one-for-one; the adapter board performs
its own routing.

For a DIY direct-wire method, the clean abstraction is instead:

```text
KTAG -> 144300T104 -> verified individual BDM wires -> ten MSV70 BDM pads
```

or purchase the real/equivalent `14AS00T03S` and use the documented 8-pin
header workflow.

Do not invent an `14AS00T03S` equivalent from an arbitrary 2x5 header unless its
routing has first been continuity-mapped from a real adapter or from the
existing Siemens probe hardware.

## 6. Sources and evidence quality

### Primary / manufacturer-level BDM definition

- EVC BDM100 product list: MSV70/MSS70 = MPC563, AM29BDD160, ST95320,
  BDM141; external power required:
  https://www.evc.de/en/product/bdm/
- EVC BDM100 manual, MSV70 connection and BDM-pad alignment sections:
  https://www.evc.de/ftp/winols/BDM100-en.pdf
- P&E Micro / Power Architecture BDM technical summary, standard 10-pin MPC5xx
  BDM signal layout:
  https://www.pemicro.com/downloads/tech_summary/PE3313%20-%20Technical%20Summary%20for%20Power%20Architecture%20Rev%20B.pdf
- NXP-hosted P&E Multilink technical summary, MPC5xx/8xx BDM pinout:
  https://www.nxp.com/docs/en/supporting-information/USBMLUNIVERSALFX.pdf

### Alientech accessory documentation

- K-TAG manual: `14AS00T03S` is the Siemens/Marelli BDM Motorola MPC5xx
  solder board using an 8-pin header; `14AM00TBAS` + `14AM00TB01` is the
  positioning-frame Siemens path:
  https://www.mk3.com/uploads/files/user-manual/K-TAG_manuale_S_ENU.pdf

### Secondary / community cable mapping

- AutoElectronics101, `144300T104` MPC5xx BDM color map:
  https://autoelectronics101.com/index.php/misc-edu-files/alientechs-rainbow-cable-144300t104-pinout-for-kess3-ktag
- Grand Tuning Forum thread on using `144300T104` instead of `14AS00T03S`:
  https://gt-forum.info/threads/5981/

## Confidence boundary

**High confidence:** standard BDM signal numbering; MSV70 uses BDM141/original
1.27-mm BDM pinout; the project board's two measured ground pads resolve the
ECU-side orientation shown above; pin 9 is target-VDD sense/reference, not 12 V.

**Medium confidence until local continuity verification:** `144300T104` color to
BDM-signal assignment. It is consistent across the current secondary evidence,
but the project's clone cable must be measured before soldering.
