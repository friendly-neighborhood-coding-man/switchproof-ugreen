# Verification policy

This reference is normative for the parent Skill. It constrains claims; it is not an executable script.

## Completion invariant

A verified file-content migration **must** have (a) both approved accessible roots, (b) evidence that every in-scope directory was exhaustively enumerated, (c) a complete and stable set of file records, (d) a matching distinct destination occurrence for each expected source occurrence, and (e) no unresolved mismatch or ambiguity.

Do not confuse a listing bounded by `limit` with an exhaustive inventory. Because published `ls` and `find` do not promise pagination or total counts, a list that reaches its bound or lacks definitive completeness evidence may remain INCOMPLETE even if returned entries match perfectly. Recurse only into actual discovered subdirectories. A user-defined finite subset can be verified as a scoped set; an unbounded tree may be unprovable with current capabilities.

## Per-file outcomes

- `EXACT_CONTENT_MATCH`: same approved relative path, same size and SHA-256, stable-enough evidence.
- `RELOCATED_CONTENT_CANDIDATE`: different path, matching size/hash, one-to-one occurrence match; warn that path and structure were not preserved.
- `DIFFERENT_CONTENT`: same mapped relative path, different reliable SHA-256.
- `MISSING_CONTENT`: source file has no appropriate matching destination occurrence among confirmed complete listings.
- `AMBIGUOUS_MATCH`: several competing candidates or duplicate multiplicities prevent a one-to-one conclusion.
- `UNREADABLE`: required stat or hash missing/failed.
- `UNSTABLE`: modification indicated during inspection or consistent snapshot not established.
- `NOT_ENUMERATED`: listing truncation, unreadable subtree, or unscanned region.

Never infer a negative (missing) from an incomplete destination inventory; report `NOT_ENUMERATED` instead. Do not conflate changed file names with changed content.

## Top-level status precedence

1. `BLOCKED` when a mandatory prerequisite or root access is unavailable and the intended audit cannot start. Preserve already observed facts.
2. `INCOMPLETE` when coverage cannot be established, an item remains unreadable/unstable, or a mapping is ambiguous. Disclose any confirmed differences alongside it.
3. `MISMATCH` when coverage is complete and there is at least one confirmed missing or changed content occurrence.
4. `VERIFIED` **only** when the complete declared file-content scope is supported by the evidence and no issue remains. Never label a full NAS migration VERIFIED. An in-scope source set with zero observed file occurrences is never `VERIFIED`; report `INCOMPLETE` with an empty-scope finding so the user can confirm the roots and scope.

These are user-facing statuses defined by SwitchProof, not official UGOS delegate statuses. If multiple conditions occur, keep all underlying issue labels in the report.

## Security and operational rules

- Read-only by default; never delete, move, overwrite, or attempt to repair files in this Skill.
- Never follow symlinks or cross outside approved roots; do not ingest credentials or remote admin tokens.
- Never invent a tool output, file count, file path, hash, directory boundary, or confirmed completion.
- A digest match is content evidence and does not guarantee a recoverable application, preserved permissions, or safety of deleting the old NAS.
- Expose sampling, explicit exclusions, time of inspection, and list/query limits in the report.
