# Step 3 — Embeddings and matching

Status: implemented in source; requires PDI testing.

## Decisions applied

- Embeddings use the Gemini `embedContent` API through `EmbeddingService`.
- Matching is hybrid: structured skills/tools, core skills, experience, and embedding similarity.
- Default shortlist is top 5, controlled by `x_2311790_skills_0.matching_top_n`.
- Matching runs before Internship Application insert/update.
- Internship requirements are stored in structured fields; `u_requirements` remains free text for the embedding input and human context.

## Runtime flow

1. Step 2 finishes and sets Student Profile `u_parse_status=parsed`.
2. `Generate Profile Embedding After Parse` calls `EmbeddingService.prepareProfile()`.
3. Internship save calls `EmbeddingService.prepareInternship()`.
4. Application save calls `MatchingEngine.scoreApplication()`.
5. `Rank Internship Applications` assigns `u_match_rank` for the top 5 applications.

## EmbeddingService

Artifact:

```text
update/sys_script_include_c100000587b0db103d6bcbf5cebb3505.xml
```

Public methods:

```javascript
prepareProfile(profile)     // -> { ok, vector, dim, model, version, attempts }
prepareInternship(internship)
embedText(text)
cosine(vectorA, vectorB)
```

- Model property: `x_2311790_skills_0.embedding_model` (default `text-embedding-004`).
- Endpoint override: `x_2311790_skills_0.embedding_endpoint`.
- Dimension property: `x_2311790_skills_0.embedding_dim` (default `768`).
- Retries: 3 total attempts for `429`, `5xx`, and status `0`.
- API key is reused from `x_2311790_skills_0.gemini_api_key`.

Vectors are stored as compact comma-separated decimal values, not JSON. A 768-dimension vector stays below the existing 8000-character field limit and can be parsed directly by `cosine()` without a binary encoding dependency.

Student Profile writes to the existing `embedding` field. Internship writes to `u_embedding`. Both store `u_embedding_model`, `u_embedding_version`, `u_embedding_dim`, `u_embedded_at`, and `u_embedding_error`.

## MatchingEngine

Artifact:

```text
update/sys_script_include_c100000687b0db103d6bcbf5cebb3506.xml
```

Public methods:

```javascript
scoreApplication(current)
rankApplicants(internshipSysId)
```

Signals and weights are normalized over whichever signals are available:

| Signal | Weight | Source |
|---|---:|---|
| Required skills | 40 | `u_required_skills` vs normalized Skill Master names |
| Preferred skills | 10 | `u_preferred_skills` vs normalized Skill Master names |
| Core skills | 10 | Step 2 `u_core=true` rows overlapping requirements |
| Tools | 10 | `u_skill_type=tool` overlap with required/preferred names; excluded when no tool overlaps |
| Experience | 10 | `u_experience_years` vs `u_min_experience_years` |
| Embedding similarity | 20 | cosine similarity when both vectors exist |

If a signal is unavailable, its weight is excluded and the remaining weights are renormalized. Scores are therefore comparable for records with the same available signals.

## Persisted application fields

| Field | Purpose |
|---|---|
| `u_match_score` | 0–100 hybrid score |
| `u_match_rank` | Top-5 rank assigned after scoring |
| `u_matched_skills` | JSON array of matched required/preferred skills |
| `u_missing_requirements` | JSON array of missing required skills |
| `u_score_breakdown` | JSON array of weight/ratio/points per signal |
| `u_match_rationale` | Human-readable explanation |
| `u_matched_at` | Last score timestamp |
| `u_matching_model_version` | `matching-hybrid-v1` |
| `u_match_error` | Last matching error, if any |

## Business rules

| Rule | Artifact | Timing |
|---|---|---|
| Generate Profile Embedding After Parse | `update/sys_script_c100000787b0db103d6bcbf5cebb3507.xml` | before update, parsed profile with new attachment |
| Generate Internship Embedding | `update/sys_script_c100000887b0db103d6bcbf5cebb3508.xml` | before insert/update |
| Match Application On Change | `update/sys_script_c100000987b0db103d6bcbf5cebb3509.xml` | before insert/update |
| Rank Internship Applications | `update/sys_script_c100002087b0db103d6bcbf5cebb3520.xml` | after insert/update |

## Properties

| Property | Default |
|---|---|
| `x_2311790_skills_0.embedding_model` | `text-embedding-004` |
| `x_2311790_skills_0.embedding_endpoint` | empty |
| `x_2311790_skills_0.embedding_dim` | `768` |
| `x_2311790_skills_0.matching_top_n` | `5` |

## Failure behavior

- Embedding failures write `u_embedding_error`; they do not change parse status.
- Matching failures write `u_match_error` and leave the previous score untouched.
- Reparsing a profile sets a new `u_parsed_attachment_sys_id`, which forces a new profile embedding.
- Changing internship requirement fields forces a new internship embedding.

## Validation

Run `docs/vertical-slices/step-3-test-plan.md` in the PDI before Step 4.
