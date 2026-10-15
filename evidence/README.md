# Evidence availability in the public edition

This edition contains seven edited research summaries, not the private evidence corpus:

- [Dispatch continuation](../analysis/derived/review-2026-09-04-dispatch-continuation/findings.json)
- [Load continuation](../analysis/derived/review-2026-09-04-load-continuation/findings.json)
- [Offline MAF closure](../analysis/derived/review-2026-09-04-offline-maf-closure/findings.json)
- [Pressure continuation](../analysis/derived/review-2026-09-04-pressure-continuation/findings.json)
- [Pressure interface closure](../analysis/derived/review-2026-09-04-pressure-interface-closure/findings.json)
- [Injector equation closure](../analysis/derived/review-2026-09-05-injector-equation-closure/findings.json)
- [Lambda topology closure](../analysis/derived/review-2026-09-05-lambda-topology-closure/findings.json)

Each report begins with a `public_edition` block recording its original report SHA-256, canonical image identity, omitted fields and rewritten sections. Original source hashes remain provenance identifiers. They do not provide the corresponding bytes or independently authenticate every interpretation. Historical extraction counts, tool results and source-evidence references describe the original review; they are not new public-edition test results.

## Included and omitted material

The summaries preserve original research reasoning, mathematical expressions, decoded addresses, selected scalar observations, conclusions, qualifications and unresolved outcomes. They do not constitute firmware source, executable replacements, calibration approval or a safe operating envelope.

The public edition does not include firmware images, instruction/byte dumps, reconstructed firmware C, Ghidra projects or exports, extracted definition/fact datasets, third-party manuals, supplier workbooks or supplier characterization datasets. Embedded byte/word sequences and supplier cell/table projections were omitted or replaced with explicitly identified research interpretations where needed. The original private reports retain the fuller evidence context outside this edition. Personal source-image identifiers and private execution paths are not part of the public provenance.

`private-static-export:*` and `private-source:*` values are descriptive non-public identifiers, not URLs or repository files. Other retained historical filenames and line/function locators in provenance likewise do not promise a distributed artifact. A public source URL identifies its publisher's material, not a local copy or a redistribution permission. Line anchors into private exports do not apply to these edited summaries.

## Independent review boundary

Reproduction requires separately and lawfully obtained, hash-matching source material, appropriate tools and sufficient permissions. No access entitlement, public download, automatic access on request or rights-clearance determination is represented here. Commands discussed elsewhere that depend on the original corpus are examples for such a separately obtained environment, not workflows made runnable by this public edition.

Omitting a field is not a negative finding: it does not show that a firmware branch, supplier measurement or physical effect is absent. Conversely, a retained interpretation is not independently reproducible merely because its source hash is public. Static software topology is distinct from runtime activation, physical wiring, sensor transfer, delivered fuel/ignition and operating safety. Those distinctions and the reports' unresolved inputs remain in force.

No new tests, builds, lint, firmware execution, ECU reads or vehicle observations were performed when preparing these public summaries. No blanket license assignment is made for firmware or third-party material.
