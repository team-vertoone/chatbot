MODULE SPEC: M12 — Integrations & API
Phase: Phase 2
Purpose: Let businesses connect your platform to their own tools via public REST API, webhooks, and pre-built integrations.

MONGOOSE MODELS
ApiKey (M12)
WebhookEndpoint (M12)
WebhookDelivery (M12)
IntegrationConfig (M12)

EVENTS EMITTED
none

EVENTS CONSUMED
ALL events (to trigger webhook delivery)

API ROUTES
/api/developer/api-keys
/api/developer/webhooks
/api/developer/webhooks/:id/test
/api/integrations/email-pipe/inbound
/api/integrations/slack/connect

MONGODB SCHEMAS
// ApiKey.model.ts
{ businessId, name, keyHash, scopes:[String], lastUsedAt }

// WebhookEndpoint.model.ts
{ businessId, url, events:[String], secret, enabled }

// WebhookDelivery.model.ts
{ endpointId, eventType, payload:Object, status, responseCode, attemptCount }

// IntegrationConfig.model.ts
{ businessId, integrationType, config:Object, enabled }

PUBLIC REST API
  - API key management: Generate / revoke keys. Raw key shown once on creation. Store only bcrypt hash in ApiKey doc. Scopes: read:conversations, write:conversations, read:contacts, write:contacts.
  - Conversations API: Full CRUD on conversations and messages. Authenticated via X-API-Key header. Separate middleware from agent JWT.
  - Contacts API: Full CRUD on contacts. Same data as dashboard.
  - Rate limiting: 100 requests/min per API key via Redis counter. Reject with 429 if exceeded.

WEBHOOKS
  - Register webhook endpoints: Business registers a URL + selects event types to subscribe to. Secret generated for HMAC signing.
  - Async delivery via BullMQ: M12 subscribes to ALL EventBus events in init(). For each event, checks WebhookEndpoint docs for subscribers, queues webhookQueue job per endpoint.
  - Retry with exponential backoff: BullMQ retries failed deliveries 5 times: 1m, 5m, 30m, 2h, 12h.
  - Delivery log: WebhookDelivery doc per attempt: status, responseCode, responseBody, durationMs.
  - HMAC signature header: X-Signature: sha256=HMAC(JSON.stringify(payload), endpoint.secret) sent on every delivery.
  - Test delivery: POST /api/developer/webhooks/:id/test sends a sample payload immediately without queuing.

PRE-BUILT INTEGRATIONS
  - Email piping: POST /api/integrations/email-pipe/inbound — parses inbound email from Mailgun or SendGrid webhook. Creates Conversation + Message docs.
  - JWT identity verification: Business generates HMAC(userId, WIDGET_SECRET) server-side. Widget sends to /widget/identify. M2 verifies.

MODULE RULES
- All webhook HTTP calls are async BullMQ jobs — never await fetch() inside an EventBus handler.
- API key is shown raw ONCE on creation. Only the bcrypt hash is ever stored.
