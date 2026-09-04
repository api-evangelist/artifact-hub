---
name: artifact-hub-inspect-chart-config
description: Read a Helm chart's default values, its values JSON Schema, and its rendered templates from Artifact Hub before deploying it — without pulling the chart.
api: Artifact Hub Packages API
base_url: https://artifacthub.io/api/v1
auth: none
operations:
  - getHelmPackageDetails
  - getChartValues
  - getPackageValuesSchema
  - getHelmChartTemplates
  - getPackageChangelog
generated: '2026-09-04'
method: generated
source: openapi/_original/artifact-hub-openapi.yml v1.23.0 (operationIds verified against the spec)
---

# Inspect a Helm chart's configuration without installing it

This is the flow that makes Artifact Hub useful to an agent: you can read what a chart will
do before you run `helm install`, over plain anonymous HTTP.

## Prerequisites

A `package_id` (uuid) and a `version`. Get them from `artifact-hub-find-a-package`, or from
`GET /packages/helm/{repoName}/{packageName}` (**getHelmPackageDetails**), whose
`available_versions[]` lists every version with its `ts` and `contains_security_updates` flag.

Check `has_values_schema` on the detail payload first — not every chart ships one.

## 1. Default values

`GET /packages/{packageID}/{version}/values` (**getChartValues**)

The chart's `values.yaml` as published. This is what you would be overriding.

## 2. The values schema

`GET /packages/{packageID}/{version}/values-schema` (**getPackageValuesSchema**)

A JSON Schema document for those values. This is the one to prefer when you are generating
an override file programmatically: it gives you types, enums and defaults, so you can
validate your overrides before they reach a cluster instead of after.

Only call it when `has_values_schema` is `true`.

## 3. The rendered templates

`GET /packages/{packageID}/{version}/templates` (**getHelmChartTemplates**)

The chart's templates, so you can see which Kubernetes objects a release will create —
Deployments, CRDs, RBAC, hostPath mounts, privileged securityContexts — before creating any
of them.

## 4. What changed between versions

`GET /packages/{packageID}/changelog` (**getPackageChangelog**)

Gate this on `has_changelog`. Entries are typed by `ChangelogItemKind`
(added / changed / fixed / removed / security), which is the same annotation format Artifact
Hub asks publishers to use.

## Conventions that apply

- Timestamps are unix epoch **seconds** as integers.
- Search results and stats are cached at the edge (`max-age=300` and `max-age=21600`); there
  is no `ETag`, so conditional requests are not available.
- No pagination on any of these — they return whole documents.
- A `404` here usually means the version is gone or the chart never had a schema. Re-read
  `available_versions[]` before retrying.
