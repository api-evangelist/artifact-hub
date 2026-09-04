---
name: artifact-hub-find-a-package
description: Search Artifact Hub for a cloud-native package (Helm chart, operator, policy, plugin, Tekton task…) and resolve it to a package_id you can use for every follow-up call.
api: Artifact Hub Packages API
base_url: https://artifacthub.io/api/v1
auth: none
operations:
  - searchPackages
  - getPackageSummary
  - getHelmPackageDetails
  - getArtifactHubStats
generated: '2026-09-04'
method: generated
source: openapi/_original/artifact-hub-openapi.yml v1.23.0 (operationIds verified against the spec)
---

# Find a package on Artifact Hub

No credentials are needed. Every operation in this skill is anonymous.

## 1. Search

`GET /packages/search` (**searchPackages**)

Required: `facets` (boolean — set `false` unless you want aggregation counts).

Useful parameters, all from the contract:

| Parameter | Notes |
|---|---|
| `ts_query_web` | Plain websearch-style text, e.g. `postgres operator` |
| `ts_query` | Boolean expression, e.g. `(automation \| configuration)` |
| `kind` | Repeatable. Integer repository kind — `0` Helm charts, `3` OLM operators, `2` OPA policies, `15` Kyverno policies, `7` Tekton tasks. See `RepositoryKind`. |
| `org`, `repo`, `user` | Repeatable, narrow to a publisher |
| `official`, `verified_publisher`, `cncf` | Booleans — the trust filters |
| `deprecated` | Defaults to `false`; leave it alone unless you want dead packages |
| `license` | Repeatable SPDX identifier |
| `sort` | `relevance` (default) \| `stars` \| `last_updated` |
| `limit` | Default 20, **maximum 60** — asking for more is a 400 |
| `offset` | Page with this |

Read the total from the **`Pagination-Total-Count`** response header, not from the body.

```
GET /api/v1/packages/search?facets=false&ts_query_web=nginx%20ingress&kind=0&official=true&limit=20
```

## 2. Pick the right result

Each hit is a `PackageSummary`. Prefer, in this order:

1. `official: true` — the publisher is the project that makes the software.
2. `verified_publisher: true` — ownership of the repository is proven.
3. `cncf: true` — published by a CNCF project.
4. Higher `stars`, and a recent `ts` (unix epoch **seconds**, not milliseconds).

Reject anything with `deprecated: true`. Note `security_report_summary` — the
critical/high/medium/low/unknown counts are right there in the search result.

## 3. Keep both identifiers

You need two things from the hit, because the API is addressed two different ways:

- the triple `repository.kind` (as its path segment, e.g. `helm`), `repository.name`, `name` —
  used by the detail operations;
- `package_id` (uuid) — used by values, values-schema, templates, security-report,
  changelog, stars and views.

## 4. Get the details

`GET /packages/{repoKindParam}/{repoName}/{packageName}/summary` (**getPackageSummary**) for a
light read, or the per-kind detail operation for everything —
`GET /packages/helm/{repoName}/{packageName}` (**getHelmPackageDetails**) for a Helm chart.
Each of the 27 supported kinds has its own path segment and its own operation; there is no
generic detail endpoint.

The detail payload adds `available_versions[]`, `readme`, `links[]`, `maintainers[]`,
`has_values_schema` and `has_changelog` — check those two booleans before calling the
skills that read them.

## Failure modes

- **404** — the version disappeared upstream. Artifact Hub drops versions that vanish from the
  source repository, so an id that resolved yesterday can 404 today. Re-search.
- **400** — almost always `limit` over 60, a malformed uuid, or a bad `sort` value.
- **429** — undocumented and header-free. Back off exponentially with jitter; there is no
  `Retry-After`. If you are reading the whole catalogue, do not page search at all — see
  `artifact-hub-mirror-the-catalog`.
- Errors are `{"message": "..."}`. There is no error code to branch on.
