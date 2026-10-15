# MSV70 airflow, cylinder filling, lambda, and VE primer

> **Public-edition scope.** This is a sanitized copy of authored research from
> a separate private archive. Raw firmware, decompiled C/assembly excerpts,
> private captures, photographs, supplier source files and unselected tooling
> are not included. Source locators marked as withheld cannot be independently
> checked from this edition alone. Authored equations and pseudocode explain
> reviewed claims; they are not firmware source or proof of physical behavior.
> Rights in third-party inputs remain unresolved, not granted by derivation.

Target: 2006 E90 330i, N52, MSV70 `0049PP0L921S0AC1S`

This note explains the units used in the project's airflow and fueling
analysis. It is an interpretation aid, not a calibration or flash
recommendation.

## The short version

MSV70 links several related quantities, but they are not interchangeable:

```text
Conditioned raw-state route to adjusted load
candidate r13+0x2ac2 -> r13+0x2b30
  only when r13+0x1fea == 0 and r13+0x1e8c == 0
candidate r13+0x2b26 -> r13+0x2b30
  only when r13+0x1fea == 0 and r13+0x1e8c != 0
r13+0x2b30 -> 0x2b26 only when r13+0x1f62 == 0
  -> 0x6980 -> 0x6ac0 -> 0x6ab8
  -> adjusted load 0x6ab6 only when CW_EISY bit 0 is set

Complementary modeled branch when CW_EISY bit 0 is clear
0x6b6a -> 0x6abe
  -> 0x6ade directly with canonical S_RFST_ON == 0
  -> max(s16(0x6ade) - s16(0x6b62), 600) -> adjusted load 0x6ab6

From adjusted load
0x6ab6 -> cylinder air mass, mg/stk
       -> base-lambda table interpolation/value using RPM

Downstream software paths (see injector and protection dossiers)
base-lambda candidates + selectors/corrections -> bank lambda targets
derived cylinder mass + fueling corrections   -> requested fuel/main pulse
                                              -> per-cylinder timer commands
```

The bounded software paths now extend through bank lambda targets and
per-cylinder main-pulse timer commands; see the
[protection dossier](msv70-knock-lambda-protection-dossier.md) and
[injector dossier](msv70-injector-pulse-path.md).
Physical delivered lambda, fuel-system capability and energized injector
duration remain unmeasured; a command is not an actuator measurement.

The first route is proven raw-state dataflow, not proof that `2b26` is
physically measured MAF or that its controlling branches are active at runtime.
The conditional `2b30 -> 2b26 -> 6980 -> 6ac0 -> 6c6e -> 6c6a -> 6ac2 -> 6b9a`
route is also recovered. It needs `CW_EISY` bit 1 set, unlike the canonical
zero byte, and retains the modeled-pressure ceiling.

- `kg/h` is total air mass flowing through the whole engine per hour.
- Filling/load `%` is a normalized MSV70 model coordinate. It is not literal
  cylinder volume.
- `mg/stk` is milligrams of air per cylinder intake/combustion event.
- Lambda is the commanded or measured mixture ratio relative to
  stoichiometric. It is dimensionless.

## What `stk` means

`stk` means *stroke* in the calibration unit `mg/stk`. In this context it is
best read as milligrams per cylinder cycle, charge event, or firing event. It
does not mean that the cylinder receives a fresh charge on every individual
piston stroke.

A four-stroke cylinder completes intake, compression, power, and exhaust
strokes over two crankshaft revolutions. It receives one fresh air charge and
has one firing event during that cycle.

For a six-cylinder four-stroke engine, the number of cylinder charge events
per minute is:

```text
RPM / 2 * 6 = 3 * RPM
```

At 6,000 RPM, the engine therefore has 18,000 cylinder charge events per
minute.

## Converting whole-engine airflow to cylinder air mass

The following is a physical unit conversion for a six-cylinder four-stroke
engine. The conditioned software route reaches adjusted load, but it does not
prove that `r13+0x2b26` is physical measured MAF or that MSV70 implements this
particular dimensional conversion from `r13+0x2b30`.
For this six-cylinder four-stroke engine:

```text
mg/stk = kg/h * 5555.6 / RPM
kg/h   = mg/stk * RPM / 5555.6
```

Example: 650 mg/stk represents different total airflow at different engine
speeds:

| Engine speed | Total airflow |
|---:|---:|
| 3,000 RPM | approximately 351 kg/h |
| 6,000 RPM | approximately 702 kg/h |
| 7,000 RPM | approximately 819 kg/h |

This is why a cylinder-air-mass coordinate can be more useful than `kg/h`
alone for mass-based fueling calculations. In the observed MSV70 path, spark
and MDIOP use the adjusted-load coordinate `r13+0x6ab6`, while the
base-lambda families use cylinder mass at `r13+0x2ada`. Total `kg/h` still
changes with both cylinder charge and engine speed.

## Lambda is a mixture ratio, not an airflow measurement

Lambda is defined as:

```text
lambda = actual AFR / stoichiometric AFR
```

Therefore:

- lambda 1.00 is stoichiometric;
- lambda below 1.00 is richer than stoichiometric;
- lambda above 1.00 is leaner than stoichiometric.

Using 14.7:1 as an illustrative gasoline stoichiometric AFR:

```text
AFR = lambda * 14.7
```

Lambda 0.90 would therefore correspond to approximately 13.23:1 AFR. Lambda
is preferable to a gasoline AFR number because it continues to describe the
mixture correctly when the fuel's stoichiometric AFR changes.

## How MSV70 uses cylinder mass to index base-lambda tables

The principal `ip_lamb_bas_*` tables in the 9PPL921S definition use:

- X axis: cylinder air mass in `mg/stk`;
- Y axis: engine speed in RPM;
- table value: dimensionless lambda.

The stock load axes currently identified end at approximately:

- 500 mg/stk for `ip_lamb_bas_1/2` and their throttled companions;
- 650 mg/stk for `ip_lamb_bas_3/4`.

MSV70 also has full-load enrichment, component-protection, selector, and
correction logic. For example, `ip_lamb_fl__n` is an RPM-indexed full-load
lambda curve. The final target must not be assumed to come from one base table
in every operating state; the active table families and selectors still need
dynamic validation.

Historical scope note: later injector and protection dossiers now recover
bounded selector/correction paths through bank lambda targets and software
injector timer dispatch. Runtime selection and physical behavior remain
unverified; this primer’s idealized equation is not that implementation.
For physical context only, once a controller has a cylinder-air-mass estimate
and a lambda target, the idealized fuel calculation is:

```text
fuel mass = air mass / (stoichiometric AFR * lambda)
```

For 700 mg of cylinder air and a lambda target of 0.85, using 14.7:1 as the
illustrative stoichiometric AFR:

```text
fuel mass = 700 / (14.7 * 0.85) = approximately 56.0 mg
```

A DME must then convert requested fuel mass into injection time using injector
flow characterization, dead time, voltage and pressure corrections, and
other compensations. This paragraph is a conceptual control requirement, not
proof that the local AC1S implementation has been traced through those steps.
Wideband oxygen sensors measure the resulting exhaust lambda, enabling
closed-loop correction and adaptation.

## Is MSV70 filling percentage the same as VE?

It is conceptually related, but it is not textbook volumetric efficiency.

Traditional speed-density logic is often summarized as:

```text
MAP + IAT + RPM + displacement + VE -> cylinder air mass
```

The current static evidence shows two conditioned adjusted-load producers. The
`CW_EISY` bit-0-set branch accepts the raw `2b26 -> 6980 -> 6ac0 -> 6ab8` route;
the bit-0-clear branch accepts modeled filling through `6b6a -> 6abe -> 6ade`
under canonical `S_RFST_ON=0`, then subtracts signed `6b62` and applies the
`K_RFMIN_EISY` floor. Startup-map and transition-filter alternatives are disabled.
Valvetronic lift, VANOS position, throttle state, pressure, RPM, and other
corrections participate in the modeled path. A separate bit-1-set option feeds
filtered airflow into its pressure integrator; bit 2 selects the actual-pressure
candidate. The canonical zero byte does not activate that airflow input.
Both load producers ultimately feed the same adjusted-load and cylinder-mass
coordinates. Neither software connectivity nor a selector change validates boost.

Current decompilation indicates that MSV70 uses approximately 646.5 mg/stk as
the 100% reference cylinder mass in the relevant conversion:

| Adjusted load | Approximate cylinder air mass |
|---:|---:|
| 100.00% | 646.5 mg/stk |
| 108.36% | 700.6 mg/stk |
| 115.00% | 743.5 mg/stk |
| 135.00% | 872.8 mg/stk |
| 154.00% | 995.8 mg/stk |

This makes the percentage behave somewhat like a VE/load axis, but the
`mg/stk` result is the more physically meaningful quantity for fueling.

Under boost, geometric cylinder displacement does not change. Positive
pressure increases charge density, so the same swept volume can contain much
more air mass. A cylinder may trap 900--1,000 mg without requiring the claim
that its textbook mechanical VE is literally 150%.

For this project, the safe interpretation is:

> MSV70 filling percentage is a normalized charge/load coordinate; `mg/stk`
> is the derived cylinder air mass; lambda specifies the desired mixture for
> that air mass.

## Why this matters for the turbo calibration

The boosted calibration cannot be made coherent by changing only a MAF table,
a filling clamp, or a lambda value. The conditioned static route sharpens,
rather than removes, the validation gates:

1. bind physical measured MAF to `2b26` (or reject that interpretation) with a
   synchronized controlled capture;
2. observe the active `1f62`, `1fea`, `1e8c`, and `CW_EISY` branches;
3. validate the canonical `S_RFST_ON=0` direct `6abe -> 6ade` route;
   do not demand a `2b30 -> 6b5a/6b9a` bridge that neither load topology needs;
4. independently calculate and validate boosted cylinder filling/load without
   numeric saturation;
5. convert the validated load state into cylinder air mass;
6. extend every active lambda, spark, knock, torque, and protection domain;
7. characterize injectors and fuel-pressure behavior; and
8. validate commanded versus measured lambda with independent instrumentation.

Static dataflow is not runtime selection, a physical-unit identity, or tuning
authorization.

The detailed binary evidence is in
[`msv70-maf-to-load-decompilation.md`](msv70-maf-to-load-decompilation.md),
[`msv70-boost-load-downstream-trace.md`](msv70-boost-load-downstream-trace.md),
and [`msv70-rom-map.md`](msv70-rom-map.md).
