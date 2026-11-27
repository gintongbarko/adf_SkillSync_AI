# Step 1 — Resume intake and parsing

## Purpose

Make the PDF resume upload path deterministic and observable before adding skills, embeddings, matching, or recruiter UI.

## State machine

```text
PDF attached to Student Profile
    -> u_parse_status = pending

Gemini success
    -> u_parse_status = parsed
    -> u_headline and u_parsed_resume written
    -> u_parse_error cleared

Gemini failure or invalid response
    -> u_parse_status = failed
    -> u_parse_error = sanitized error code and message

No PDF, wrong MIME type, missing key, or bad input
    -> terminal failed result with a specific error code
```

## Sequence

1. The widget uploads a PDF to `/api/now/attachment/file`.
2. `Set Resume Pending On Attachment` fires after the attachment insert.
3. The attachment business rule sets the owning profile to `pending`, resets attempts and error, stores the attachment sys_id, and clears stale parsed output.
4. `Parse Resume When Pending` fires asynchronously.
5. `GeminiUtils` finds the newest PDF, calls Gemini with bounded retry, validates the JSON response, and returns a result object.
6. The parse business rule writes `parsed` or `failed`.

## Result contract from `GeminiUtils`

Success:

```javascript
{
  ok: true,
  data: {
    headline,
    parsed_resume,
    education,
    experience_years,
    projects,
    skills: [],
    tools: [],
    truncated
  },
  status: 200,
  attempts: 1,
  attachmentSysId: "<sys_id>"
}
```

The extra `data` fields are consumed by Step 2. Step 1 behavior only depends on `headline` and `parsed_resume`.

Failure:

```javascript
{
  ok: false,
  errorCode: "HTTP_429",
  retryable: true,
  errorMessage: "...",
  attempts: 3
}
```

Known error codes:

```text
MISSING_API_KEY
INVALID_INPUT
NO_PDF
ATTACHMENT_READ
HTTP_<status>
HTTP_EXCEPTION
RETRY_EXHAUSTED
EMPTY_CANDIDATES
EMPTY_PARTS
EMPTY_TEXT
INVALID_RESPONSE
INVALID_FIELDS
EMPTY_FIELDS
INVALID_SKILL_ARRAY
INVALID_TOOL_ARRAY
INVALID_STRUCTURED_RESPONSE
EXCEPTION
UNKNOWN_FAILURE
```

## Configuration

| Property | Purpose | Default |
|---|---|---|
| `x_2311790_skills_0.gemini_api_key` | Gemini API credential; set per environment | none |
| `x_2311790_skills_0.gemini_model` | Gemini model name | `gemini-3.5-flash` |
| `x_2311790_skills_0.gemini_endpoint` | Optional endpoint override for testing | empty (derive from model property) |

The API key ciphertext was removed from source. Rotate the previously exported key if this snapshot was shared.

## Retry policy

- Maximum attempts: 3.
- Retryable statuses: `0`, `429`, `5xx`.
- Backoff: 1 second after attempt 1, 2 seconds after attempt 2.
- Non-retryable errors fail immediately.

## Field contracts

| Field | Rule |
|---|---|
| `u_headline` | Trimmed; truncated to 100 characters with `...` |
| `u_parsed_resume` | Trimmed; truncated to 4000 characters with `...` |
| `u_parse_status` | `pending`, `parsed`, or `failed` |
| `u_parse_error` | Sanitized code and message, max 1000 characters |
| `u_parse_attempts` | Number of parse business-rule executions |
| `u_parsed_attachment_sys_id` | Attachment parsed or targeted for parsing |

## Out of scope for Step 1

- Structured skills arrays and `student_skill` rows.
- Embeddings and vector storage.
- Matching scores and rationale.
- Recruiter portal and approvals.
- Row-level ACL hardening.
