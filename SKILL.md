---
name: switchproof
description: Read-only file migration audit with an evidence-linked report of verified, mismatched, incomplete, and blocked files for two user-authorized NAS file trees. Use when a user asks whether files copied from a previous NAS to a UGREEN NAS arrived intact.
---

# SwitchProof - Cross-NAS Migration Confidence Auditor

# Goal

## When to use

Use when a user asks whether files copied from a previous NAS to UGREEN AI NAS arrived intact, or asks for a read-only comparison of two user-authorized file trees. This Skill checks *evidence*, not the success status reported by a copy program. Its default scope is files and their bytes, not a complete application or NAS configuration migration.

## User-visible result

Provide a source/destination comparison report with: the user-approved roots and included/excluded scope; observed file coverage; per-file SHA-256 evidence where available; missing, changed, relocated, ambiguous, and unexamined records; reasons preventing a verified conclusion; and practical next steps. A successful read-only audit never modifies original files.

# Required input

1. A user-selected source root and destination root, both actually accessible to authorized UGOS file operations. Obtain real accessible paths from the user or file-agent; never invent paths or assume third-party SMB shares are mounted or visible.
2. A precise comparison scope: included folders/file types and any explicit exclusions. Ask for clarification if scope is ambiguous.
3. Whether a user-approved report should be saved; display results in chat by default.

Do not request credentials, cloud tokens, remote NAS admin access, or private keys. Do not broaden the requested scope on failure. Do not process symlinks or external files outside the approved roots.

# Generation-time capability evidence

- `ug-filesystem` → `file-agent`
- `ug-knowledge` → `knowledge-agent`

Only the registered experts may execute these capabilities. The parent Skill only delegates and must never use `cli.invoke`, shell commands, scripts, containers, or undocumented APIs.

## Supported actions and boundaries

- `ug-filesystem` `ls`/`find`: discover returned real paths, subdirectories, and available IDs. Listing is bounded by tool capabilities; no automatic assertion of exhaustiveness.
- `ug-filesystem` `stat`: obtain real file size/type/timestamps, before and after hashing where possible.
- `ug-filesystem` `sha256sum`: independently obtain source and destination content digests.
- `ug-filesystem` `getfattr`: optionally record available extended attributes; absence is not proof that metadata was preserved.
- `ug-knowledge` `save-text`: optionally persist a report with explicit user approval of content and destination; waiting approvals must finish before claiming it saved.

These are all selected from the published capability catalog. The Skill is read-only unless the user separately requests and approves the optional report save. No file copy, delete, move, overwrite, migration, NAS configuration, or off-device upload is performed.

# Orchestration

Mandatory ordered gates. Pass only the real results returned by each upstream step into downstream delegations; if a required real result is missing, stop the dependent step. Each instruction is one shell single-quoted string: when a substituted path or report text contains a single quote, write it as '"'"' so the instruction stays intact.

### Gate 0: Scope and access preflight

- Capture the exact two user-authorized roots and scope. Reject identical roots, nested or overlapping roots (one root inside the other, which would let a tree be compared with itself), inaccessible roots, ambiguous mappings, or any request to include third-party locations without a verified path and permission.
- Delegate a bounded `ls`/`stat` preflight to `file-agent`. Replace each angle-bracket value with the exact root the user approved in this conversation before sending; if either value is not available, do not send and mark BLOCKED. The expert performs actual UGOS operations.

  `delegate --target file-agent --instruction 'Read-only preflight for a file migration audit. Source root: <approved source root>. Destination root: <approved destination root>. For each root, run stat and a bounded non-recursive ls, and return the real path, whether it is accessible, and its object type. Do not copy, move, rename, delete, or modify anything. Do not follow symlinks or read outside these two roots. If a root is missing or inaccessible, report that fact for that root.'`
- Require returned real path, access result, and object type for **both** roots before proceeding. If either is missing, mark BLOCKED. Do not claim to have connected to a competitor NAS.

### Gate 1: Enumerate without losing files

- Delegate exact, bounded, non-recursive `ls` work per observed directory through `file-agent`, preserving the precise path returned in each response. Use actual directory entries returned by the upstream listing to construct child tasks; do not extrapolate names.

  `delegate --target file-agent --instruction 'Read-only listing. Directory: <real directory path returned by the previous listing>. Run a non-recursive ls with limit <requested limit>. Return every entry with its real path, name, type, and size, the number of entries returned, and whether the limit was reached. Do not follow symlinks, do not read outside the approved roots, and do not modify anything. If the directory is empty, say so explicitly.'`
- Track each directory as COMPLETE, UNSURE_LIMIT, ACCESS_DENIED, or ERROR. If the count reaches the requested limit or the expert cannot establish the complete listing, flag UNSURE_LIMIT; do **not** infer there are no additional files. `find` may help locate scoped candidates, but its bounded result alone is not proof of completeness.
- Use only observed, in-scope non-symlink items. Never follow paths outside authorized roots. A zero-item result is a genuine empty result, not a reason to search other locations.
- Proceed with item-level comparisons only for actually enumerated files. Track unenumerated directory regions as INCOMPLETE; never compute a misleading 100% coverage denominator.

### Gate 2: Collect independent evidence

- For each observed in-scope file, ask `file-agent` to obtain `stat` before, `sha256sum`, and `stat` after. Preserve the exact observed path and all actual tool-returned values for each side.

  `delegate --target file-agent --instruction 'Read-only evidence collection. Files: <real file paths returned by the listing step>. For each file, in order: run stat, run sha256sum, then run stat again. Return the real path, both stat results, and the SHA-256 digest, or the exact failure for that file. Do not modify, move, or copy any file and do not read outside the approved roots.'`
- If the before/after metadata suggests a file changed, or the hash is absent or fails, mark the item UNSTABLE or UNREADABLE; do not assume a valid immutable snapshot. Identical metadata alone cannot prove no concurrent change, so disclose this limitation.
- Use bounded batches and keep counted outcomes. A completed delegate request without the required stat/hash data does not imply success.

### Gate 3: Reconcile source and destination

- First match content by SHA-256 digest and size; same digest is *content evidence* only. Use observed relative path as additional evidence, not as the sole proof.
- Preserve multiplicities: two distinct source paths with identical bytes are **two file occurrences**, not one migrated file. One destination occurrence cannot silently satisfy multiple expected occurrences.
- Classify exact path + hash matches as EXACT_CONTENT_MATCH; a changed relative path with digest match as RELOCATED_CONTENT_CANDIDATE; absent digest as MISSING_CONTENT; same path with different digest as DIFFERENT_CONTENT; multiple possible mappings as AMBIGUOUS_MATCH.
- An identical hash does not prove preservation of ACLs, ownership, NAS application settings, photo album structure, or access rights. If the user requested those guarantees, say they are out of scope.

### Gate 4: Independent completion audit

- Apply the decision rules in [Verification policy](references/verification-policy.md). Do not accept a delegate `completed` status without actual data, directory coverage evidence, and reconciliation counts.
- Mark `VERIFIED` only for the declared file-content scope when all directory listings are demonstrably complete, at least one in-scope source file occurrence was observed, every required file occurrence is accounted for, hashes are available and consistent, and no unresolved instability or ambiguity remains.
- If verified differences exist, report `MISMATCH`; if some file regions were not examined, report `INCOMPLETE`; if prerequisites or access are unavailable, report `BLOCKED`. If multiple conditions apply, disclose each and use the conservative top-level result defined in the verification policy.
- Never say the full NAS migration, backup recovery, permissions, or app configuration is verified based on file hashes alone.

### Gate 5: Evidence-linked report and optional save

- Use [Report format](references/report-format.md) to disclose exact observed counts, exceptions, access limitations, and what can and cannot be concluded.
- Show the report to the user. To save it, ask for explicit confirmation of its content and destination, then delegate the save to `knowledge-agent` (`ug-knowledge` `save-text`). Replace the angle-bracket values with the complete approved report text and the exact save directory or .md file the user gave; omit the destination sentence if the user did not give one.

  `delegate --target knowledge-agent --instruction 'Save this user-approved SwitchProof report as Markdown with filename hint SwitchProof report. Save destination given by the user: <approved save destination>. Report text: <complete approved report text>. Do not change the text. Return whether the save was verified and the real output file path.'`
- Wait through any `waiting_approval` status; neither a dispatch acknowledgment nor a natural-language "yes" is an approval receipt. If saving fails, still report the audit truthfully and state the report was not saved.

# Completion and failure

- `completed`: check actual task-specific outputs, not just status. Missing outputs => halt the dependent step and disclose the gap.
- `failed`: stop dependent work; preserve independently observed facts; disclose exact expert and failing step.
- `cancelled`: stop without covert retries, disclose incomplete audit.
- `waiting_user`: resume the same task after input; do not launch a substitute in parallel.
- `waiting_approval`: wait for platform-gateway approval; do not claim persistence before a real completion receipt.
- An empty authoritative discovery result => report it as empty; do not silently widen search or substitute similar files.

# Validation and disclosures

See [Validation cases](references/validation-cases.md). Round 1 package status: it passed local static checking with the official uliya-skill-kit 0.7.7 checker with 0 errors and 4 warnings (DELEGATE_DYNAMIC_VALUE_UNRESOLVED, caused by the runtime placeholder values in the delegate templates). It has not yet been tested on actual UGREEN NAS hardware. Upload, installation, and runtime compatibility remain unverified. Rule-level checks used only simulated delegation responses, not official Uliya runtime results. The external NAS share visibility and listing semantics need confirmation on the assigned device in Round 2.

Third-party components: none packaged. No external services, telemetry, or network transfer; only host-provided, authorized UGREEN operations. AI-assistance disclosure: OpenAI ChatGPT was used to draft the initial Skill instructions, proposal language, and test cases; it is not a deployed runtime dependency. Principal original contribution for this event: the SwitchProof migration verification orchestration, coverage gates, evidence policy, and reporting format. No third-party source code, model weights, or datasets are included.
