# Step 3 PDI test plan

Run this after Step 1 and Step 2 pass.

## Setup

1. Confirm `x_2311790_skills_0.gemini_api_key` is set and the key permits `embedContent`.
2. Confirm properties:
   - `x_2311790_skills_0.embedding_model = text-embedding-004`
   - `x_2311790_skills_0.embedding_dim = 768`
   - `x_2311790_skills_0.matching_top_n = 5`
3. Confirm these script includes are active:
   - `EmbeddingService`
   - `MatchingEngine`
   - `StructuredSkillsWriter`
4. Confirm these business rules are active:
   - `Generate Profile Embedding After Parse`
   - `Generate Internship Embedding`
   - `Match Application On Change`
   - `Rank Internship Applications`
5. Confirm the new fields exist on Student Profile, Internship, and Internship Application.

## Embedding generation

| # | Action | Expected result |
|---:|---|---|
| 1 | Create or reparse a profile through the upload widget. | `embedding` is populated; `u_embedding_dim=768`; `u_embedded_at` is set; `u_embedding_error` is empty. |
| 2 | Copy the vector and count comma-separated values. | Exactly 768 values. |
| 3 | Check the vector length. | Under 8000 characters. |
| 4 | Change `u_required_skills` on an internship and save. | `u_embedding`, `u_embedding_dim`, `u_embedded_at` update; `u_embedding_error` clears. |
| 5 | Save an internship with no requirement text. | `u_embedding_error` contains `EMPTY_INTERNSHIP_TEXT`; no unhandled exception in System Logs. |
| 6 | Temporarily clear the API key and reparse a profile. | `u_embedding_error` contains `MISSING_API_KEY`; profile parse status remains `parsed`. Restore the key afterward. |

## Structured internship requirements

| # | Action | Expected result |
|---:|---|---|
| 1 | Set `u_required_skills` to `["Python","SQL"]`. | Matching uses both names without parsing `u_requirements`. |
| 2 | Set `u_preferred_skills` to `["Flask"]`. | Preferred signal contributes when the candidate has Flask. |
| 3 | Set `u_min_experience_years` to `2`. | Experience signal contributes 10 weight. |
| 4 | Leave `u_required_skills` empty. | Required-skill weight is excluded and the other weights are renormalized. |

## Matching

Use a candidate profile with normalized skills `Python`, `SQL`, `Flask`, core `Python`, `SQL`, tool `Git`, and `u_experience_years=2.5`.

| # | Internship requirement | Expected result |
|---:|---|---|
| 1 | Required `Python, SQL`; preferred `Flask`; min years `2`. | High score; `u_matched_skills` includes Python/SQL/Flask; `u_missing_requirements` is `[]`; rationale mentions 2 of 2 required and 1 of 1 preferred. |
| 2 | Required `Python, Docker`; preferred `Flask`; min years `2`. | Docker appears in `u_missing_requirements`; score is lower than case 1. |
| 3 | No vectors on either record. | Score uses structured signals only; no embedding component appears in `u_score_breakdown`. |
| 4 | Both vectors present. | `u_score_breakdown` includes `embedding` with weight 20. |
| 5 | Change only the applicant's `u_student`. | New score and `u_matched_at` are written. |
| 6 | Change only `u_match_rank`. | Score and `u_matched_at` do not change. |

## Ranking

| # | Action | Expected result |
|---:|---|---|
| 1 | Create 7 applications for one internship. | `u_match_rank` is 1 through 5 for the top five; remaining rows are blank. |
| 2 | Change a lower-ranked application so it becomes the strongest match. | Ranks update so it becomes 1 and the prior leader moves down. |
| 3 | Set `x_2311790_skills_0.matching_top_n=3` and re-save an application. | Only ranks 1 through 3 are assigned. Restore to 5 afterward. |
| 4 | Run `MatchingEngine().rankApplicants('<internship sys_id>')` in Script Background. | Returned results are sorted descending by score and have sequential ranks. |

## Failure behavior

| # | Action | Expected result |
|---:|---|---|
| 1 | Force an embedding error with an invalid endpoint. | `u_embedding_error` contains an error code; application matching still runs and omits the embedding component. Restore the endpoint. |
| 2 | Delete the applicant's Student Profile. | Application save writes `u_match_error` containing `PROFILE_NOT_FOUND`; no unhandled exception. |
| 3 | Set an invalid dimension property. | `EmbeddingService` falls back to 768 or reports `DIMENSION_MISMATCH`; no partial vector is written. |

## Regression checks

1. Step 1 upload still accepts PDF only and writes `Parsed`, `Failed`, or `Pending`.
2. Step 2 still writes max 10 resume-derived skill rows with top 3 marked core.
3. Manual skills survive a reparse.
4. All XML imports cleanly and no `u_student_skill` stale references remain.
5. `checksum.txt` is expected to be stale until the snapshot is regenerated from ServiceNow.
