---
name: artifact-hub-check-package-security
description: Read Artifact Hub's Trivy security report for a specific package version, and use the search-level summary to screen candidates before you look at any one of them in detail.
api: Artifact Hub Packages API
base_url: https://artifacthub.io/api/v1
auth: none
operations:
  - searchPackages
  - getHelmPackageDetails
  - getPackageSecurityReport
generated: '2026-09-04'
method: generated
source: >-
  openapi/_original/artifact-hub-openapi.yml v1.23.0 and
  https://artifacthub.io/docs/topics/security_report/ (operationIds verified against the spec)
---

# Check what is wrong with a package before you deploy it

Artifact Hub scans the container images referenced by the packages it lists using
[Trivy](https://trivy.dev/) and stores the **full Trivy JSON output**. That means you get the
real scanner report, not a badge.

## 1. Screen at search time — do not fetch reports one by one

Every `PackageSummary` returned by `GET /packages/search` (**searchPackages**) already
carries `security_report_summary` with counts by severity:

```
security_report_summary: { critical, high, medium, low, unknown }
```

Filter on this first. It costs you nothing extra and it keeps you off the rate limiter.

Also read `all_containers_images_whitelisted` — when it is `true` the publisher has excluded
images from scanning, so a clean summary means less than it looks like.

## 2. Fetch the full report for the finalists

`GET /packages/{packageID}/{version}/security-report` (**getPackageSecurityReport**)

Needs `package_id` and an exact `version`. The response is Trivy's own JSON.

## 3. Read the flags that sit alongside it

From the package detail payload (**getHelmPackageDetails**):

- `signed` and `signatures[]` (`prov`, `cosign`) — is the artifact signed, and how.
- `contains_security_updates` on each entry of `available_versions[]` — which upgrade is the
  security upgrade.
- `deprecated` — do not deploy it.
- `official`, `verified_publisher`, `cncf` — who is actually behind the listing.

## Honest limits

- **No report is not the same as no vulnerabilities.** Charts with no scannable container
  images, with scanning disabled on the repository (`scanner_disabled`), or with more than 15
  images in a version (the scanner skips those) simply have no report.
- Reports go stale. The repository payload carries `last_scanning_ts` and
  `last_scanning_errors`; repositories are processed every 30 minutes, but a scan is not
  guaranteed on every pass.
- To be told about new findings instead of polling, subscribe to event kind `1`
  (Security alerts) — see `artifact-hub-watch-for-releases`.
