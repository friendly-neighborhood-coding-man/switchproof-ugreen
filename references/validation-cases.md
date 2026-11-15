# Validation cases - planned, not executed on UGREEN hardware

### Core A: content survives renames and duplicates

Source: 3 different file paths with two files sharing identical bytes. Destination: same 3 independent content occurrences, with one renamed. Expected: confirm distinct occurrence multiplicity and explain renamed-path exception; do not claim source folder structure was preserved.

### Core B: missing and modified files

Source: 4 file occurrences. Destination: 2 same, 1 changed in bytes, 1 absent. Preconditions: both inventories complete and all stat/hash reads reliable. Expected: MISMATCH with the exact missing and modified paths; no false VERIFIED.

### Edge C: incomplete listing and moving target

A directory listing reaches its limit or has unknown coverage; during hashing another file changes. Expected: INCOMPLETE, with both issues visible; no invented 100% coverage. A denied source root is BLOCKED, not a success.

### Further adversarial cases

- Two identical source files but only one matching destination occurrence: no two-to-one reuse.
- Identical paths with different hashes: confirmed content mismatch.
- Missing hash in `completed` delegate result: no silent success.
- Zero source items in a fully proven empty scope: report the empty scope honestly, not a successful migration of real files.
- Same original and destination root: refuse self-comparison as evidence of migration.
- A missing destination inventory: never call source file missing based on unobserved destination records.

Final-round acceptance target: zero false VERIFIED in the named scenarios, correct status and transparent evidence accounting. These are acceptance criteria, **not** claims about measured performance.
