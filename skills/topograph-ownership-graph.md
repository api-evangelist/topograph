---
name: Retrieve ownership, UBOs and subsidiaries
description: Request the ownership graph for a company - shareholders, ultimate beneficial owners, and subsidiaries - traversing multi-level cross-border structures.
api: openapi/topograph-openapi-original.json
operations:
  - CompanyController_getCompany_v2
  - CompanyController_getCompanyRequest_v2
---

# Retrieve ownership, UBOs and subsidiaries

Use this for UBO checks, shareholder verification, and subsidiary/ownership-graph analysis.

## Auth
`x-api-key` header; base URL `https://api.topograph.co`.

## Steps

1. **Request ownership datapoints.** Call `POST /v2/company` (`CompanyController_getCompany_v2`)
   with the company `id` and `countryCode`, adding the ownership `dataPoints` you need:
   `shareholders`, `ultimateBeneficialOwners`, and/or `subsidiaries`. To walk a multi-level
   structure, use `graphContinueFromNodeIds` and cap spend with `graphMaxBudget` /
   `graphInteractive`. The response returns a `requestId`.

2. **Poll the result.** Call `GET /v2/company/{requestId}`
   (`CompanyController_getCompanyRequest_v2`). Ownership resolves progressively:
   - `shareholders[]` — each `ShareholderDTO` is an individual or a `company` (references another
     company, enabling recursion) with `sharePercentage`, `numberOfShares`, `nominalCapitalHeld`.
   - `ultimateBeneficialOwners[]` — each `UltimateBeneficialOwnerDTO` carries `name`, `birthDate`,
     `nationality`, a `control` object, and (where the register provides it) a `ubo_extract`
     certificate document.
   - `subsidiaries[]` — each `SubsidiaryDTO` references the held `company` with
     `sharePercentage` / `votingRightsPercentage`.

3. **Traverse.** Follow `company` references on shareholders/subsidiaries and re-request to build
   the full cross-border ownership graph.

## Notes
- UBO coverage follows each register (e.g. Cyprus, Slovenia, Portugal resolve with a `ubo_extract`).
- Data-model detail: `data-model/topograph-data-model.yml`. Conventions/billing:
  `conventions/topograph-conventions.yml`.
