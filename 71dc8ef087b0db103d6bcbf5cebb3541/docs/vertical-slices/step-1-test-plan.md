# Step 1 PDI test plan

Run these tests after importing the source changes into a personal developer instance.

## Setup

1. Create at least two student users with a Student Profile each.
2. Give each profile a valid `u_user` reference.
3. Set `x_2311790_skills_0.gemini_api_key` in the target instance.
4. Confirm `x_2311790_skills_0.gemini_model` is set to a model available to your API key.
5. Confirm the three choices exist: `pending`, `parsed`, `failed`.
6. Confirm the old `Set Resume as Pending` flow is inactive and the old parser/REST message records are gone.
7. Refresh `checksum.txt` from ServiceNow after making the equivalent changes in the instance.

## Test matrix

| ID | Scenario | Expected result |
|---|---|---|
| T1 | Student A uploads a valid PDF | Profile A becomes `pending`, then `parsed`; headline <= 100; parsed resume <= 4000 |
| T2 | Student B uploads a valid PDF | Profile B parses; Profile A is unchanged |
| T3 | Edit an unrelated profile field | Status is not reset to `pending` by a profile edit alone |
| T4 | Try to upload DOCX | Browser rejects it; no attachment and no status change |
| T5 | Force `pending` with no PDF | Status becomes `failed`; error starts with `NO_PDF` |
| T6 | Blank API key | Status becomes `failed`; error starts with `MISSING_API_KEY`; no plaintext key appears in logs |
| T7 | Use `x_2311790_skills_0.gemini_endpoint` to point at a mock that returns 429 or 5xx | Three total attempts occur; status becomes `failed`; attempts = 1 |
| T8 | Mock malformed Gemini JSON | Status becomes `failed`; error starts with `INVALID_RESPONSE` or `INVALID_FIELDS` |
| T9 | Oversized model output | Values are truncated safely; `parsed` is still written |
| T10 | Upload a newer PDF after a successful parse | Newest attachment wins; stale parse results are skipped; profile returns to `parsed` |
| T11 | Student tries to access another profile | ACL blocks read/write/attachment access |
| T12 | Fresh import into clean instance | One active parser, one active attachment BR, choices present, widget is PDF-only |
| T13 | Two uploads close together | Last uploaded PDF is the one reflected in `u_parsed_attachment_sys_id`; no older result overwrites it |

## Manual checks

- System Logs > All: no unhandled exceptions from GeminiUtils.
- Student Profile form: `embedding` is not displayed.
- Widget: status, attempt count, and sanitized error are visible when present.
- Flow Designer: `Set Resume as Pending` is inactive; no published flow updates a hardcoded sys_id.
