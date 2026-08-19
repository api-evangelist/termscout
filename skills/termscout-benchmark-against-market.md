---
generated: '2026-08-14'
method: generated
name: Benchmark a contract term against market data
description: Use TermScout's contract-positions market data to compute how common a contract term is across the corpus, and place one contract against that benchmark.
api: openapi/termscout-data-openapi.yml
operations: ['GET /contract-positions', 'GET /contracts/{contract-id}/overview', 'GET /contracts/{contract-id}/prediction-detail']
source: >-
  Grounded in openapi/termscout-data-openapi.yml, harvested 2026-08-14 from
  https://api.termscout.com/docs (verbatim original in openapi/_original/). The
  two-call numerator/denominator method below is TermScout's own, quoted from
  the description of GET /contract-positions in that spec — it is not an
  API Evangelist invention. Operations referenced by method + path because the
  spec declares no operationIds.
---

# Benchmark a contract term against market data

`GET /contract-positions` is the only endpoint on this API that returns aggregate data rather than data about one contract. It answers "how common is this term in the market?" — which is what turns a TermScout contract score into a negotiating position.

## Auth

`x-api-key` **and** `Authorization` on every request. Base URL `https://api.termscout.com`. See `authentication/termscout-authentication.yml`.

## How the endpoint behaves

`GET /contract-positions` returns `ContractPositions`, an array of `ContractPosition` (`id`, `name`, `quantity`). A contract position is the party that drafted the form — the spec states `1` = vendor form on the related `contract-position-id` parameter.

Parameters:

- `template-id` — **required**. With no other parameter, you get total contract counts per contract position. The spec notes it defaults to the IT template.
- `label-id` — optional. Narrows counts to one label.
- `selected` — optional. `1` for counts where the label is **true**, `0` for counts where the label is **not** true.

## Steps — computing a frequency percentage

TermScout documents this as an explicit two-call procedure. Quoting the spec: *"To construct frequency percentages for a given label, first call the endpoint with no `label-id` to get the denominator. Then call the endpoint with `label-id` and `selected` included to get the numerator. Then you can divide the numerator by the denominator for each contract position returned."*

1. **Denominator** — `GET /contract-positions?template-id={template}`. No `label-id`. Record `quantity` per contract position `id`.
2. **Numerator** — `GET /contract-positions?template-id={template}&label-id={label}&selected=1`. Record `quantity` per contract position `id`.
3. **Divide per position, matching on `id`.** The result is "X% of vendor-form contracts on this template contain this term". Do **not** sum across positions and divide once — vendor-form and customer-form contracts are different populations and that is the entire point of the split.
4. **Optionally get the inverse** — repeat step 2 with `selected=0` for the count where the label is explicitly *not* true. Note this is not necessarily `denominator − numerator`: a contract may be unscored for a label rather than scored false. If the two do not sum to the denominator, that gap is unscored contracts, not an error.

## Placing one contract against the benchmark

5. **Get the contract's own position** — `GET /contracts/{contract-id}/overview` returns `contract_position` and `contract_position_id`, plus `rating_percentile`, which is TermScout's own precomputed placement of that contract against the corpus. Use the same `template_id` from this response as the `template-id` in step 1 so you are comparing against the right population.
6. **Get which labels the contract actually has** — `GET /contracts/{contract-id}/prediction-detail`, then benchmark the `label_id`s it returns using steps 1–3.

## Rules an agent must follow

- **`quantity` is typed as a string in the spec**, not a number, even though it is a count. Parse it before doing arithmetic.
- **Always compare within one `template-id`.** Counts are per template; mixing templates compares unlike contract types and produces a meaningless percentage.
- **Match numerator to denominator by contract position `id`**, never by array order. Nothing in the spec guarantees stable ordering between the two calls.
- **`selected=0` is not the complement of `selected=1`.** See step 4.
- **You cannot discover `template-id` or `label-id` from this API.** Neither has a list endpoint. Take `template_id` from a contract's `overview` response and `label_id`s from its `prediction-detail` response, or get the vocabulary from your account contact. See `data-model/termscout-data-model.yml`.
- **This is aggregate market data about third-party contracts.** Report it as frequency across TermScout's corpus, with the template named. Never present a market frequency as a statement about any identifiable counterparty.
- **No paging and no published limits.** `ContractPositions` is an unbounded bare array, and a benchmark sweep across many labels is the most likely way to trip an undisclosed rate limit — sequence the calls rather than firing them in parallel. See `rate-limits/termscout-rate-limits.yml`.
