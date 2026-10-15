# MSV70 knowledge wiki

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This wiki explains what this project knows about Siemens MSV70, how the evidence supports it, and what remains unverified. It targets the project’s N52-powered 2006 E90 330i and software `0049PP0L921S0AC1S`. It is not a universal MSV70 definition or a flash-ready turbo calibration.

**Start with [identity and memory](identity-and-memory.md) and [evidence rules](evidence-and-tools.md).** Addresses, table layouts, and conclusions belong to the identified firmware image unless a page explicitly says otherwise.

## Find an answer

Choose a page by the question you need to answer:

| Question | Wiki page |
|---|---|
| Which ECU, software image, and address space does this describe? | [Identity and memory](identity-and-memory.md) |
| How does airflow become load and cylinder charge? | [Airflow and load](airflow-and-load.md) |
| Which pressures are measured, reconstructed, or modeled? | [Pressure model](pressure-model.md) |
| How does cylinder charge become an injector command? | [Fuel and injection](fuel-and-injection.md) |
| How do driver demand, torque models, and monitors differ? | [Torque and controls](torque-and-controls.md) |
| What do ignition, knock, lambda, and component protection actually do? | [Ignition, lambda, and protection](ignition-lambda-and-protection.md) |
| How are tables encoded, and which limits are distinct? | [Calibration and limits](calibration-and-limits.md) |
| What hardware, wiring, and recovery evidence exists? | [Hardware and recovery](hardware-and-recovery.md) |
| What can CAN and diagnostic logging establish? | [Logging and validation](logging-and-validation.md) |
| Where are the instructions, source hashes, and search tools? | [Evidence and tools](evidence-and-tools.md) |
| What must be resolved before changing the operating envelope? | [Open questions](open-questions.md) |
| What do the abbreviations and raw-state names mean? | [Glossary](glossary.md) |

## Read the evidence labels

The wiki distinguishes software facts from runtime and physical claims:

- **Instruction-proven (archive classification)**: the archive reports recovered instructions establishing an operation or conditional dataflow edge; native listings are withheld here, so this label is not independently demonstrated by the public page alone.
- **Calibration**: source bytes establish a value or layout; an XDF definition supplies its stated engineering interpretation.
- **Inference**: semantic naming or a cross-build comparison needs qualification.
- **Runtime-unverified**: the project has not established the actual branch, update timing, or state history with an accepted capture.
- **Physical-unverified**: software evidence does not establish sensor behavior, engine behavior, or hardware safety.

These reader-facing labels do not replace the formal classifications in the [findings index](../msv70-findings-index.md). Read the [evidence rules](evidence-and-tools.md) before promoting a candidate name into a physical signal.

## Understand the current conclusion

The project has recovered substantial software behavior across airflow, pressure, filling, fueling, torque, ignition, and protection. The canonical selectors choose modeled filling; neither the NoMAF comparison nor `S_EISY_TURBO` establishes a boost-capable system.

The [dynamic-validation protocol](../msv70-dynamic-validation-protocol.md) still records planned observations rather than accepted operating-envelope proof. Numeric headroom, map coverage, a passing software check, and a safe engine operating point are different claims.

## Use the wiki with the underlying research

The wiki is the navigation and explanation layer. The [detailed findings index](../msv70-findings-index.md), public domain dossiers, and curated findings records preserve reported claims and their conditions. The private archive retains the full instruction listings and source-bound evidence. A public record can preserve a conclusion without supplying enough material to independently reproduce it; withheld-source labels make that limitation explicit.

Technical pages retain authored mathematical/pseudocode derivations and repository-authored analysis-tool Python beside the claims they explain. Firmware assembly and Ghidra-reconstructed C excerpts are withheld pending a rights decision, as are their full native sources. Start with the [address-base walkthrough](identity-and-memory.md#read-the-address-bases-in-an-instruction), [excerpt representation rules](evidence-and-tools.md#read-code-excerpts-without-overstating-them), and [instruction glossary](glossary.md#read-the-powerpc-excerpts). These explanations are not standalone firmware, original BMW source, independent source-code verification, or evidence of live execution.

For project context outside MSV70 behavior, use the [repository overview](../../README.md). The build and parts tracker and reference inventory are excluded private-project context (supporting artifacts withheld from public edition).

## Keep this knowledge current

Update the relevant wiki page when its supporting evidence changes. Preserve raw addresses, source identity, selector conditions, and unresolved questions instead of replacing them with stronger prose. If a newer finding corrects an older dossier, cite the instructions and state the qualification explicitly.

This public edition uses repository-relative Markdown links and needs no documentation framework. Browse it in a Markdown viewer. Preparing this edition does not itself publish a GitHub Wiki or website. The [public target record](../../TARGET.md) retains software identity while excluding vehicle-specific security and personal information; the original firmware and private archive remain excluded.
