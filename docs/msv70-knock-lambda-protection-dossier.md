# MSV70 knock, lambda, and component-protection dossier

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

## Evidence and safety boundary

This issue-#38 dossier is a static review of stock MSV70 software, combining
the manifest-bound baseline with the source-hashed fresh continuation below.
It is **not permission** to weaken a monitor, zero a diagnostic,
change a fallback, or select boosted calibration values. XDF names, units, and
decoded ranges establish calibration intent. The instruction listing,
manifest-bound focused C, and canonical access facts establish software
behavior. Neither establishes physical detonation margin, measured exhaust
metal temperature, delivered lambda, delivered spark, actuator authority, or
hardware adequacy.

The `protection` domain declares 516 memberships: 17 functions, 56 exact XDF
components, 191 direct runtime-calibration references, and all 252 raw RAM
offsets accessed by those functions. The reviewed classification projects all
766 matching RAM instructions independently, including per-instruction width
and signedness. This reproducible predicate is intentionally broader than a
name grep and avoids silently treating an omitted selector as an explicit
unknown.

## Single-turbo lambda topology: offline closure

The [source-bound continuation](../analysis/derived/review-2026-09-05-lambda-topology-closure/findings.json)
recovers separate sensor records (`0x2d8` stride), controller records (`0x14c`
stride), and the bank path `4438/4710 -> controller histories -> 41e6/4332 ->
3dda/3f2a`. Canonical pointer `00407410` selects distinct normal/aged sensor-lag
and exhaust-delay maps for each bank.

Both sensor conversions use common **modeled** exhaust pressure `2b34`.
`00106998` builds it from `ip_prs_ex_inc(2ac6) + 2ccc`, not a turbine-inlet
pressure measurement. Its approximately 600–1600 hPa correction domain is
neither a sensor safety rating nor proof of accurate pre-turbine compensation.

Two probes in one fully mixed exhaust do not restore independently observable
three-cylinder-bank feedback. Bank-preserving sampling must be representative
and precede the first mixing junction; two bungs or a divided-looking flange
do not establish that. Pre-turbine sampling also requires part-specific
pressure, temperature, pulsation and transport characterization.

22RPD advertises a single-bank **code patch** with one post-turbo sensor, but
does not disclose an AC1S implementation, revision support or diagnostic/
adaptation handling. ESS's N52 header warning independently identifies feedback
timing as an application dependency. These are vendor evidence, not permission
to parallel signals, disable a bank or remove monitoring.

The retained exterior manifold photo does not expose internal separation.
The immediate owner input is a runner/bank/first-merge/bung diagram and internal
views, plus exact sensor identity. No tests or firmware/hardware actions were
performed. Original manufacturer/factory sources and alternatives are linked
from the continuation and prior-art manifest (supporting artifact withheld from public edition).

## Five separate software lanes

These lanes are related operationally, but the static evidence does not justify
flattening them into one synthetic safety chain.

### 1. Ignition selection

`func:000b0424` consumes adjusted-load state `ram:r13:6ab6` and separate
normal/safe/full-load calibration families. The reviewed map descriptors include
`cal:runtime:008524de`, `cal:runtime:00852668`,
`cal:runtime:00852b68`, and `cal:runtime:00852bea`. Its selector and blending
logic publishes the signed raw software result `ram:r13:6356` at
`insn:000b1dc0`, `insn:000b1dd4`, or `insn:000b1ddc`.
`func:0046d15c` reads that state at `insn:0046d184` and converts it into a
downstream raw software value. That consumer is not proof of coil actuation or
delivered spark angle. The active old/new-mode populations and every physical
consequence remain issue-#39 runtime questions.

### 2. Knock processing

`func:00439ef8` contains two different computations that the previous review
incorrectly joined:

- **Measurement-window scheduling:** selector `ram:r13:21d2` selects
  `ip_knkwb_0__n__maf` through `ip_knkwb_5__n__maf` (runtime descriptors
  `0084af8c`, `0084afdc`, `0084b02c`, `0084b07c`, `0084b0cc`, `0084b11c`)
  and the corresponding `ip_knkwe_*` descriptors at `0084b16c` through
  `0084b2fc` in steps of `0x50`. The beginnings are six **16-bit** values
  `ram:r13:2d54..2d5e`, stride two; the ends are six **16-bit** values
  `ram:r13:2d60..2d6a`, stride two. If an end is not greater than its
  beginning, the selected pair becomes raw `0x20/0x90`. These are not
  threshold-factor outputs (`focused/decompiled/00439ef8.c:58-116`).
- **Threshold processing:** selector `ram:r13:2023`, copied from
  `ram:r13:21d5`, selects `ip_fac_thd_knk_0__n__maf` through
  `ip_fac_thd_knk_5__n__maf`. Shared airflow/RPM storage remains
  `cal:xdf:014bf:x` and `cal:xdf:015b1:y`; the distinct z components remain
  `02a82:z`, `02ac2:z`, `02b02:z`, `02b42:z`, `02b82:z`, and `02bc2:z`.
  The actual final results are **bytes** at `r13+0x2049+cylinder`
  (`007fa049..007fa04e`), written by `insn:0043aa70`, not the window
  words (`focused/decompiled/00439ef8.c:307-354`).

The threshold uses the selected filtered word at `r13+0x2d48+2*cylinder`,
its high byte floored by `c_nl_min` (`00844cdd`), and the adjustment curve
`00843652`. It multiplies the adjusted byte by the selected threshold factor,
shifts right five, saturates through the unsigned-16 and unsigned-byte helpers,
conditionally multiplies by `(c_fac_tra_knk << 3)/256` when either
`ram:r13:2054` or `ram:r13:5570` is nonzero, then caps at `c_knk_thd_max`
(`00844b33`). Canonical raw constants are respectively 10, 33, and 225.
The selectors' static meanings are recovered; live cylinder populations and
their timing are not observed. In particular, the code does not establish a
safe behavior for a selector outside the reviewed range 0..5.

#### Fresh consumer continuation, 2026-09-04

All paths in this subsection are relative to
`analysis/derived/review-2026-09-04-knock/` (supporting artifact withheld from public edition). `provenance.json` records exact
commands, input and exporter hashes, Java/Ghidra identity, and output hashes.
Three fresh disposable Ghidra projects exported 27 functions and their complete
instruction bodies; old evidence and firmware inputs were not replaced.
`union-access-evidence.jsonl` retains the direct-r13, absolute-memory, and
pointer-fact search hits. Those tiers do not exhaust variable-indexed accesses:
the selected threshold read and write require their actual instructions.

1. `decompiled/00439ef8.c:187-265` reads the **previously stored** selected
   threshold byte at `insn:0043a514` and the selected unsigned word at
   `r13+0x3bf2+2*cylinder` at `insn:0043a520`. The normal compare requires
   `2053 != 0`, `2056 == 0`, `1eac == 0`, and countdown `556d == 0`.
   At or above the threshold, byte `2055` becomes one; exceeding twice the
   threshold selects the larger decrement. The update maintains per-cylinder
   words at `018025ae` and `018025a2`, with raw `0..0x8000` handling, a
   floor derived from `2052` and selected `1ff7`, and an aggregate-relative
   difference bound from `00844b29`. Disabled/countdown paths have distinct
   updates; threshold comparison is not unconditional.
2. The high bytes become arrays `2042..2047` and `203c..2041` at
   `insn:0043a6b4` and `insn:0043a6c8`. Only selector zero updates their
   respective six-byte means `2051` and `2048`. Thus an unchanged current
   threshold or mean does not imply no per-cylinder processing.
3. `decompiled/00457884.c:32-84` consumes `203c+cylinder`, combines it
   with base state `1dc8`, signed calibration corrections `00844b1e/1f+index`,
   and `1dd7`, subtracts `0x100`, and clamps to `0..255`. Further per-cylinder
   correction and limits feed `2000+cylinder`. Only the non-overridden path
   feeds this result to `1feb..1ff0` at `insn:00457a44`: `1fe0`,
   `008451fb`, and the `1fe1/008451fc` pair select fixed or alternate
   sources; `2008/551b` implement transition behavior. The scheduler calls
   this function with argument zero, so its argument/RPM entry gate is open.
4. `decompiled/00439014.c:14-30` processes three cylinders beginning at
   `21da`, wrapping at six. It converts each selected `1feb` byte to
   `byte*16-0x5f0`; record byte `r13+0x3160+0x18*cylinder` bit 7 chooses
   `0041e318` when clear or `0041e22c` when set. `0041e13c` first selects
   the corresponding bit in `global:01802a54`.
5. `decompiled/0041e318.c:16-25` and `0041e22c.c:16-25` adjust the
   signed input by a position-derived low byte and `0x700`, store a 16-bit
   value in `01802a46+2*cylinder` or `01802a38+2*cylinder`, and, under the
   matching mode bit, write it at **pointer-table target +2**, with the
   pointer loaded from `01802a20+4*cylinder`. The final stores are
   `insn:0041e3b4` and `insn:0041e2c8`, followed by service `00433da8`.
   This is a recovered software dispatch path, not a resolved physical
   actuator transfer or measured ignition retard.

Additional concrete consumers are retained without inventing cross-lane edges:
`00434d98.c:14-23` converts `2042..2047` to signed words `6952..695c`;
`000c8328.c:12-15` converts the means to `695e` and `global:01802d94`,
all gated by `1fb8 != 0`. `00476b8c.c:18-29,63-79` converts mean `2048`
into other software correction states. `0043aaf0` and `0043aca0` consume
the filtered/acquisition words for diagnostic history; merely sharing selector
`2023` does not make them readers of the threshold byte array.

The corrected screen starts at `2049..204e`, not the window beginnings.
It records `software_dispatch_recovered_6356_join_unresolved`.
For the concrete scalar-ignition hypothesis, `000b0424` adds `6358`, `63c8`,
and `6212` into `6356`; fresh `00105b3c.c:9-14` resets `6212` to zero
and `000b26dc.c:25-71` produces `63c8` from a separately gated computation.
No exact join from the threshold bytes into `6356` is established. This
does **not** negate the recovered per-cylinder dispatch path, nor prove global
absence of indirect influence. The continuation below resolves software
acquisition and canonical dispatch-pointer initialization without asserting
physical transfer. Correlation and co-access cannot supply physical proof.

#### Concrete dispatch and acquisition continuation

Fresh evidence:
[`dispatch-continuation/findings.json`](../analysis/derived/review-2026-09-04-dispatch-continuation/findings.json)
and its provenance retain 32 C exports/instruction bodies from three
successful fresh projects. The scalar `6356` join is **not a prerequisite**
for this independently connected per-cylinder route.

- Canonical descriptor `004049c4` contains count six and a channel-list pointer
  to `004049ac`, whose bytes are `00 01 02 03 04 05`. `001562a8` and
  `00156654` supply that descriptor to `00160eb8`. Its loop calls
  `00433784(channel)` and initializes `01802a20+4*i` to
  **`00704100+0x10*i`**, configuring each channel via `004337b8(channel,7,2,0)`.
- The previously indirect `+2` halfword destinations therefore resolve to
  **`00704102, 00704112, 00704122, 00704132, 00704142, 00704152`** under
  this initialization. `0041e13c` first publishes cached `+2`/`+0` words
  while switching the mode; the subsequent helper publishes the new command.
  This sequence is not one atomic new-value write.
- `00433da8` selects **`0070401a`** for canonical channels zero through five.
  Both polls require the selected two-bit field to clear before the store at
  `00433e70`. Request one produces words
  `0001, 0004, 0010, 0040, 0100, 0400`. `00161018` separately uses
  requests two/three for stop/start and updates the active mask and channel
  parameters. Initialization alone does not prove mode/cache reset ordering.
- Acquisition-word producer **`0041c3f0`** calls `0041f184(2)`, extracts
  bits 2..9 and stores `((result >> 2) & 255) << 8` at
  `3bf2+2*(argument&15)` (`0041c438`). The logical read uses a descriptor-selected
  acquisition block (`00704800`/`00704c00`), not the dispatch table above.
  Callback scheduling, valid argument population, sensor conditioning and
  physical knock units remain unproven.

The CPU-side parameter publication and service-request boundary is now
concrete. Runtime table integrity, exact silicon register semantics, timer
program, pins/drivers/coils and delivered ignition remain unverified.
Known addresses are not authorization to add MMIO reads to the RAM logger.

### 3. Lambda target selection

`func:0046e3e8` consumes raw cylinder-mass state `ram:r13:2ada`, uses six
base-lambda map families, and publishes raw intermediate states including
`ram:r13:2b96`, `2b98`, `2b9a`, and `2b9c`. `func:0046e7b0` selects a fixed
fallback or a curve-derived full-load software target and writes
`ram:r13:2b9e` at `insn:0046e7f0`. `func:0046ea20` combines those inputs with
the protection staging states and publishes raw summary outputs
`ram:r13:2ba8`, `ram:r13:2baa`, and `ram:r13:2bac`.

The map selection, clamps, and raw stores are proven. Runtime branch
populations, delivered lambda, injector capacity, fuel quality, and the
physical mixture needed by modified hardware are explicit unknowns.

### 4. Modeled component protection

`func:0010fcd4` uses the target-temperature table `KF_TA_VKSOLL`, the lambda
limit table `KF_BS_MX`, policy selector `CW_ATMOD`, calibration offsets, and
diagnostic service states. It publishes `global:01802fdc` and
`global:01802fde`. The word “temperature” here follows XDF calibration intent;
the reviewed path is a modeled/target calculation, not proof of a physical EGT
sensor.

The exact downstream software path is:

1. `func:0010fcd4` writes `global:01802fdc` at `insn:00110774` or
   `insn:001107b0`, and `global:01802fde` at `insn:00110c88`,
   `00110cc4`, `00110d28`, or `00110d64`.
2. `func:0046d0ac` shifts the two globals and writes `ram:r13:7478` and
   `ram:r13:747a` at `insn:0046d0b8` and `insn:0046d0c8`.
3. `func:0046e2f0` conditionally stages them, or fixed alternatives, into
   `ram:r13:2b5a`, `2b5c`, `2b5e`, and `2b60`.
4. `func:0046ea20` consumes those states and the full-load input before
   publishing `ram:r13:2ba8`, `2baa`, and `2bac`.

This is a connected software handoff. Physical temperature, delivered lambda,
thermal durability, and runtime activation remain unknown.

### 5. Diagnostic and fail-safe policy

`func:0043acf0` gates knock-sensor diagnostic behavior using raw operating
states and calibration thresholds, then issues service-report calls for raw
results `0x29` and `0x2a`. The report mechanics are reviewed. The final DTC
ownership and downstream ignition, lambda, torque, fueling, throttle, or
shutdown consequence are **explicit unknown**.

The pressure, airflow/load, fueling, and torque dossiers retain their own
monitor and fallback boundaries. Issue #38 does not invent a universal
cross-domain diagnostic consequence. Every policy row is marked
`required_not_disable`; bypassing or zeroing a safeguard is not a supported
resolution.

## Limits are not interchangeable

The machine-readable classification separates:

- calibration-axis endpoints from helper behavior and operating limits;
- sensor/electrical limits, which remain explicit unknown;
- arithmetic/storage saturation from calibrated control limits;
- diagnostic policy/debounce from a physical failure threshold;
- modeled component-protection targets from measured thermal state; and
- physical detonation and thermal safety, which remain explicit unknown.

A table maximum, integer clamp, diagnostic threshold, or modeled target is not
a physical safety limit.

## Issue #39 validation transfer

Five rows in `analysis/validation/msv70-dynamic-observations.jsonl` (supporting artifact withheld from public edition) remain
`execution_status:not_run`: threshold-byte feedback/per-cylinder dispatch,
ignition selectors, lambda selection, component-protection thermal behavior,
and diagnostic consequence. They require synchronized raw RAM plus independent knock
acquisition, commanded ignition timing, wideband lambda, both-bank EGT,
absolute MAP, post-intercooler IAT, and RPM as applicable. Each row defines a
capture and falsification condition. Correlation alone cannot upgrade a raw
alias, establish causality, validate physical safety, or authorize calibration
changes.

The knock capture must distinguish the six indexed threshold bytes and their
selector `2023` from window selector `21d2` and the 16-bit window pairs.
Synchronized pre/post observations of `2053/2056/1eac/556d`, acquisition
words `3bf2`, filtered words `2d48`, output bytes `203c/2042`, and dispatch
bytes `1feb` distinguish actual feedback from stale or overridden outputs.
These indirect arrays require instruction-reviewed width/address views;
absence from direct-r13 facts is not evidence that they are absent at runtime.
