# TermScout

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

TermScout is a contract intelligence and certification company for legal, procurement, and sales teams. It analyzes and benchmarks commercial agreements against market standards through Certify, and certifies them with TrustMark as a third-party signal of fairness.

TermScout publishes a machine-readable contract. An OpenAPI 3.0.1 definition for the `termscout-data` API is served anonymously at [https://api.termscout.com/docs](https://api.termscout.com/docs) — 11 operations covering contract upload, processing status, extracted fields, citations, predicted labels and red flags, playbook results, and aggregate market data across contract positions. The API itself is key-gated (`x-api-key` plus an `Authorization` credential) and access is arranged through sales.

There is no developer portal, prose API reference, SDK, CLI, Postman collection, status page, changelog, or published rate limit, and the definition declares no operationIds, no tags, and no error responses. TermScout does publish a real `llms.txt` at [https://app.termscout.com/llms.txt](https://app.termscout.com/llms.txt), indexing its public TrustMark contract reports, and a staffed trust centre at [https://security.termscout.com/](https://security.termscout.com/).

Backed by: techstars — https://termscout.com/
