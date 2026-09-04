---
name: artifact-hub-mirror-the-catalog
description: Pull the whole Artifact Hub catalogue in bulk using the integration dump endpoints, which is what the provider tells you to do instead of paging search into a rate limit.
api: Artifact Hub Integrations API
base_url: https://artifacthub.io/api/v1
auth: none
operations:
  - getHarborReplicationDump
  - getHelmExporterDump
  - getNovaDump
  - getArtifactHubStats
generated: '2026-09-04'
method: generated
source: >-
  openapi/_original/artifact-hub-openapi.yml v1.23.0 and
  https://artifacthub.io/docs/topics/faq/ (operationIds verified against the spec)
---

# Read the whole catalogue without hitting the rate limit

Artifact Hub's own FAQ answers "How can I get all charts listed on artifacthub.io without
hitting rate limits?" with: use the integration dump endpoints. Paging `searchPackages` for
the whole catalogue is the wrong tool and will get you throttled.

Anonymous. No credentials.

## The three dumps

| Operation | Path | Shape |
|---|---|---|
| **getHarborReplicationDump** | `GET /harbor-replication` | Every listed Helm chart, in the format Harbor's replication adapter consumes. The general-purpose one. |
| **getHelmExporterDump** | `GET /helm-exporter` | The Helm exporter format. |
| **getNovaDump** | `GET /nova` | The Fairwinds Nova format. Includes package stars since 1.21.0. |

Each returns the whole set in a single response.

## Use it well

1. Call `GET /stats` (**getArtifactHubStats**) first for the current package and release
   counts, so you know roughly what you are about to download and can sanity-check what you
   got.
2. Honour the edge cache. `/stats` is served `max-age=21600` (6 hours) and search
   `max-age=300`, behind CloudFront. There is **no `ETag` and no `Last-Modified`**, so you
   cannot do a conditional request — which is another reason to pull rarely and in bulk.
3. Repositories are re-processed every 30 minutes and changes appear within about 5 minutes of
   processing, so nothing is gained by pulling more often than hourly.
4. Even here, `429` is documented. There is no `Retry-After` and no rate-limit header on any
   response, so back off exponentially with jitter.

## What the dump does not give you

Per-version detail. Values, values schemas, rendered templates, security reports and
changelogs are per-`package_id`+`version` reads — see `artifact-hub-inspect-chart-config`
and `artifact-hub-check-package-security`. Use the dump to build the index, then fetch detail
only for what you actually care about.
