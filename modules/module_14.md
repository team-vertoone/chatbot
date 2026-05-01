MODULE SPEC: M14 — White-label & billing
Phase: Phase 3
Purpose: Full client branding customization and per-client usage tracking for your billing.

MONGOOSE MODELS
CustomDomain (M14)
BusinessBranding (M14)

EVENTS EMITTED
none

EVENTS CONSUMED
All (for usage metering)

API ROUTES
/api/whitelabel/domain
/api/whitelabel/branding
/api/billing/usage

MONGODB SCHEMAS
// CustomDomain.model.ts
{ businessId, domain, verified:Boolean, sslStatus:String, verifiedAt }

// BusinessBranding.model.ts
{ businessId, hidePoweredBy:Boolean, customSenderName, widgetDomain }

WHITE-LABEL
  - Custom domain for help center: Business sets CNAME to your server. SSL via Let's Encrypt (node-acme-client). CustomDomain doc tracks verification + sslStatus. Provisioning is async BullMQ job.
  - Remove powered-by branding: BusinessBranding.hidePoweredBy flag. Widget bundle and help center check this before rendering the badge.
  - Custom email sender name: BusinessBranding.customSenderName. EmailService uses this as the From name in all emails for this business.
  - Custom widget domain: Widget served from business-specific subdomain rather than your CDN URL.

USAGE METERING
  - Daily usage snapshot: BullMQ repeatable cron (daily) counts Conversations, Messages, Agents per business. Writes to ReportSnapshot in M9 with metric='usage'.
  - Admin billing dashboard: Super-admin reads ReportSnapshot metric=usage across all businesses for billing decisions.
  - Plan limit alerts: If businessConversations > Business.planLimit, EmailService notifies platform admin.

MODULE RULES
- SSL provisioning is async — show sslStatus:pending in dashboard, update to active or failed via BullMQ job.
- Usage data reads from ReportSnapshot written by M9. M14 never runs its own aggregation pipelines.
