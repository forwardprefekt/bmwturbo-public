# Contributing to the MSV70 research edition

Start with the [wiki](docs/wiki/README.md), [open questions](docs/wiki/open-questions.md), and [evidence boundary](NOTICE.md). No vehicle access is needed for documentation review. This checkout does not include the executable research toolchain or its source corpus.

## Useful first contributions

1. **Review one derivation.** Check a load-to-charge, pressure, interpolation, or timing calculation for dimensions, signedness, rounding, saturation, and branch assumptions. Submit a concrete counterexample or a bounded correction; distinguish an arithmetic review from an instruction-level verification.
2. **Improve one evidence boundary.** Identify a claim that confuses software state with a physical measurement, calibration coverage with safe operating headroom, or static selectors with observed execution. Cite the page, claim, and missing evidence.
3. **Design one measurement protocol.** Review channel definitions, independent instrumentation, timestamp uncertainty, or acceptance criteria in the validation documentation. A proposed protocol is not an executed experiment or authorization to test on a vehicle.

Instruction-level peer review requires legally accessible matching inputs obtained separately. Record the software identity, function/address range, method, and limitations without uploading restricted artifacts. If those inputs are unavailable, explicitly label the review as documentation/math review only.

## Issue or pull-request contents

Include:

- The exact page/section and bounded claim being discussed.
- Whether the contribution is a calculation, interpretation, source review, proposed procedure, or actual observation.
- Source identity, units, conditions, counterexamples, and uncertainty where applicable.
- What would falsify the conclusion and what remains unresolved.
- The provenance and redistribution permission for any new material.

Keep `not_run`, unresolved outcomes, and limitations intact until the required evidence actually exists. Do not silently promote a suggested name or sensor interpretation to an authenticated firmware symbol.

## Do not upload

No personal ECU dumps, VIN/security-state records, credentials, private messages, proprietary diagnostic programs, supplier workbooks, OEM manuals, or unlicensed firmware/code exports. Do not propose disabling safeguards as a shortcut to evidence collection. Avoid personal filenames, shipping details, account identifiers, and workstation paths.

By submitting an original contribution for inclusion, you agree to license your contribution under this edition's [license split](LICENSE.md): CC BY 4.0 for documentation and research-report expression, MIT for original software and software examples. You retain your copyright; no copyright assignment is required. Submit only work you own or have permission to contribute under the applicable terms. Describe third-party material separately with its provenance and license so its rights can be assessed; the project licenses do not override those terms.
