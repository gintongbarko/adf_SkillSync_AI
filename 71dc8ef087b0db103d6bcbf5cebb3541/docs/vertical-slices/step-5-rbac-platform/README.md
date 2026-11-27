# Step 5 — RBAC and platform hardening

Status: implemented in source; requires PDI testing.

## Decisions applied

- Row ownership uses existing fields only. Student Profile uses `u_user`; Student Skill uses `u_profile.u_user`; Internship Application uses `u_student` plus its internship's `u_org`; Internship uses `u_org`.
- Org heads are identified by `x_2311790_skills_0.org_head` plus membership in the internship's `u_org` group. No new schema field was added.
- Students can read and edit only their own profile and skill rows. Recruiters can read candidate data but do not own student data. Administrators retain full CRUD.
- Org heads can read and maintain only internships in their own `u_org` group and applications for those internships.
- Sensitive profile fields are ACL-enforced. Owners, recruiters, and administrators can read them; owners and administrators can write the fields that parsing/embedding automation updates. `u_user` is administrator-write-only.
- Internship embedding fields are readable by administrators, recruiters, owning org heads, and students viewing posted internships, so matching still works when an applicant triggers scoring.
- `u_application_status` can be defaulted on create by the applicant, recruiter, or administrator, but changed after creation only by recruiters or administrators. Review metadata is recruiter/admin-write-only.
- Resume attachments on Student Profile are readable by the owner, recruiter, and administrator. Create/write/delete is limited to the owner and administrator.
- Scope settings are tightened: `private=true`, `restrict_table_access=true`, and `runtime_access_tracking=enforcing`.
- The unused `api_key_credentials` cross-scope privilege is denied. All privileges actually used by parsing, embeddings, attachments, and REST calls remain allowed.
- All three flows now use `run_as=system`. No persistent service account record was added; record-triggered Flow Designer flows support User/System User, not a custom service identity.
- Portal pages and instances are role-gated. The dashboard now renders a role-aware `Hello, [role]` header, and orphaned instances are inactive rather than deleted.
- `u_parsed_resume` was removed from the default Student Profile form and list and is enforced by field ACLs.

## Row ACL matrix

| Table | Read | Create | Write | Delete |
|---|---|---|---|---|
| Student Profile | Owner, recruiter, admin | Admin | Owner, admin | Admin |
| Student Skill | Owner, recruiter, admin | Owner, admin | Owner, admin | Owner, admin |
| Skill Master | All configured app roles | Admin, skill master user | Admin, skill master user | Admin, skill master user |
| Internship | Admin, recruiter, org-head owner, student for posted rows | Admin, org-head owner | Admin, org-head owner | Admin |
| Internship Application | Admin, recruiter, student owner, org-head owner | Admin, student owner | Admin, recruiter, student owner, org-head owner | Admin |
| sys_attachment (SkillSync profiles) | Admin, recruiter, owner | Admin, owner | Admin, owner | Admin, owner |

Every ACL still requires a mapped role. The script is an additional row/field condition; it does not replace role membership.

## Field ACLs

| Area | Fields | Behavior |
|---|---|---|
| Student Profile internals | `embedding`, `u_parsed_resume`, `u_parse_error`, `u_parse_attempts`, `u_parsed_attachment_sys_id`, `u_embedding_model`, `u_embedding_version`, `u_embedded_at`, `u_embedding_dim`, `u_embedding_error`, `u_embedded_attachment_sys_id` | Read: owner, recruiter, admin. Write: owner, admin. |
| Student Profile ownership | `u_user` | Write: admin only. |
| Internship embedding internals | `u_embedding`, `u_embedding_model`, `u_embedding_version`, `u_embedded_at`, `u_embedding_dim`, `u_embedding_error` | Read: admin, recruiter, org-head owner, student viewing posted. Write: admin, org-head owner. |
| Application lifecycle | `u_application_status` | Create: admin, recruiter, applicant. Write after create: admin, recruiter. |
| Application review metadata | `u_reviewed_by`, `u_reviewed_at`, `u_review_note` | Write: admin, recruiter. |

## Runtime effects

1. A student uploads a resume. The upload widget and Step 1/Step 3 automation run as that student, and field ACLs allow the owner to write the parse/embedding fields on their own profile.
2. An applicant submits an application. The default status write is allowed on create; later status changes through the portal are restricted to recruiters and administrators.
3. A recruiter opens `/recruiter_review`. Role mappings and row ACLs permit reading candidate profiles, skills, and applications; field ACLs permit reading embeddings and parsed resume data.
4. A recruiter accepts an application. The field ACL permits the status change, the review business rule writes audit metadata, and `Accept Applicant` runs as System User to send the email.
5. An org head opens the dashboard. The greeting shows `Hello, Org Head.` and links to `/internships`; ACLs limit visible internships and applications to their `u_org` group.
6. Users without SkillSync roles cannot open the gated portal pages.

## Portal changes

| Artifact | Change |
|---|---|
| `update/sp_widget_e1000040000000000000000000000001.xml` | New `SkillSync Role Header` widget with server-side role detection and role-specific links. |
| `update/sp_column_e1000040000000000000000000000002.xml` | New full-width column on the dashboard's first row. |
| `update/sp_instance_e1000040000000000000000000000003.xml` | Places the role header on `/ss_internships`. |
| `sp_page_3b7852f887f0db103d6bcbf5cebb35ad.xml` | Dashboard page role-gated. |
| `sp_page_5c7b56b08734db103d6bcbf5cebb3542.xml`, `sp_page_a0cd56b08734db103d6bcbf5cebb3553.xml` | Upload pages role-gated for student/admin roles. |
| `sp_page_9fc752b887f0db103d6bcbf5cebb35ca.xml` | Internships page role-gated. |
| `sp_instance_8b8ed2388734db103d6bcbf5cebb35fa.xml` | Resume upload instance role-gated. |
| `sp_instance_23cb52f08734db103d6bcbf5cebb35ca.xml`, `sp_instance_37e796b887f0db103d6bcbf5cebb355f.xml` | Orphaned instances set inactive. |

The empty `upload_resume` and `internships` pages are retained but gated; they cause no data exposure. The orphaned `My Profile` instance still references a widget that is not in the snapshot, so it is disabled rather than repaired.

## Flows and privileges

| Artifact | Change |
|---|---|
| `sys_app_71dc8ef087b0db103d6bcbf5cebb3541.xml` | Private app, restricted table access, enforcing runtime access tracking. |
| `update/sys_scope_privilege_3acbfe7c87f4db103d6bcbf5cebb35aa.xml` | `api_key_credentials` read privilege set to `denied`. |
| `update/sys_hub_flow_0b76127487f0db103d6bcbf5cebb35b8.xml` | Active acceptance email flow runs as System User. |
| `update/sys_hub_flow_13d13af887b4db103d6bcbf5cebb358e.xml` | Disabled parse flow also set to System User. |
| `update/sys_hub_flow_6049eef88774db103d6bcbf5cebb3569.xml` | Disabled pending-status flow also set to System User. |

## Explicitly out of scope for Step 5

- Role inheritance changes; roles remain flat.
- Approval Engine records.
- A dedicated service user record. Flow Designer record-triggered flows use System User instead.
- Rebuilding the empty portal pages or restoring the missing `My Profile` widget.
- Regenerating `checksum.txt`; it remains stale because the generated XML was edited locally.

## Validation

Run `docs/vertical-slices/step-5-test-plan.md` in the PDI before treating the app as production-ready.
