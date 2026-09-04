---
name: artifact-hub-watch-for-releases
description: Register an Artifact Hub webhook so you are notified when a package you depend on ships a new version or gets a security alert — and test the delivery before you rely on it.
api: Artifact Hub Webhooks API
base_url: https://artifacthub.io/api/v1
auth: api-key (X-API-KEY-ID + X-API-KEY-SECRET)
operations:
  - triggerWebhookTest
  - addUserWebhook
  - getUserWebhooks
  - getUserWebhookDetail
  - updateUserWebhook
  - deleteUserWebhook
  - addOrganizationWebhook
  - addPackageSubscription
generated: '2026-09-04'
method: generated
source: openapi/_original/artifact-hub-openapi.yml v1.23.0 (operationIds verified against the spec)
---

# Get told when a dependency moves

Unlike the read surface, this one needs credentials: send **both** `X-API-KEY-ID` and
`X-API-KEY-SECRET` on every call. Sending one is the same as sending none (401). Keys are
issued from the Artifact Hub web control panel; there is no API to mint them.

## 1. Test the delivery FIRST

`POST /webhooks/test` (**triggerWebhookTest**)

```json
{
  "url": "https://your-endpoint.example/hooks/artifact-hub",
  "content_type": "application/json",
  "template": "{\"text\": \"Package {{ .Package.Name }} version {{ .Package.Version }} released! {{ .Package.URL }}\"}",
  "event_kinds": [0]
}
```

This delivers a sample notification without creating anything. Do this before step 2 — it is
the only rehearsal facility the API has, and it costs nothing to get wrong.

## 2. Create the webhook

`POST /webhooks/user` (**addUserWebhook**), or `POST /webhooks/org/{orgName}`
(**addOrganizationWebhook**) for a team.

Body fields, from `WebhookSummary`: `name`, `description`, `url`, `secret`, `content_type`,
`template`, `active`, `event_kinds[]`, and the packages to watch.

Event kinds — note the enum is **sparse**, there is no `3`:

| Kind | Meaning |
|---|---|
| `0` | New package release |
| `1` | Security alerts |
| `2` | Repository tracking errors (your repo stopped updating) |
| `4` | Repository scanning errors |

**The payload is yours.** Artifact Hub renders your Go `template` string, so there is no
canonical event schema — your receiver parses exactly what you wrote here and nothing else.

**Do not assume signature verification.** The `secret` field exists, but the contract and the
docs specify no signature header, digest algorithm or verification procedure. Treat the
endpoint as unauthenticated and validate the content, or keep the URL unguessable.

## 3. Confirm and monitor

`GET /webhooks/user/{webhookID}` (**getUserWebhookDetail**) returns `last_notifications[]`,
each with `created_at`, `processed`, `processed_at` and, on failure, `error`. Poll this after
a release you expected. Retry policy and retention are not documented.

## 4. Email instead of HTTP

If you want notifications rather than an integration, `POST /subscriptions`
(**addPackageSubscription**) takes a `packageID` and an `event_kind` and needs no receiver.

## Idempotency warning

There is none. No `Idempotency-Key`, no client request id. A retried `POST /webhooks/user`
after a timeout can leave you with two webhooks firing on every release — list with
**getUserWebhooks** and reconcile before retrying, and clean up with **deleteUserWebhook**.
Deleting a webhook does not recall notifications already delivered.
