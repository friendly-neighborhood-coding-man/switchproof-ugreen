# SwitchProof — Round 1 validation report

**Report date:** 2026-11-15  
**Project:** `switchproof`  
**Package reviewed:** `switchproof-final.zip` (original ZIP SHA-256: `5602269c195435ab239ef78cd7eb4b144e3398825ab4fe92ed653bcf10f7d744`)  
**Recommendation reported by the synthetic test operator:** `READY_FOR_ROUND1_SUBMISSION` **conditional on retaining fixes T04 and T15**.

> **Evidence boundary:** All delegation results in the reported test suite were *simulated*. No UGREEN NAS was touched. No actual Uliya LLM/runtime execution, installation, upload or interoperability test passed. The reported outcome validates a deterministic reference model's interpretation of the Skill's rules, not live-device reliability.

## Static checks

The operator ran the official `uliya-skill-kit` version **0.7.7 (en)**, `scripts/check-skill.mjs`, check format **2.0.0**, catalog **2.2.0**, using Node 22 in an isolated environment.

| Check | Before fixes | Final ZIP |
| --- | ---: | ---: |
| Errors | 11 | **0** |
| Warnings | 1 | **4** |
| Checker status | `not-uploadable` | `static-pass-needs-target-validation` |

All four remaining warnings are `DELEGATE_DYNAMIC_VALUE_UNRESOLVED` on the dynamic delegate instruction templates (reported lines 52, 59, 68, 91). They involve user-selected roots, values obtained from preceding delegations or approved report text; removing the warning by hiding a placeholder would not verify runtime substitution. The placeholders must be populated from real authorized results before a call is dispatched.

Catalog ownership was checked manually in the report: `ug-filesystem` operations belong to `file-agent`, and optional `ug-knowledge save-text` belongs to `knowledge-agent` and requires approval. This is **not** a proof that live delegation behaves as expected.

## Synthetic adversarial rules test

The operator's deterministic Python reference model `adversarial-harness/switchproof_model.py` was run against simulated `delegation_result` envelopes. The harness was **not included** in the Uliya Skill ZIP. The reported results were:

| Ruleset | Passed | Failed | False VERIFIED |
| --- | ---: | --- | ---: |
| Before two rule corrections | 41/43 | T04, T15 | 2 |
| Final submitted rules | **43/43** | none | **0** |

Test mix: **5 positive controls** and **38 adversarial scenarios**, including missing roots, incomplete listings, duplicate contents, altered files, unavailable hashes, `completed` with missing evidence, approval outcomes, symlinks, and conflicting responses.

### Two rule defects found and fixed

- **T04 — Nested source/destination roots:** When one root sat inside the other, the destination listing could include the original files themselves. The audit could then return VERIFIED even when nothing was migrated. **Fix:** Gate 0 rejects identical, nested or overlapping roots. Final expected status: BLOCKED.
- **T15 — Empty source scope:** An empty but exhaustively listed source could satisfy every matching condition vacuously. **Fix:** `verification-policy.md` and Gate 4 require at least one in-scope source file occurrence for VERIFIED; proven-empty source returns INCOMPLETE.

One additional failed initial test was attributed to a harness fixture mistake in T19 (directory-entry modification instead of the file size); the operator corrected the fixture and did **not** classify it as a Skill defect.

### Positive controls

- P01: identical, complete trees → VERIFIED.
- P02: exclusions are explicitly scoped, remainder complete → VERIFIED.
- P03: identical names in different folders, different contents, properly accounted → VERIFIED.
- P04: destination-only extras do not invalidate source-content completeness → VERIFIED, with extras disclosed.
- P05: a renamed destination file can match content → VERIFIED for content scope, **not** path preservation.

### Selected adversarial expectations

| Case | Expected final result |
| --- | --- |
| T04 nested/overlapping roots | BLOCKED |
| T08 listing reaches its bound | INCOMPLETE |
| T09 missing authoritative listing-completeness signal | INCOMPLETE |
| T15 proven-empty source | INCOMPLETE |
| T18 source modified during verification | INCOMPLETE |
| T20 hash missing from completed delegation | INCOMPLETE |
| T22 `completed` with an empty result | INCOMPLETE |
| T26 confirmed missing file after complete enumeration | MISMATCH |
| T27 matching path/size, different bytes | MISMATCH |
| T28 two identical source occurrences, one destination occurrence | MISMATCH |
| T33 save approval rejected | Audit status unchanged; SAVE_FAILED |

The exact per-case synthetic outputs were reported separately by the operator; they are **not present** as executable fixtures in this repository. A rules test can pass while the runtime interprets the same instructions differently.

## Unresolved risk and follow-up

1. **LLM adherence not tested.** A live Uliya model may interpret instructions or sub-agent outputs differently.
2. **Real enumeration semantics unknown.** The published `ls`/`find` capability does not guarantee pagination or an authoritative full-count signal; large trees may remain INCOMPLETE.
3. **Third-party NAS source visibility unknown.** A mounted share accessible in a UI is not proof of file-agent access.
4. **Symlinks:** whether runtime listing distinguishes symlink types is not established. Test T16 assumes they are distinguishable.
5. **Concurrent changes:** before/after metadata helps detect instability but does not establish an atomic snapshot.
6. **Prompt injection:** malicious file or folder names were not assessed with a live LLM.
7. **Coverage and usability:** destination-only files are not yet a dedicated category in the report template; duplicate relocations may conservatively remain INCOMPLETE.
8. **Save receipt behavior:** required result fields and approval transitions must be verified on device.

## SHA-256 of the four source Markdown files in the final ZIP

| Path | SHA-256 |
| --- | --- |
| `SKILL.md` | `e18bdf31d0b87cb922a762abcaca4d8f125368e6c03d7d8ef6eef5aa59812792` |
| `references/verification-policy.md` | `e03020eeacc6a25e70a38d8f74fbf66f8125edaf58eab0228b490ba1ab7c1fe4` |
| `references/report-format.md` | `354bd19b0433f29ec2247ad1843b714872d5dbc6322fdfa48c8ab471ffadae9f` |
| `references/validation-cases.md` | `669cac1ff4ef3f4111e2250316cac73930c833c7469cefda3fe855d151982036` |

**Round 1 interpretation:** A statically checked, synthetically evaluated *proposal-stage Skill*. Actual reliability, source NAS access, installation and compatibility are unverified and belong to the target-device evaluation.
