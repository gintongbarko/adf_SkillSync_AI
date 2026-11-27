# Step 5 PDI test plan — RBAC and platform hardening

Run these tests after applying Steps 1–5 and setting the Gemini properties from the earlier test plans. Use at least these users:

- `studentA` with `x_2311790_skills_0.student` and `x_2311790_skills_0.student_profile_user`, owning Student Profile A.
- `studentB` with the same roles, owning Student Profile B.
- `recruiter` with `x_2311790_skills_0.recruiter`.
- `orgHead1` with `x_2311790_skills_0.org_head` and membership in Organization Group 1.
- `orgHead2` with the same role and membership in Organization Group 2.
- `skillAdmin` with `x_2311790_skills_0.admin`.

## A. Application and scope settings

| # | Test | Expected result |
|---|---|---|
| 1 | Open the SkillSync application record. | `Private`, `Restrict table access`, and `Runtime access tracking = Enforcing` are set. |
| 2 | Open Cross-Scope Privileges for SkillSync. | `api_key_credentials` is `Denied`; Gemini/REST/attachment privileges remain `Allowed`. |
| 3 | Run a resume parse and an embedding request as `studentA`. | Both still succeed; no cross-scope denial appears in System Logs. |
| 4 | Open the three SkillSync flows. | `Run As` is System User for all three; the two disabled flows remain inactive. |

## B. Student data isolation

| # | Test | Expected result |
|---|---|---|
| 5 | As `studentA`, open `/resume_upload`. | Profile A, parse status, skills, and tools are shown. |
| 6 | As `studentA`, query the Student Profile list/API for Profile B. | Profile B is not returned or is access-denied. |
| 7 | As `studentA`, try to read Profile B's `embedding`, `u_parsed_resume`, or `u_parse_error` by sys_id. | Access denied. |
| 8 | As `studentA`, read Profile A's own `embedding` and `u_parse_error` through the API. | Allowed; owner access works for automation. |
| 9 | As `studentA`, try to set Profile A's `u_user` to `studentB`. | Write denied by the `u_user` field ACL. |
| 10 | As `studentA`, try to create a Student Skill pointing at Profile B. | Create denied. |
| 11 | As `studentA`, create/edit/delete a Student Skill on Profile A. | Allowed. |
| 12 | As `studentA`, try to delete Profile A. | Delete denied; only admins can delete profiles. |

## C. Resume attachments

| # | Test | Expected result |
|---|---|---|
| 13 | As `studentA`, upload a PDF resume. | Upload succeeds; attachment is associated with Profile A. |
| 14 | As `studentA`, read/download the resume attachment. | Allowed. |
| 15 | As `studentB`, read/download the same attachment by sys_id. | Access denied. |
| 16 | As `recruiter`, read/download the attachment. | Allowed. |
| 17 | As `studentB`, write or delete the attachment. | Access denied. |
| 18 | As `skillAdmin`, delete a test attachment. | Allowed. |

## D. Internships and org heads

| # | Test | Expected result |
|---|---|---|
| 19 | Create Internship 1 with `u_org = Organization Group 1` and Internship 2 with `u_org = Organization Group 2`. | Records save. |
| 20 | As `orgHead1`, open the internship list. | Only Internship 1 is visible. |
| 21 | As `orgHead1`, edit Internship 1. | Allowed. |
| 22 | As `orgHead1`, try to read or edit Internship 2 by sys_id. | Access denied. |
| 23 | As `orgHead1`, create an internship with `u_org = Organization Group 2`. | Create denied. |
| 24 | As `studentA`, read Internship 1 while its status is `posted`. | Allowed. |
| 25 | As `studentA`, set Internship 1 status to `draft`, then try to read it. | Access denied. |
| 26 | As `recruiter`, read both internships and their embedding fields. | Allowed. |

## E. Applications and workflow

| # | Test | Expected result |
|---|---|---|
| 27 | As `studentA`, submit an application for Internship 1. | Create succeeds; `u_application_status = submitted`; match fields are populated. |
| 28 | As `studentB`, try to read that application by sys_id. | Access denied. |
| 29 | As `orgHead1`, read the application for Internship 1. | Allowed. |
| 30 | As `orgHead2`, try to read the application for Internship 1. | Access denied. |
| 31 | As `studentA`, PATCH/PUT `u_application_status = accepted` on their own application. | Write denied by the field ACL; status remains `submitted`. |
| 32 | As `studentA`, try to write `u_review_note` or `u_reviewed_by`. | Write denied. |
| 33 | As `recruiter`, open `/recruiter_review`, select Internship 1, and click Review. | Status changes to `under_review`; `u_reviewed_by` and `u_reviewed_at` are set. |
| 34 | As `recruiter`, click Accept. | Status changes to `accepted` and the acceptance email is sent. |
| 35 | Check the acceptance email flow context. | Flow ran as System User, not the recruiter. |
| 36 | Try to change an accepted application back to `submitted`. | Denied by widget/server logic and field ACL. |
| 37 | As `recruiter`, try to write a Student Profile's `embedding` directly. | Write denied. |

## F. Portal role experience

| # | Test | Expected result |
|---|---|---|
| 38 | Open `/ss_internships` as `skillAdmin`. | Header reads `Hello, SkillSync Administrator.` and includes Recruiter Review. |
| 39 | Open `/ss_internships` as `recruiter`. | Header reads `Hello, Recruiter.` and includes Recruiter Review. |
| 40 | Open `/ss_internships` as `orgHead1`. | Header reads `Hello, Org Head.` and includes Internships. |
| 41 | Open `/ss_internships` as `studentA`. | Header reads `Hello, Student.` and includes Upload Resume. |
| 42 | Open `/upload_resume` and `/resume_upload` as `orgHead1`. | Access denied by page roles. |
| 43 | Open `/recruiter_review` as `studentA`. | Access denied. |
| 44 | Confirm the two orphaned instances are inactive. | They do not render on any page. |
| 45 | Open the default Student Profile form/list. | `u_parsed_resume` is not displayed; other sensitive internals remain hidden. |

## G. Regression checks

| # | Test | Expected result |
|---|---|---|
| 46 | Repeat a Step 1 resume parse. | Parse status and parsed skills still update correctly. |
| 47 | Repeat a Step 2 manual skill edit. | Manual skill rows are preserved and still visible to the owner. |
| 48 | Repeat a Step 3 application submission. | Matching score, rank, rationale, matched skills, and missing requirements populate. |
| 49 | Repeat a Step 4 rejection. | Status becomes `rejected`; no acceptance email is sent. |
| 50 | Search System Logs for ACL denials during the positive tests. | No unexpected denials for allowed workflows. |

## Known limitations

- Org-head visibility depends on `u_org` being populated and the org head belonging to that group.
- Field ACLs allow the owner to write parse/embedding internals so current automation keeps working; this is not a separate service-identity model.
- `checksum.txt` remains stale until the source snapshot is regenerated from ServiceNow.
