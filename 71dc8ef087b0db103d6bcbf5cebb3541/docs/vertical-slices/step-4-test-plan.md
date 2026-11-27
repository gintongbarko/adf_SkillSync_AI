# Step 4 PDI test plan

Run this after Step 3 passes.

## Setup

1. Confirm the Step 4 fields exist on Internship Application:
   - `u_application_status` with choices `submitted`, `under_review`, `accepted`, `rejected`
   - `u_review_note`, `u_reviewed_by`, `u_reviewed_at`
2. Confirm these records are active:
   - `Set Submitted Application Status`
   - `Record Application Review`
   - `Recruiter Review` widget
   - `/recruiter_review` page and widget instance
   - `Accept Applicant` flow
3. Confirm `x_2311790_skills_0.recruiter` exists and is assigned to at least one test recruiter.
4. Confirm an admin test user and a student test user are available.
5. Create or confirm an internship with structured requirements and at least two scored applications from Step 3.

## Lifecycle defaults

| # | Action | Expected result |
|---:|---|---|
| 1 | Create an Internship Application without setting status. | `u_application_status=submitted`; `u_reviewed_by`, `u_reviewed_at`, and `u_review_note` are empty. |
| 2 | Create an application and explicitly set `u_application_status=under_review`. | The value is preserved; the default rule does not overwrite it. |
| 3 | Change only `u_match_score` on a submitted application. | `u_reviewed_by` and `u_reviewed_at` remain empty. |

## Portal access

| # | Action | Expected result |
|---:|---|---|
| 1 | Open `/recruiter_review` while signed out. | Portal redirects to login or denies access; no applicant data is visible. |
| 2 | Open `/recruiter_review` as a student-only user. | Widget shows `Your account is not allowed to review applicants.` |
| 3 | Open `/recruiter_review` as the test recruiter. | Page loads; internship selector is populated; ranked table appears for the default internship. |
| 4 | Open `/recruiter_review` as an admin. | Page loads with the same data access. |

## Ranked review UI

| # | Action | Expected result |
|---:|---|---|
| 1 | Select a different internship. | Table refreshes and shows only that internship's top `x_2311790_skills_0.matching_top_n` applicants. |
| 2 | Click Refresh. | Data reloads without losing the selected internship. |
| 3 | Compare the table with `MatchingEngine().rankApplicants('<internship sys_id>')`. | Same order, ranks, scores, matched skills, and missing requirements. |
| 4 | Open an application whose student has no Student Profile. | It is absent from the widget list, consistent with `rankApplicants()`. |
| 5 | Inspect an application with a match error. | The widget omits it from the ranked list rather than showing an unscored row. |

## Review actions

| # | Action | Expected result |
|---:|---|---|
| 1 | Enter a note and click Review. | Status becomes `under_review`; note is saved; `u_reviewed_by` is the recruiter; `u_reviewed_at` is set. |
| 2 | Click Accept. | Status becomes `accepted`; note is saved; acceptance email is sent once. |
| 3 | Click Reject. | Status becomes `rejected`; note is saved; no acceptance email is sent. |
| 4 | Try to click Review/Accept/Reject on an already accepted or rejected row. | Buttons are disabled; status does not change. |
| 5 | Submit a forged `set_status` request against an accepted or rejected application. | Server returns `Application is already in a terminal status.` and nothing is written. |
| 6 | Submit a forged `set_status` request with an invalid status. | Server returns `Invalid application status.` and nothing is written. |
| 7 | Submit a forged `set_status` request for an application sys_id that does not exist. | Server returns `Application was not found.` |

## Retargeted email flow

| # | Action | Expected result |
|---:|---|---|
| 1 | Set an application's status to `accepted` through the widget. | `Accept Applicant` runs once in the background and sends the email to `u_student.email`. |
| 2 | Set another application's inherited `state` to `3` / `Closed Complete` without changing `u_application_status`. | No acceptance email is sent. |
| 3 | Set status to `under_review`, then `accepted`. | Email is sent only on the transition to `accepted`. |
| 4 | Set status to `accepted` twice using a direct update that does not change the value. | No second email, because the trigger uses `CHANGESTO`. |
| 5 | As the student user, try to open the portal page and inspect network/server logs. | No unauthorized review data is returned. |

## Regression checks

1. Step 1 upload still accepts PDF only and writes `Parsed`, `Failed`, or `Pending`.
2. Step 2 still writes max 10 resume-derived skill rows with top 3 marked core.
3. Step 3 still scores, ranks, and persists match fields on application create/update.
4. The old `state=3` email trigger text no longer appears in the active `Accept Applicant` flow.
5. All XML imports cleanly; no duplicate sys_ids; no placeholder text remains.
6. `checksum.txt` is expected to be stale until the snapshot is regenerated from ServiceNow.
