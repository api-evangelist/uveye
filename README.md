# UVeye

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

UVeye builds AI-powered automated vehicle inspection systems — "the MRI for cars." A vehicle drives
through a scanning lane and UVeye returns a full condition report in seconds across four modules:
Artemis (tires and tread), Helios (undercarriage), Atlas (exterior body damage) and Apollo
(interior). The systems run in dealership service lanes, fleet and leasing depots, auctions and
remarketing lots, rental returns, OEM plants and logistics/PDI ports.

## API

**UVeye Public API v1** — third-party access to inspection data.

- Documentation: https://api.v1.uveye.dev/ (published as a Postman collection on a UVeye custom domain)
- Development base URL: `https://api.uveye.dev/v1/`
- Auth: `uveye-api-key` header (issued in the UVeye Back Office); Merchandise endpoints use a
  separate `Authorization: Bearer` credential issued at onboarding.
- 8 operations plus one outbound webhook: inspection details, latest inspections, public link,
  inspection shared, quote, image, merchandise inventory submit, merchandise mark-sold.

UVeye publishes no OpenAPI of its own. The OpenAPI in `openapi/` is **derived** from UVeye's own
published Postman collection, which is saved verbatim in `postman/`.

## Links

- https://uveye.com/
- https://api.v1.uveye.dev/ — API documentation
- https://us.backoffice.uveye.app/ — UVeye Back Office (API keys)
- https://uveye.com/partners/integrations/ — integrations
- https://trust.uveye.com/ — trust center (ISO/IEC 27001:2022, SOC 2 Type 2)
- https://uveye.com/customer-support/
- https://github.com/UVeye
