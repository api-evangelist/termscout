---
generated: '2026-08-14'
method: generated
name: Review red flags and playbook results
description: Pull TermScout's predicted red flags, per-topic predicted labels and playbook pass/fail detail for a processed contract, with the source language behind each finding.
api: openapi/termscout-data-openapi.yml
operations: ['GET /contracts/{contract-id}/prediction-flags', 'GET /contracts/{contract-id}/prediction-detail', 'GET /contracts/{contract-id}/prediction-summary', 'GET /contracts/{contract-id}/playbook-detail', 'GET /contracts/{contract-id}/citations', 'GET /contracts/{contract-id}/status']
source: >-
  Grounded in openapi/termscout-data-openapi.yml, harvested 2026-08-14 from
  https://api.termscout.com/docs (verbatim original in openapi/_original/). Every
  path, verb, parameter and response schema field named below was verified in
  that spec. The spec declares no operationIds, so operations are referenced by
  method + path. Cross-cutting rules per conventions/termscout-conventions.yml,
  errors/termscout-problem-types.yml and data-model/termscout-data-model.yml.
---

# Review red flags and playbook results

This is the procurement/legal review flow. The contract has already been submitted and processed (see `termscout-analyze-a-contract.md`); now you extract the risk findings and check them against a playbook.

## Auth

`x-api-key` **and** `Authorization` on every request. Base URL `https://api.termscout.com`. See `authentication/termscout-authentication.yml`.

## Preconditions

Confirm `GET /contracts/{contract-id}/status` reports the contract as processed before reading anything below. Prediction endpoints on an unprocessed contract return empty structures, which look identical to "clean contract" — a dangerous ambiguity in a risk review.

## Steps

1. **Get the red flags** — `GET /contracts/{contract-id}/prediction-flags` returns `PredictionFlags`: `contract_id` plus a `details` array of `PredictionFlagsDetail`, each a named bucket with `name`, `flag_count`, and a `flags` array of `PredictionFlag`.

   Each flag carries `label_id`, `label_display`, `confidence`, `defined_term`, `selected`, and both `citation` and `source_language`. Because the citation travels with the flag, you can present a finding *and* its evidence without a second call.

2. **Get the full predicted-label picture** — `GET /contracts/{contract-id}/prediction-detail` returns `PredictionDetail`, whose `details` array groups `PredictionDetailTopic` by `topic_id` / `topic_name`, each holding a `predictions` array of `PredictionDetailLabel` with the same field shape as a flag.

   Red flags are the subset worth alarming on; this is everything the model predicted, flagged or not.

3. **Summarise a topic in prose** — `GET /contracts/{contract-id}/prediction-summary` requires `topic-id` (**required**, unlike most filters on this API) and returns `PredictionSummary` with a `summary` string for that one topic. Call it per topic you care about; there is no all-topics variant.

4. **Run the playbook** — `GET /contracts/{contract-id}/playbook-detail`, optionally narrowed by `filter-id`, returns `PlaybookDetail` per question: `display_name`, `answer_display`, `filter_passed` (boolean), `tier`, `rank`, `issue_number`, `footnote`, `citation`, and the label-set arrays `required_label_ids`, `acceptable_label_ids`, `unacceptable_label_ids`, `selected_label_ids`, `unselected_label_ids`, plus a `details` array of `PlaybookPassFailDetail` (`selection`, `reason`, `is_fail`, `label_id`).

   `filter_passed` is the pass/fail verdict; `details[].reason` is the human-readable justification and `details[].is_fail` marks which specific selections drove a failure.

5. **Chase any finding back to the contract text** — `GET /contracts/{contract-id}/citations` with the `label-id`, `topic-id` or `question-id` from any of the above returns the `Citations` for exactly that finding.

## Rules an agent must follow

- **`confidence` is on every prediction — surface it, never drop it.** TermScout's public material describes prediction accuracy as a range, not a certainty. A flag with low confidence presented as fact is a misrepresentation of the vendor's own output. Always carry `confidence` through to whatever you show a human.
- **Never present a finding without its `citation` / `source_language`.** Every prediction and flag ships with the contract language behind it. Dropping it turns a traceable, auditable finding into an unsourced assertion about someone's legal agreement.
- **These are predictions, not legal advice.** `review-level` on submission distinguishes model output from human-verified review. If the contract was not submitted with `review-level=verified`, say so when reporting.
- **`selected` is a number, not a boolean.** It appears as `0`/`1` on `PredictionFlag` and `PredictionDetailLabel`. The `selected` query parameter on `GET /contract-positions` uses the same convention (`1` = label true, `0` = label not true).
- **Label, topic and question ids have no list endpoint.** You cannot enumerate them from the API. Harvest them from a processed contract's `prediction-detail` response, or get the vocabulary from your account contact. See `data-model/termscout-data-model.yml`.
- **`filter_passed` is a boolean but `is_fail` is a number.** Type handling differs between the two levels of the playbook response. Do not assume one shape.
- **No error contract, no paging, no rate limits published.** Same caveats as every other operation on this API — see `errors/termscout-problem-types.yml`, `conventions/termscout-conventions.yml` and `rate-limits/termscout-rate-limits.yml`.
