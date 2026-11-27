# Step 2 PDI test plan

Run these tests after importing Step 2 and confirming Step 1 still works.

## Setup

1. Confirm the new fields exist on Student Profile, Skill Master, and Student Skill.
2. Confirm choices exist:
   - Skill Master `u_category`: `skill`, `tool`
   - Student Skill `u_skill_type`: `skill`, `tool`
   - Student Skill `u_source`: `resume`, `manual`
3. Confirm unique indexes are present:
   - Skill Master `u_normalized_name`
   - Student Skill `u_profile + u_skill`
4. Confirm `StructuredSkillsWriter` is active and accessible.
5. Confirm `Cap Student Skills` targets `x_2311790_skills_0_student_skill`.
6. Confirm `gemini_api_key` is set and the model supports structured JSON output.

## Test matrix

| ID | Scenario | Expected result |
|---|---|---|
| T1 | Upload a PDF with clear skills | Skill Master and Student Skill rows are created |
| T2 | Re-upload a newer PDF | Old `u_source=resume` rows are removed and new rows are created |
| T3 | Add a manual Student Skill row, then reparse | Manual row survives and is untouched |
| T4 | Extract `Python`, `python`, and `PYTHON` | One Skill Master row with normalized name `python` |
| T5 | Gemini returns more than 10 combined skills/tools | Exactly 10 Student Skill rows are written |
| T6 | More than 3 rows parsed | First 3 have `u_core=true`; the rest have `u_core=false` |
| T7 | Tools are extracted | Rows use `u_skill_type=tool` |
| T8 | Education, years, and projects are present | Values persist on Student Profile |
| T9 | Malformed structured Gemini response | Parse status becomes `failed`; error starts with `INVALID_SKILL_ARRAY`, `INVALID_TOOL_ARRAY`, or `INVALID_STRUCTURED_RESPONSE` |
| T10 | Writer exception | Parse status becomes `failed`; error starts with `SKILL_WRITE_FAILED` |
| T11 | Step 1 behavior | Headline, parsed resume, retries, and status transitions still pass |
| T12 | Existing manual rows plus 10 parsed rows | Cap rule allows the 10 parsed rows and does not delete manual rows |
| T13 | Widget after successful parse | Read-only skills and tools chips are visible; top 3 are highlighted |
| T14 | Duplicate skill/tool name in one parse | Only one Student Skill row is created for that profile |
| T15 | Empty skills and tools arrays | Parse succeeds with no Student Skill rows |

## Manual checks

- Student Skill list: `u_source`, `u_skill_type`, `u_rank`, `u_core`, and `u_extracted_at` are populated.
- Skill Master list: `u_normalized_name` is lowercase and unique.
- No `u_student_skill` table references remain in active scripts.
- System Logs: no unhandled exceptions from `StructuredSkillsWriter`.
- Upload widget: no edit controls appear for parsed skills in Step 2.

## Rollback note

Reparsing deletes only resume-derived rows. To restore manual rows, do nothing; they are never deleted by the writer. To clear parsed rows, delete records where `u_source=resume`.
