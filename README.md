# Artifact Hub (artifact-hub)
Artifact Hub is a CNCF incubating web-based application that enables finding, installing, and publishing cloud-native packages. Built primarily in TypeScript and Go, it addresses fragmentation in the cloud-native ecosystem by providing a single discovery experience for consumers. It supports 27+ artifact types including Helm charts, OPA policies, Falco rules, OLM operators, Tinkerbell actions, kubectl plugins, Tekton tasks, KEDA scalers, CoreDNS plugins, and more. Artifact Hub provides a searchable catalog with versioning, security reports via Trivy and Snyk, changelog tracking, and webhook notification support. Licensed under Apache 2.0 and governed by the CNCF.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/artifact-hub/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Cloud Native, CNCF, Helm Charts, Package Registry, Discovery, Open Source

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### Artifact Hub API
The Artifact Hub REST API provides endpoints for searching and retrieving cloud-native packages across all supported artifact types, managing repositories, handling user authentication and sessions, managing organization memberships, configuring webhooks for release notifications, and administering package security reports. The API is the backend service powering artifacthub.io and is available for self-hosted instances.

**Human URL:** [https://artifacthub.io/docs/api/](https://artifacthub.io/docs/api/)

#### Tags:

 - Package Search, Registry, REST API, Cloud Native

#### Properties

- [Documentation](https://artifacthub.io/docs/api/)
- [GitHubRepository](https://github.com/artifacthub/hub)

## Common Properties

- [Artifact Hub Documentation](https://artifacthub.io/docs/)
- [Artifact Hub GitHub Organization](https://github.com/artifacthub)
- [Artifact Hub Source Repository](https://github.com/artifacthub/hub)
- [Artifact Hub](https://artifacthub.io/)
- [CNCF Project Page](https://www.cncf.io/projects/artifact-hub/)
- [Release Notes](https://github.com/artifacthub/hub/releases)

## Features

| Name | Description |
|------|-------------|
| Package Discovery | Unified search across 27+ cloud-native artifact types including Helm charts, Kubernetes operators, OPA policies, Falco rules, and Tekton tasks from a single interface. |
| Security Reports | Automated security scanning of Helm chart images using Trivy and Snyk, with visualized vulnerability reports and severity ratings. |
| Webhook Notifications | Configurable webhooks for receiving notifications when new package versions are published or security issues are discovered. |
| Repository Management | Publishers add and manage their Helm chart repositories, OCI registries, and other sources via the Artifact Hub API. |
| Schema and Template Explorer | Interactive exploration of Helm chart values schemas and template structures directly in the browser. |
| Self-Hosting Support | Artifact Hub can be deployed on-premise using the official Helm chart, enabling organizations to run their own private artifact registry. |

## Use Cases

| Name | Description |
|------|-------------|
| Helm Chart Discovery | Platform engineers discover and evaluate Helm charts across multiple repositories from a single searchable interface with version history and security report data. |
| Package Publishing | Open source maintainers publish their Helm charts, operators, and other cloud-native packages to Artifact Hub for discoverability. |
| Security Auditing | Security teams review Artifact Hub security reports to identify vulnerable container images used in Helm charts before deployment. |
| Release Monitoring | Development teams configure webhooks to receive notifications when new versions of dependencies like Helm charts are published. |

## Integrations

| Name | Description |
|------|-------------|
| Helm | Native integration with Helm chart repositories including support for OCI-based chart distribution via container registries. |
| Trivy | Integration with Aqua Security's Trivy for container image vulnerability scanning in Helm chart security reports. |
| Snyk | Integration with Snyk for additional container security scanning capabilities in Artifact Hub security reports. |
| CNCF Landscape | Artifact Hub is an official CNCF incubating project integrated into the Cloud Native Computing Foundation's ecosystem. |

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
