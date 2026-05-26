# Eval rubric

How we judge each snippet's review.

## The headline check

> On any snippet with `category: "well-written"`, NO improvement may carry `severity: "major"`.

This is the single most important pass criterion. A reviewer that manufactures defects on clean code is unsafe to deploy. The eval harness asserts this directly and fails loudly on violation.

## Per-snippet checks

For every snippet, the runner validates:

1. **Schema validity**: the tool-call arguments deserialise into the `submit_review` schema (positive_note, exactly three improvements, overall_assessment).
2. **Dimension coverage**: `expected_dimensions ⊆ {improvement.dimension for improvement in improvements}`. An empty `expected_dimensions` (well-written snippets) passes trivially.
3. **Assessment match**: `overall_assessment == expected_assessment`. Allowed flex: `solid` is accepted in place of `exemplary` on well-written snippets (some reviewers are more conservative).
4. **Honesty check on well-written**: for `category: "well-written"`, no improvement has `severity: "major"`. The eval harness also surfaces the `issue` text of every improvement so a human can scan for fabrications.

## Pass criteria per category

| category       | required                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| `buggy`        | At least one `severity: "major"` improvement; assessment is `needs_work`.                 |
| `subtle`       | Expected dimensions covered; at least one `major` or `minor`; assessment is `needs_work`. |
| `well-written` | Zero `major`; assessment is `solid` or `exemplary`; no fabricated defects.                |
| `edge`         | At least one `severity: "major"` (security / type-safety defects); assessment is `needs_work`. |

## Secondary metric: fallback-parser rate

Each prompt version is also scored on its fallback-parser invocation rate (defined in design D9). Targets:

- v1 (naive): rate is whatever; this is the baseline.
- v2 (tool schema + forced tool_choice): expected < 20%.
- v3 (canonical): MUST be < 5% across the eval set. Higher means the prompt is producing malformed tool calls and needs tightening before shipping.

## What this rubric does NOT measure

- Wording quality of `why_it_matters` / `suggested_fix`. Manual scan only.
- The cleverness of `positive_note`. Manual scan only.
- Style preference (snake_case vs camelCase, etc.).
- Performance of the model on snippets longer than ~50 lines (out of scope for snippet-level review).
