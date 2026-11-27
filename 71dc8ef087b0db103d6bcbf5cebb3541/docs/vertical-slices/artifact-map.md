# Artifact map

This map is a developer view only. It does not move generated ServiceNow files.

## Step 1 — Resume intake and parsing

| Concern | Generated artifact |
|---|---|
| Canonical Gemini parser | `update/sys_script_include_dc8e36f08738db103d6bcbf5cebb35a8.xml` |
| Parse orchestration and status writes | `update/sys_script_f3de32348738db103d6bcbf5cebb35e8.xml` |
| Pending trigger from PDF attachment insert | `update/sys_script_c2f4a0d887b0db103d6bcbf5cebb3502.xml` |
| Upload widget, PDF-only validation, status display | `update/sp_widget_278a9a308734db103d6bcbf5cebb35ea.xml` |
| Gemini API key property | `update/sys_properties_5e9cfabc87f4db103d6bcbf5cebb3536.xml` |
| Gemini model property | `update/sys_properties_b7e1c0f887b0db103d6bcbf5cebb3501.xml` |
| Gemini endpoint override property | `update/sys_properties_a8f2d1e887b0db103d6bcbf5cebb3506.xml` |
| Profile fields | `dictionary/x_2311790_skills_0_student_profile.xml` |
| Profile field definitions | `update/sys_dictionary_x_2311790_skills_0_student_profile_*.xml` |
| Parse status choices | `update/sys_choice_x_2311790_skills_0_student_profile_u_parse_status.xml` (copied from `author_elective_update/` so `update/`-only promotion includes them) |
| Internal field hidden from default form | `update/sys_ui_section_ea956ab08774db103d6bcbf5cebb35c5.xml` |
| Internal field hidden from default list | `update/sys_ui_list_x_2311790_skills_0_student_profile_null.xml` |
| Disabled broken flow | `update/sys_hub_flow_6049eef88774db103d6bcbf5cebb3569.xml` |
| Deleted unfinished parser | `update/sys_script_include_4e06baf087f4db103d6bcbf5cebb3505.xml` |
| Deleted unused REST message | `update/sys_rest_message_8a66b63487f4db103d6bcbf5cebb353c.xml` and related function/header files |

## Step 2 — Structured skills extraction

| Concern | Generated artifact |
|---|---|
| Structured Gemini parser | `update/sys_script_include_dc8e36f08738db103d6bcbf5cebb35a8.xml` |
| Structured skills writer | `update/sys_script_include_f100000e87b0db103d6bcbf5cebb350e.xml` |
| Structured parse orchestration | `update/sys_script_f3de32348738db103d6bcbf5cebb35e8.xml` |
| Skill master schema | `dictionary/x_2311790_skills_0_skill_master.xml` |
| Student skill schema | `dictionary/x_2311790_skills_0_student_skill.xml` |
| Profile schema with education/years/projects | `dictionary/x_2311790_skills_0_student_profile.xml` |
| New field definitions | `update/sys_dictionary_x_2311790_skills_0_*.xml` |
| New choice definitions | `update/sys_choice_x_2311790_skills_0_*.xml` |
| Corrected cap rule | `update/sys_script_2a355e3487f0db103d6bcbf5cebb3536.xml` |
| Read-only parsed skills UI | `update/sp_widget_278a9a308734db103d6bcbf5cebb35ea.xml` |
| Table roles | `update/sys_user_role_*skill*.xml` |

## Step 3 — Embeddings and matching

| Concern | Generated artifact |
|---|---|
| Embedding service | `update/sys_script_include_c100000587b0db103d6bcbf5cebb3505.xml` |
| Matching engine | `update/sys_script_include_c100000687b0db103d6bcbf5cebb3506.xml` |
| Profile embedding trigger | `update/sys_script_c100000787b0db103d6bcbf5cebb3507.xml` |
| Internship embedding trigger | `update/sys_script_c100000887b0db103d6bcbf5cebb3508.xml` |
| Application scoring trigger | `update/sys_script_c100000987b0db103d6bcbf5cebb3509.xml` |
| Application ranking trigger | `update/sys_script_c100002087b0db103d6bcbf5cebb3520.xml` |
| Profile embedding/version metadata | `update/sys_dictionary_x_2311790_skills_0_student_profile_u_embedding_*.xml` |
| Internship requirements and embedding metadata | `update/sys_dictionary_x_2311790_skills_0_internship_*.xml` |
| Application score/rank fields | `update/sys_dictionary_x_2311790_skills_0_internship_application_*.xml` |
| Embedding and matching properties | `update/sys_properties_c100000*.xml` |
| Profile schema | `dictionary/x_2311790_skills_0_student_profile.xml` |
| Internship schema | `dictionary/x_2311790_skills_0_internship.xml` |
| Application schema | `dictionary/x_2311790_skills_0_internship_application.xml` |

## Step 4 — Recruiter portal and workflow

| Concern | Generated artifact |
|---|---|
| Application lifecycle schema | `dictionary/x_2311790_skills_0_internship_application.xml` |
| Application status field and choices | `update/sys_dictionary_x_2311790_skills_0_internship_application_u_application_status.xml`, `update/sys_choice_x_2311790_skills_0_internship_application_u_application_status.xml` |
| Review metadata fields | `update/sys_dictionary_x_2311790_skills_0_internship_application_u_reviewed_by.xml`, `update/sys_dictionary_x_2311790_skills_0_internship_application_u_reviewed_at.xml`, `update/sys_dictionary_x_2311790_skills_0_internship_application_u_review_note.xml` |
| Default status rule | `update/sys_script_e100000587b0db103d6bcbf5cebb3505.xml` |
| Review audit rule | `update/sys_script_e100000687b0db103d6bcbf5cebb3506.xml` |
| Recruiter role | `update/sys_user_role_e100000787b0db103d6bcbf5cebb3507.xml` |
| Recruiter review widget | `update/sp_widget_e100000887b0db103d6bcbf5cebb3508.xml` |
| Recruiter page and layout | `update/sp_page_e100000987b0db103d6bcbf5cebb3509.xml`, `update/sp_container_e100000a87b0db103d6bcbf5cebb350a.xml`, `update/sp_row_e100000b87b0db103d6bcbf5cebb350b.xml`, `update/sp_column_e100000c87b0db103d6bcbf5cebb350c.xml`, `update/sp_instance_e100000d87b0db103d6bcbf5cebb350d.xml` |
| Retargeted acceptance email flow | `update/sys_hub_flow_0b76127487f0db103d6bcbf5cebb35b8.xml` |
| Existing internship/application tables | `dictionary/x_2311790_skills_0_internship*.xml` |
| Existing resume upload widget | `update/sp_widget_278a9a308734db103d6bcbf5cebb35ea.xml` |

## Step 5 — RBAC and platform hardening

| Concern | Generated artifact |
|---|---|
| Row-scoped table ACLs | The existing 20 `update/sys_security_acl_*.xml` files, now advanced scripts with row ownership conditions |
| ACL role mappings | Role mapping records in the existing `update/sys_security_acl_role_*.xml` files and appended to each ACL file |
| Student Profile field ACLs | `update/sys_security_acl_e1000020*.xml` for `embedding`, parse internals, raw resume fields, and `u_user` |
| Internship field ACLs | `update/sys_security_acl_e1000020*.xml` for embedding internals |
| Application field ACLs | `update/sys_security_acl_e1000020*.xml` for `u_application_status` and review metadata |
| Attachment ACLs | `update/sys_security_acl_e1000030*.xml` for create/read/write/delete on SkillSync resume attachments |
| Scope settings | `sys_app_71dc8ef087b0db103d6bcbf5cebb3541.xml` — private, restricted table access, enforcing runtime tracking |
| Denied credential privilege | `update/sys_scope_privilege_3acbfe7c87f4db103d6bcbf5cebb35aa.xml` |
| System User flows | `update/sys_hub_flow_0b76127487f0db103d6bcbf5cebb35b8.xml`, `update/sys_hub_flow_13d13af887b4db103d6bcbf5cebb358e.xml`, `update/sys_hub_flow_6049eef88774db103d6bcbf5cebb3569.xml` |
| Sensitive field presentation | `update/sys_ui_list_x_2311790_skills_0_student_profile_null.xml`, `update/sys_ui_section_ea956ab08774db103d6bcbf5cebb35c5.xml` |
| Role-aware dashboard widget | `update/sp_widget_e1000040000000000000000000000001.xml` |
| Role header column and instance | `update/sp_column_e1000040000000000000000000000002.xml`, `update/sp_instance_e1000040000000000000000000000003.xml` |
| Gated portal pages | `update/sp_page_3b7852f887f0db103d6bcbf5cebb35ad.xml`, `update/sp_page_5c7b56b08734db103d6bcbf5cebb3542.xml`, `update/sp_page_9fc752b887f0db103d6bcbf5cebb35ca.xml`, `update/sp_page_a0cd56b08734db103d6bcbf5cebb3553.xml` |
| Inactive orphan instances | `update/sp_instance_23cb52f08734db103d6bcbf5cebb35ca.xml`, `update/sp_instance_37e796b887f0db103d6bcbf5cebb355f.xml` |
| Gated upload instance | `update/sp_instance_8b8ed2388734db103d6bcbf5cebb35fa.xml` |
