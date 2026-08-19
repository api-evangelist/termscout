---
generated: '2026-08-14'
method: generated
name: Analyze a contract end to end
description: Submit a contract to TermScout, poll until analysis completes, then read back the scored overview, extracted fields and citations.
api: openapi/termscout-data-openapi.yml
operations: ['POST /contracts', 'GET /contracts/{contract-id}/status', 'GET /contracts/{contract-id}/overview', 'GET /contracts/{contract-id}/extracted-fields', 'GET /contracts/{contract-id}/citations', 'GET /contracts']
source: >-
  Grounded in openapi/termscout-data-openapi.yml, harvested 2026-08-14 from
  https://api.termscout.com/docs (verbatim original in openapi/_original/). Every
  path, verb and parameter named below was verified in that spec. The spec
  declares NO operationIds, so operations are referenced by method + path;
  the operationIds in overlays/termscout-data-overlay.yaml are API Evangelist's
  additions and TermScout will not recognise them. Auth per
  authentication/termscout-authentication.yml, conventions per
  conventions/termscout-conventions.yml, failure handling per
  errors/termscout-problem-types.yml.
---

# Analyze a contract end to end

This is the core TermScout flow: a contract goes in, structured contract intelligence comes out. The work is asynchronous — the upload returns immediately, analysis happens out of band, and there is no webhook, so you poll.

## Auth

Send **both** credentials on **every** request:

- `x-api-key: <your key>`
- `Authorization: <your credential>`

The spec states plainly: *"All requests require both authentication and an API key."* Base URL is `https://api.termscout.com`. There is no self-serve signup — keys come from TermScout sales. See `authentication/termscout-authentication.yml`.

## Before you start: you need ids the API cannot give you

`POST /contracts` requires `template-id`, and there is **no endpoint that lists templates**. The same is true for label, topic, question and filter ids used later in this flow. Get these from your TermScout account contact and cache them. An agent cannot discover them by calling the API. See `data-model/termscout-data-model.yml`.

## Steps

1. **Submit the contract** — `POST /contracts`. Note the unusual shape: this operation declares **no request body**. Everything is a query parameter.
   - Required: `document-name`, `template-id`.
   - Common optional: `url` (a hosted public contract to ingest), `party-a-defined-term` (the vendor/discloser/issuing entity), `party-b-defined-term` (the customer/receiver/investing entity), `contract-position-id` (`1` = vendor form), `review-level` (`verified` for human review), `is-public` (`1` public, `0` private), `defined-term`.
   - Returns `ContractUpload`: `contract_id` and `document_id`. Persist both immediately — you cannot look up a contract you did not record, and there is no search endpoint.

2. **Poll until processing completes** — `GET /contracts/{contract-id}/status`, returning `ContractStatus` with `contract_id` and `status`.
   - `status` is an **untyped string with no enum** in the spec, so the terminal values are undocumented. Do not hard-code a match you have not observed against your own account; treat any unexpected value as "still working" and rely on your timeout, not on a guessed vocabulary.
   - TermScout publishes no recommended poll interval and no rate limits. Poll conservatively with backoff — start around 10s and widen. See `rate-limits/termscout-rate-limits.yml`.

3. **Read the scored overview** — `GET /contracts/{contract-id}/overview` returns `ContractOverview`: `party_a`, `party_b`, `overall_score` against `max_score`, `rating`, `rating_display`, `rating_percentile` (position against TermScout's corpus), `overall_clarity`, `contract_position`, `category`, plus `badge_url` and `logo_url` for the TrustMark artifacts and `last_updated`.

4. **Read the extracted data points** — `GET /contracts/{contract-id}/extracted-fields` returns an array of `ExtractedField`, each carrying `extracted_field_name`, `value`, `measurement_name`, `type_name`, and — importantly — `source_language` and `citation` so every value is traceable back to the contract text.

5. **Pull the supporting language** — `GET /contracts/{contract-id}/citations` returns `Citations`. Narrow with `question-id`, `label-id` or `topic-id` to get the language behind one specific finding rather than the whole set.

6. **Re-list later if you lost track** — `GET /contracts` returns every contract on the account with `contract_id`, `status` and `template_id`. This is your only recovery path, and it takes no parameters.

## Rules an agent must follow

- **`POST /contracts` is not idempotent, and a blind retry costs you a duplicate contract.** There is no `Idempotency-Key` header anywhere in this API. The *only* convergence mechanism is the optional `contract-id` parameter, described in the spec as *"If supplied, will update existing contract with new document"*. So: if a submit times out, do **not** re-POST bare. Either re-POST with the `contract-id` you were reusing, or call `GET /contracts` first and reconcile against `document-name`. See `conventions/termscout-conventions.yml`.
- **Do not read analysis before status completes.** The prediction, citation and extracted-field endpoints will not carry meaningful data until processing finishes. An empty array here usually means "not ready", not "nothing found".
- **You have no documented error contract.** All 11 operations declare only a `200`; there are no 4xx or 5xx responses in the spec at all. Live probes return the AWS API Gateway envelope `{"message": "..."}`. Note that `403` is overloaded — it means both "missing/invalid credential" *and* "route not matched" — so a `403` may be your URL, not your key. See `errors/termscout-problem-types.yml`.
- **Nothing is paged.** `GET /contracts`, `/citations` and `/extracted-fields` all return bare unbounded arrays with no cursor, limit or total. Budget memory for a large account.
- **Watch the misspelled field.** `Citation.source_langauge` is spelled that way in the published spec, while `ExtractedField.source_language` is spelled correctly. Handle both.
- **Capture `x-amzn-requestid`** from responses. It is undocumented but it is the only request identifier available to quote to support.
- **`is-public` is a disclosure decision, not a formatting flag.** Setting `is-public=1` marks the contract public; TermScout publishes public TrustMark reports at `app.termscout.com/certify/<slug>`. Never set it without explicit human instruction.
