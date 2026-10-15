# Check a claim against its evidence

[Wiki home](README.md) | [Identity and memory](identity-and-memory.md)

**Public-edition evidence boundary:** This page preserves the archive’s reported findings and uncertainties, not a new independent firmware verification. Raw firmware, native instruction listings, reconstructed firmware C, and other restricted supporting artifacts are not included. Firmware-code excerpts are withheld pending a rights decision; derivation alone does not resolve redistribution rights. Public summaries and reviewed findings records do not replace the withheld evidence.

This page explains the archive’s evidence model and how to distinguish the public summaries from the canonical bytes, reviewed code, and measurements that remain withheld. The wiki summarizes evidence; it does not create new authority by repeating a conclusion. Independent reproduction of a firmware claim requires lawful access to the exact source image, relevant definitions, and analysis context, none of which is supplied by a prose summary alone.

## Choose evidence that answers the question

Different artifacts answer different questions. A readable calibration name cannot establish a physical sensor connection, and a correlated log cannot create a missing static dataflow edge.

| Question | Starting evidence | Boundary |
|---|---|---|
| Which bytes and firmware version were analyzed? | [Vehicle identity](../../TARGET.md), loader provenance (supporting artifact withheld from public edition) | Artifact identity does not establish the vehicle’s current flashed state |
| Which operation reads or writes an address? | Structured facts (supporting artifact withheld from public edition), retained instruction listings | Preserve width, signedness, address space, and instruction identity |
| How does a function calculate a result? | Focused decompilations (supporting artifact withheld from public edition), domain instruction listings | Decompiled C is a reconstruction; check instructions for rounding, shifts, and branches |
| Which conditional relationship has been reviewed? | Relationship graph contract (supporting artifact withheld from public edition), domain dossiers | A direct call or nearby address is not a semantic transfer proof |
| What does a calibration encode? | Calibration component atlas (supporting artifact withheld from public edition) | XDF labels and display equations do not prove RAM identities or physical validity |
| What happened on the bench or vehicle? | Retained raw capture, provenance, calibration, timing, and fault records | A proposed procedure or `not_run` template is not a measurement |

The [findings index](../msv70-findings-index.md) defines the formal evidence classes. The retained-evidence guide (supporting artifact withheld from public edition) describes the historical run bundles.

## Interpret the wiki labels

The wiki uses reader-facing labels without changing the source classifications:

| Wiki label | Meaning |
|---|---|
| Instruction-proven (archive classification) | The archive reports exact operations or branch-contingent transfers supported by native-code evidence; this edition does not expose the native listing for independent checking |
| Calibration | Decoded source value or layout, with XDF meaning identified separately |
| Inference | A semantic interpretation, cross-build projection, or other conclusion requiring stated assumptions |
| Runtime-unverified | Actual branch activation, state history, or timing has not been established by an accepted capture |
| Physical-unverified | Sensor transfer, wiring behavior, combustion, actuator output, or safety remains outside the software proof |

A claim can have more than one label. For example, a subtraction can be instruction-proven while the physical meaning of both operands remains unverified.

## Read code excerpts without overstating them

The archive used three representations. This public edition retains only repository-authored Python and authored explanatory equations/layouts; the firmware representations are described here so readers can interpret the reported findings without mistaking them for included evidence:

| Representation | How to read it |
|---|---|
| PowerPC disassembly — withheld | Native instruction listings are not reproduced. In the archive, `ref READ`, `ref WRITE`, and branch/call `ref` rows are analysis annotations, not instructions |
| Ghidra-generated C — withheld | Firmware reconstructions are not reproduced pending a rights decision. They are not original BMW source; generated names, types, and variables require instruction-level checking |
| Repository-authored `python` or explanatory `text` — retained | Analysis-tool excerpts or authored equations/layout diagrams, not code running on the DME; full scripts and dependencies are withheld |

Source locators are preserved where useful, but unavailable artifacts are plain withheld-source labels, not public evidence links. Function addresses and RAM displacements belong to the [canonical image and address bases](identity-and-memory.md). Included Python excerpts and prose do not establish omitted firmware entry gates, helper contracts, callers, or runtime execution. A curated findings record preserves a reported result, not the complete native source. Obtain and review the full lawful source context before generalizing a local operation.

## Find code and source context

The private archive’s SQLite index is a disposable projection of tracked sources. **The commands below are historical examples requiring the separately obtained private tooling and complete authorized source corpus. They are not runnable instructions for this public edition:** the scripts, inventories, raw inputs, and saved Ghidra projects are not included. In a separately authorized complete workspace, these examples use a temporary index rather than either saved Ghidra project.

```bash
python3 scripts/msv70_index.py ensure --db /tmp/msv70-wiki.sqlite
python3 scripts/msv70_index.py query --db /tmp/msv70-wiki.sqlite \
  --kind address --value ram:r13:2ada
```

In that same separately obtained complete workspace, the historical bounded-context example was:

```bash
python3 scripts/msv70_graph.py context --db /tmp/msv70-wiki.sqlite \
  --seed ram:r13:6ab6 --direction downstream --max-depth 2 \
  --limit 30 --format markdown
```

Read the returned conditions, missing relationships, provenance, and truncation notices. An address search can return several namespaces or roles; an exact raw ID is more useful than an unqualified number.

The index guide (supporting artifact withheld from public edition) and graph guide (supporting artifact withheld from public edition) document the complete commands. The index inventory remains the authority for searchable membership; adding a wiki page does not automatically add it to that inventory.

### Why an exact RAM query preserves its namespace

The following **repository-authored analysis-tool Python excerpt (not firmware source)** from `_address_selector`, lines 1856–1866 (supporting artifact withheld from public edition), parses an `r13` displacement separately from absolute addresses:

```python
def _address_selector(term: str) -> tuple[str | None, int | None, str | None]:
    value = term.strip().lower()
    if value.startswith("ram:r13:"):
        offset = value.removeprefix("ram:r13:")
        sign = -1 if offset.startswith("-") else 1
        if offset.startswith("-"):
            offset = offset[1:]
        if not re.fullmatch(r"[0-9a-f]{1,4}", offset):
            raise IndexContractError("invalid r13 address query")
        number = sign * int(offset, 16)
        return "r13", number, None
```

For `ram:r13:2ada`, this branch returns namespace `r13` and displacement `0x2ada`, not absolute address `0x2ada`. Later branches handle `r2`, XDF, full-read, runtime, and other namespaces. This excerpt explains query parsing only; the index’s source verification and result provenance still determine what evidence a query returns.

## Reproduce native analysis without overwriting its sources

The private archive’s retained baseline pins Ghidra 12.0.4 and its release metadata, analyzer configuration, inputs, and JVM. Those inputs and saved projects are not supplied in this edition, so the public tree alone cannot reproduce the native analysis. The Ghidra guide (supporting artifact withheld from public edition) documents disposable rebuilds and the saved project databases.

Use a fresh output/project location for independent analysis. Never use a tracked project name for a headless import. A newer Ghidra release can provide an independent cross-check, but it is not an exact reproduction of the pinned baseline.

When reviewing arithmetic, retain the original widths and operation order. Check fixed-point rounding, 32-bit intermediate overflow, division-by-zero behavior, saturation, narrowing stores, and selector conditions. [Calibration and limits](calibration-and-limits.md) explains why these boundaries cannot be collapsed into one maximum.

## Promote a finding deliberately

Before strengthening a wiki claim, retain enough information for another reader to challenge it:

1. Bind the observation to the canonical source hash or explicitly name another image.
2. Record the exact address, instruction or source locator, and input interpretation.
3. State branch conditions, pointer identity, and any missing producer or consumer.
4. Separate calculated behavior from observed runtime behavior.
5. Preserve a contradictory result instead of renaming it as support.
6. Link an underlying artifact only if it is included and approved for release; otherwise retain an explicit withheld-source label. Update the relevant domain page and open question without claiming that an unavailable source has become publicly reproducible.

A test run or validator exit status is not a substitute for reviewing raw evidence. Emulator experiments can establish behavior for supplied synthetic state; they do not establish scheduler reachability, electrical behavior, or engine safety.

## Current unknowns

The wiki does not claim whole-program semantic reconstruction or complete physical validation. Historical reports belong to their recorded source tree and toolchain. Private firmware, source snapshots, supplier material, and bench records retain their handling restrictions even when their conclusions appear in a readable page. A reconstruction or derivative report is not automatically cleared for redistribution. Firmware assembly and reconstructed C remain omitted pending a rights decision.

Related pages: [Logging and validation](logging-and-validation.md), [open questions](open-questions.md), and the [glossary](glossary.md).
