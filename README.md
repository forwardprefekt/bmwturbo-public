# MSV70 / N52 research wiki

**Author: Griffin Heavy Industries**

A public research edition documenting Siemens MSV70 airflow/load, pressure, fueling, torque control, and protection behavior for the BMW N52. This is a technical research resource, not a flash-ready turbo tune or an operating-safety approval.

## Start here

- [Knowledge wiki](docs/wiki/README.md): topic navigation and explanations.
- [Open questions](docs/wiki/open-questions.md): what remains unproven and what evidence would resolve it.
- [Target and provenance boundary](TARGET.md): software identity without vehicle-specific identifiers.
- [How to contribute](CONTRIBUTING.md): bounded review tasks and submission requirements.
- [Public evidence availability](evidence/README.md): what the reports do and do not establish.
- [Rights and exclusions](NOTICE.md): review before redistributing or adding source material.
- [Licensing](LICENSE.md): CC BY 4.0 for original documentation/reports; MIT for original software examples.

## What this edition contains

Reviewed public copies of the wiki, supporting technical dossiers, and selected findings reports. The public copies retain qualifications and mathematical reasoning, while omitting private identifiers, firmware listings, raw source artifacts, and restricted supplier/reference material. Some technical interpretations remain informed by source artifacts not distributed here.

The findings describe source-bound static research. They do not independently establish which branches execute on a running vehicle, physical sensor transfer, delivered injector/spark behavior, or a safe boosted operating envelope. Planned observations and unperformed experiments must remain identified as such.

## What is deliberately absent

No raw ECU dumps, prepared firmware slices, saved Ghidra databases, diagnostic programs, A2L/XDF files, OEM manuals, supplier workbooks, vehicle captures, personal build/shipping records, or original repository Git history are included. Missing-source notices are deliberate evidence limitations, not invitations to obtain proprietary inputs from contributors.

This is a documentation/evidence edition, not an executable analysis environment. Commands described in research notes may require tools and a trusted, independently obtained source corpus that are not included. Do not describe those commands as reproduced from this checkout alone.

## Publication status

Prepared locally for owner review. Original documentation and research-report expression are licensed under CC BY 4.0; original software examples are licensed under MIT. See [LICENSE.md](LICENSE.md). These grants do not cover third-party rights the contributors do not hold; ownership/contributor permissions and third-party exclusions still need review before publication. See [NOTICE.md](NOTICE.md).

The separate private archive remains unchanged. This edition must be published from a fresh repository, never by changing that archive's visibility, mirroring its refs, or copying its Git metadata.
