# UVeye

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
