---
name: Resolve a company and retrieve KYB data
description: Search-first resolution of a company, then request its data and documents from official registers and poll for the result.
api: openapi/topograph-openapi-original.json
operations:
  - SearchController_searchV2_v2
  - CompanyController_getCompany_v2
  - CompanyController_getCompanyRequest_v2
---

# Resolve a company and retrieve KYB data

Use this for company onboarding and KYB verification. Topograph pulls data in real time from
official business registers across 35+ countries.

## Auth
Send your workspace key in the `x-api-key` header on every REST call. Base URL `https://api.topograph.co`.

## Steps

1. **Resolve the company (search-first).** Call `GET /v2/search` (`SearchController_searchV2_v2`)
   with a `countryCode` and a query string. Pick the `SearchResult` whose `matchReason` best fits;
   keep its `id`. If you already hold an official identifier (e.g. FR SIREN), you can skip search.

2. **Request the data.** Call `POST /v2/company` (`CompanyController_getCompany_v2`) with the
   company `id`, `countryCode`, the `dataPoints` you need, and any `documents`. Set `mode` to
   `onboarding` for the faster path or leave default for full verification. Attach a `metadata`
   object (e.g. `{ "caseId": "..." }`) to correlate later. The response returns a `requestId`.

3. **Poll for the result.** Call `GET /v2/company/{requestId}`
   (`CompanyController_getCompanyRequest_v2`) to fetch the latest available data with progressive
   delivery of documents and additional datapoints. **This lookup is always free and never
   re-billed.**

## Conventions & billing
- Idempotency is by natural key: a 24-hour deduplication window prevents duplicate billing for the
  same data block, company, and account — repeating the same `POST /v2/company` in that window
  returns the cached result. `requestId` lookups are always free.
- Every returned document carries a `source` object naming the issuing register.

## Errors
`401` invalid/missing API key; `404` requestId not found or not owned by this account. See
`errors/topograph-problem-types.yml` (custom `{statusCode,error{code,message}}` envelope).
