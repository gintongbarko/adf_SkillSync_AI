# Vertical slice developer map

The ServiceNow-generated folders are intentionally left in place:

```text
update/
dictionary/
author_elective_update/
checksum.txt
```

They must stay in their generated locations because ServiceNow source control expects those paths and checksums. Moving or renaming those files would break import/source-control validation.

This folder is the developer-owned architecture view. It groups the generated artifacts by product slice so the project can be understood and extended incrementally without rearranging the ServiceNow export.

```text
docs/vertical-slices/
├── README.md                         # This file
├── artifact-map.md                   # Generated artifact -> vertical slice mapping
├── step-1-resume-intake/
│   └── README.md                     # Implemented Step 1 behavior and contracts
├── step-2-structured-skills/
│   └── README.md                     # Implemented Step 2 behavior and contracts
├── step-3-embeddings-matching/
│   └── README.md                     # Implemented Step 3 behavior and contracts
├── step-4-recruiter-portal/
│   └── README.md                     # Implemented Step 4 behavior and contracts
├── step-5-rbac-platform/
│   └── README.md                     # Implemented Step 5 behavior and contracts
├── step-1-test-plan.md               # Step 1 PDI test matrix
├── step-2-test-plan.md               # Step 2 PDI test matrix
├── step-3-test-plan.md               # Step 3 PDI test matrix
├── step-4-test-plan.md               # Step 4 PDI test matrix
└── step-5-test-plan.md               # Step 5 PDI test matrix
```

## Rules for future work

1. Work one slice at a time.
2. Do not depend on a later slice from an earlier one.
3. Keep generated XML in the ServiceNow export folders.
4. Update `artifact-map.md` when adding or retiring records.
5. Treat `checksum.txt` as a generated artifact: refresh it from ServiceNow after the equivalent changes are made in the instance.

## Current implementation status

| Slice | Status |
|---|---|
| Step 1 — Resume intake and parsing | Implemented in source; requires PDI testing |
| Step 2 — Structured skills extraction | Implemented in source; requires PDI testing |
| Step 3 — Embeddings and matching | Implemented in source; requires PDI testing |
| Step 4 — Recruiter portal and workflow | Implemented in source; requires PDI testing |
| Step 5 — RBAC and platform hardening | Implemented in source; requires PDI testing |
