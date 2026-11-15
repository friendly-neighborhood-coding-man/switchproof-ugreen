# SwitchProof

**Evidence-based NAS migration verification for UGREEN Uliya**

> **Copying files is not the same as proving a migration succeeded.**

SwitchProof is a **read-only, SKILL.md-based Uliya agent workflow** for auditing whether files copied from an older NAS to an accessible UGREEN NAS location arrived intact. It compares files through authorized UGOS expert-agent capabilities, reports matched, mismatched and unverified content, and refuses to declare a migration verified without sufficient evidence.

**Status: Round 1 prototype — static checker passed; synthetic rule tests passed; UGREEN NAS runtime not tested.**

## The problem

Users considering a move from Synology, QNAP or another NAS may hesitate because a copy job marked “complete” does not prove every expected file survived. SwitchProof aims to make *file-content verification* understandable, explicit and conservative. It is an **auditor**, not a data-transfer engine.

## How it works

1. **Preflight** — confirm user-approved source and destination roots, reject identical, nested or overlapping roots, and confirm access.
2. **Discover** — enumerate files within the declared scope; do not mistake bounded listings for exhaustive inventories.
3. **Verify** — collect before/after metadata and SHA-256 digests for accessible files.
4. **Reconcile** — compare file occurrences one-to-one, including renamed files and duplicate-content cases.
5. **Audit** — issue **VERIFIED**, **MISMATCH**, **INCOMPLETE**, or **BLOCKED** according to evidence.
6. **Report** — show verified observations and coverage limitations; save a report only if the user explicitly approves.

**VERIFIED means only that the declared, nonempty file-content scope is supported by complete evidence.** It is **not** a claim that a full NAS migration, ACLs, application databases, snapshots, album metadata or recovery readiness are verified.

## Compatibility and safety

- **Skill type:** Uliya orchestration instructions, not an executable plugin.
- **Host-provided capabilities:** `ug-filesystem` through `file-agent` (`ls`, `find`, `stat`, `sha256sum`, optional `getfattr`); optional `ug-knowledge save-text` through `knowledge-agent`, gated by approval.
- **Read-only audit:** no copying, modification, deletion, repair, shell commands, arbitrary scripts or remote uploads.
- Requires *actually accessible and user-authorized* source and destination paths; third-party NAS share visibility has **not** been confirmed.
- Directory-listing completeness, symbolic-link behavior and real sub-agent response formats require target-device validation.

## Validation evidence (November 15, 2026)

| Check | Result | What it establishes |
| --- | --- | --- |
| Official Uliya kit static checker `0.7.7`, catalog `2.2.0` | **0 errors, 4 warnings**; `static-pass-needs-target-validation` | Local static compatibility checks only |
| Deterministic *synthetic* rule-level suite | **43/43 pass** (5 positive controls, 38 adversarial scenarios) | The reference model followed final decision rules on simulated delegated responses |
| False VERIFIED outcomes in synthetic suite | **0 after two rule fixes** | Not evidence of a zero false-positive rate in a live LLM |
| Real UGREEN NAS install, Uliya execution, migration test | **Not performed** | Runtime compatibility and behavior remain unknown |

The rule tests uncovered two conditions that originally caused **false VERIFIED**: overlapping source/destination roots (T04) and a proven-empty source scope (T15). Both are now blocked or reported INCOMPLETE respectively. See [validation report](docs/validation-report.md).

The four static warnings concern runtime placeholders in delegation instructions (`DELEGATE_DYNAMIC_VALUE_UNRESOLVED`); they remain visible intentionally, pending live target validation.

## Repository layout

```text
SKILL.md
references/
  verification-policy.md
  report-format.md
  validation-cases.md
docs/
  validation-report.md
README.md
```

The separate Python reference test harness and synthetic fixtures **are not included** in the submission Skill package or this source-only public repository. The reported test results are not a runnable test suite in this repository.

## Try it (when a compatible UGREEN runtime is available)

Consult the [official UGREEN Hackathon Starter Kit](https://ai.ugreen.com/pages/hackathon-starter-kit) and its installation and validation guidance. Give the user-visible Skill only explicitly approved, accessible folders. This repository **does not currently provide a proven NAS installation or runtime procedure**; do not interpret the static pass as a successful device test.

## Round 2 validation priorities

- Verify `file-agent` delegation response shape, exhaustive-listing signals and `limit` behavior.
- Determine whether an authorized share from another NAS is visible.
- Test symlink reporting, changes during inspection, large folder sets and partial tool responses.
- Exercise an actual Uliya model against misleading tool summaries, incomplete evidence and file-name prompt injections.

## Source and authorship

This prototype was prepared for the UGREEN AI NAS Hackathon 2026. OpenAI ChatGPT assisted with the initial Skill instructions, proposal and test-case drafting. The submitted Skill contains Markdown instructions and references, no executable components or third-party datasets.

**No software license has been selected yet.** Public visibility does not itself grant reuse rights.

## Important disclosure

The published rules and validation report describe a **proposal-stage, statically checked prototype**. Nothing here claims UGREEN approval, live-device verification, a business partnership, or universal cross-NAS compatibility.
