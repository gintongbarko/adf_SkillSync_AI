# Step 2 — Structured skills extraction

Status: implemented in source; requires PDI testing.

## Purpose

Turn a successful PDF parse into structured, reusable data without adding embeddings or matching.

Step 1 output is preserved:

- `u_headline`
- `u_parsed_resume`
- `u_parse_status`

Step 2 output is added:

- Student Profile: `u_education`, `u_experience_years`, `u_projects`
- Skill Master rows with normalized names
- Student Skill rows for both skills and tools

## Data model decisions

- Skills and tools share `x_2311790_skills_0_student_skill`.
- `u_skill_type` distinguishes `skill` from `tool`.
- A maximum of **10 combined** resume-derived rows are written per profile.
- The top **3** rows are marked `u_core = true`.
- Resume rows use `u_source = resume`; manually entered rows use `u_source = manual`.
- Reparsing deletes only `u_source = resume` rows and preserves manual rows.
- Structured writer failure marks the parse as `failed`.

## Sequence

1. `GeminiUtils` asks Gemini for Step 1 fields plus `skills`, `tools`, `education`, `experience_years`, and `projects`.
2. `GeminiUtils` validates and normalizes the response.
3. `Parse Resume When Pending` calls `StructuredSkillsWriter`.
4. `StructuredSkillsWriter` deletes old resume-derived Student Skill rows.
5. Skill Master rows are upserted by `u_normalized_name`.
6. Up to 10 Student Skill rows are inserted with rank, confidence, evidence, proficiency, type, source, and extraction time.
7. The profile receives headline, parsed resume, education, years, projects, and final parse status.

## Gemini response fields

```json
{
  "headline": "3rd Year CS Student | Backend Developer",
  "parsed_resume": "...",
  "skills": [
    {
      "name": "Python",
      "proficiency": "intermediate",
      "confidence": 0.91,
      "evidence": "Built Flask APIs in coursework",
      "rank": 1
    }
  ],
  "tools": [
    {
      "name": "Git",
      "confidence": 0.88,
      "evidence": "Used in group projects",
      "rank": 2
    }
  ],
  "education": "BS Computer Science, Expected 2027",
  "experience_years": 2.5,
  "projects": "Coursework API and analytics dashboard"
}
```

Validation rules:

- `skills` and `tools` must be arrays.
- Each item must have a non-empty `name`.
- Names are trimmed and case-normalized for lookup.
- Confidence is clamped to `0..1`.
- Combined items are capped at 10.
- Education is truncated to 2000 characters.
- Projects are truncated to 4000 characters.
- Evidence is truncated to 500 characters.

New parser error codes:

```text
INVALID_SKILL_ARRAY
INVALID_TOOL_ARRAY
INVALID_STRUCTURED_RESPONSE
```

Writer failures are stored as:

```text
SKILL_WRITE_FAILED:<writerErrorCode>
```

## Artifacts

| Concern | Artifact |
|---|---|
| Parser | `update/sys_script_include_dc8e36f08738db103d6bcbf5cebb35a8.xml` |
| Structured writer | `update/sys_script_include_f100000e87b0db103d6bcbf5cebb350e.xml` |
| Parse orchestration | `update/sys_script_f3de32348738db103d6bcbf5cebb35e8.xml` |
| Cap rule | `update/sys_script_2a355e3487f0db103d6bcbf5cebb3536.xml` |
| Widget | `update/sp_widget_278a9a308734db103d6bcbf5cebb35ea.xml` |
| Table schema | `dictionary/*.xml` |
| Field definitions | `update/sys_dictionary_*.xml` |
| Choice definitions | `update/sys_choice_*.xml` |

## Out of scope

- Embeddings and vector storage.
- Matching score, rank, or rationale.
- Recruiter portal.
- Approval workflow.
- Full row-level ACL hardening.

Run `docs/vertical-slices/step-2-test-plan.md` in the PDI before proceeding to Step 3.
