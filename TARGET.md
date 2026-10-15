# Research target and public provenance

| Property | Research identity |
| --- | --- |
| Engine and controller | BMW N52, Siemens MSV70 |
| Reference application | 2006 E90 330i, six-speed manual |
| Software ID | `0049PP0L921S0AC1S` |
| Calibration definition family | `9PPL921S` |
| Full-image length | 2,621,440 bytes (`0x280000`) |
| Canonical image SHA-256 | `757a3df8dada0ba4c593a2b823d9c8fc4a88ff97ff8eabd12ff95a44d096f495` |
| Analysis baseline | Ghidra 12.0.4, big-endian PowerPC |
| Calibration base register | `r2 = 0x00847ff0` |
| RAM base register | `r13 = 0x007f8000` |

These facts identify the reference image behind the research, not the currently installed state of any vehicle. The image is not included and its hash is not permission to redistribute it.

## Address namespaces

For this image, calibration XDF offset `x` corresponds to full-read offset `0x40000 + x` and runtime address `0x840000 + x`. A signed `r2` displacement is relative to `0x847ff0`; an `r13` displacement is relative to `0x7f8000`. Internal-flash CPU addresses use `0x400000 + (file_offset - 0x200000)`. These relationships are image-specific.

## Deliberate exclusions

DME serial numbers, service-account/project identifiers, source filenames carrying those identifiers, local home-directory paths, raw firmware, and saved analysis databases are excluded. Public findings may retain function addresses, source hashes, and non-personal source locators for traceability; a locator is not a promise that the corresponding source is distributed.

Public-edition files are transformed copies, not byte-identical original evidence. The release manifest records their public digests and their source-file digests. Withheld firmware code and original captures cannot be independently checked from this edition alone; preserve that limitation when citing conclusions.
