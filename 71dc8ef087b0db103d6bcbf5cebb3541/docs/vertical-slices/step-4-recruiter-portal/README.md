# Step 4 — Recruiter portal and workflow

Status: implemented in source; requires PDI testing.

## Decisions applied

- The application lifecycle uses a custom `u_application_status` choice field on Internship Application. The inherited `task.state` field is left untouched.
- Recruiters review applicants in a dedicated Service Portal widget at `/recruiter_review`.
- Accept, Reject, and Mark Under Review are direct widget server actions; they are not Approval Engine records.
- The existing `Accept Applicant` flow now triggers on `u_application_status` changing to `accepted` instead of `state=3`.
- Minimal role gating is included now: the page, widget, widget instance, and server script allow `x_2311790_skills_0.admin` and the new `x_2311790_skills_0.recruiter` role. Full table/field ACL hardening remains in Step 5.
- No new service identity was added. The retargeted email flow keeps `run_as=user`, so it runs as the recruiter whose status change triggered it. Service-account hardening is deferred to Step 5.

## Runtime flow

1. An Internship Application is created.
2. `Set Submitted Application Status` and the field default ensure `u_application_status=submitted`.
3. Step 3 rules score, rank, and persist the match fields.
4. A recruiter opens `/recruiter_review`, selects an internship, and the widget calls `MatchingEngine.rankApplicants()` for a live top-N shortlist.
5. The table shows rank, score, status, matched skills, missing requirements, and rationale.
6. The recruiter enters an optional note and clicks Review, Accept, or Reject.
7. The widget writes `u_application_status` and `u_review_note`.
8. `Record Application Review` sets `u_reviewed_by` and `u_reviewed_at` when the status changes away from `submitted`.
9. When the status becomes `accepted`, `Accept Applicant` sends the existing acceptance email.

## Lifecycle fields

| Field | Type | Purpose |
|---|---|---|
| `u_application_status` | Choice | `submitted`, `under_review`, `accepted`, `rejected`; default `submitted` |
| `u_review_note` | String (2000) | Optional recruiter note |
| `u_reviewed_by` | Reference to `sys_user` | User who moved the application out of `submitted` |
| `u_reviewed_at` | Date/Time | Time of that status change |

## Widget behavior

The `Recruiter Review` widget:

- Lists internships ordered by short description.
- Defaults to the first internship when none is selected.
- Refreshes on internship selection and on the Refresh button.
- Uses `MatchingEngine.rankApplicants()`, so the shortlist honors `x_2311790_skills_0.matching_top_n` (default 5).
- Shows the applicant name, rank, score, current status label, matched skills, missing requirements, and rationale.
- Disables Review/Accept/Reject once an application is `accepted` or `rejected`.
- Rejects status changes to or from `accepted`/`rejected` in the server script, so forged requests cannot bypass the disabled buttons.
- Re-checks roles in the server script even though the page and widget are role-gated.

## Business rules

| Rule | Artifact | Timing |
|---|---|---|
| Set Submitted Application Status | `update/sys_script_e100000587b0db103d6bcbf5cebb3505.xml` | before insert |
| Record Application Review | `update/sys_script_e100000687b0db103d6bcbf5cebb3506.xml` | before update |

## Portal artifacts

| Artifact | Purpose |
|---|---|
| `update/sp_page_e100000987b0db103d6bcbf5cebb3509.xml` | `/recruiter_review` page |
| `update/sp_container_e100000a87b0db103d6bcbf5cebb350a.xml` | Page container |
| `update/sp_row_e100000b87b0db103d6bcbf5cebb350b.xml` | Row |
| `update/sp_column_e100000c87b0db103d6bcbf5cebb350c.xml` | Full-width column |
| `update/sp_instance_e100000d87b0db103d6bcbf5cebb350d.xml` | Widget instance |
| `update/sp_widget_e100000887b0db103d6bcbf5cebb3508.xml` | Widget, scripts, template, and CSS |

## Email flow change

Artifact: `update/sys_hub_flow_0b76127487f0db103d6bcbf5cebb35b8.xml`

The record trigger condition changed from:

```text
state CHANGESTO 3 (Closed Complete)
```

to:

```text
u_application_status CHANGESTO accepted
```

The email action inputs are unchanged; they still use the applicant email and internship short description from the trigger record.

## Explicitly out of scope for Step 4

- Approval Engine records and multi-step approval chains.
- Table/field ACLs and role inheritance cleanup (Step 5).
- Dedicated service account for background writes (Step 5).
- Internship posting/editing UI; internships continue to be maintained through the existing table/form.

## Validation

Run `docs/vertical-slices/step-4-test-plan.md` in the PDI before Step 5.
