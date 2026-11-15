# Report format

Produce a concise, inspectable migration-verification receipt with these exact sections. Derive all numbers from actual expert-returned evidence; if unknown, say "unknown".

1. **Scope:** approved source and destination roots (display minimally), selected path/file filters, explicit exclusions; confirm read-only audit.
2. **Audit status:** one of VERIFIED / MISMATCH / INCOMPLETE / BLOCKED, always qualified as *file-content scope only*.
3. **Inventory:** number of observed source/destination file occurrences; number of directories proven complete; number incomplete/unknown; exact listing bounds and coverage evidence.
4. **Content evidence:** counts of exact hash matches, relocated candidates, confirmed differences/missing content, ambiguous candidate groups, unreadable/unstable items. Attach or show enough real paths, sizes, and hashes to reproduce a focused sample. Do not print confidential content.
5. **Exceptions:** enumerate path and reason for every mismatch, permission error, limit hit, unstable source, or incomplete subtree. If many, present the count and a bounded sample while retaining the incomplete status.
6. **Assurance boundary:** say clearly that file-content identity does not guarantee applications, permission inheritance, metadata, snapshots, external source access, or recoverability.
7. **Next steps:** only specific user-actionable instructions for unresolved items. State whether another audit is required before retiring the old NAS.
8. **Save receipt:** state NOT_REQUESTED, AWAITING_APPROVAL, SAVED (with actual expert receipt), or SAVE_FAILED; never infer SAVED from an attempted request.

Optional text persistence is an independent, user-approved action after report review. Failure to save must not retroactively change the underlying file-inspection observations.
