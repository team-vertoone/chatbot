MODULE SPEC: M13 — Security
Phase: Phase 3
Purpose: Enterprise-grade security features and GDPR compliance tooling.

MONGOOSE MODELS
TotpSecret (M13)
SsoConfig (M13)
AuditLog (M13)
DataDeletionRequest (M13)

EVENTS EMITTED
none

EVENTS CONSUMED
All auth events

API ROUTES
/api/security/2fa/setup
/api/security/2fa/verify
/api/security/sso
/api/security/audit-log
/api/security/gdpr/export
/api/security/gdpr/delete

MONGODB SCHEMAS
// TotpSecret.model.ts
{ agentId, secretEncrypted:String, enabled:Boolean }

// SsoConfig.model.ts
{ businessId, provider, config:Object, enabled }

// AuditLog.model.ts (append-only)
{ businessId, agentId, action, targetType, targetId, metadata:Object, ip, createdAt }

// DataDeletionRequest.model.ts
{ businessId, contactId, requestedBy, confirmationToken, confirmedAt, executedAt }

AUTH HARDENING
  - TOTP 2FA (Google Authenticator): speakeasy library. Setup: generate secret, return QR code URL. Verify: check 6-digit code on login. Store encrypted secret in TotpSecret doc.
  - Enforce 2FA org-wide: Business.require2fa flag. authenticateAgent middleware checks if agent has TotpSecret.enabled=true when flag is set.
  - Google OAuth login: passport.js Google strategy. On callback, match email to existing Agent. No new agent creation.
  - Microsoft OAuth login: passport.js Microsoft strategy. Same flow as Google.
  - IP allowlist: Business.ipAllowlist:[String] array. authenticateAgent middleware checks req.ip — returns 403 if not in list.

GDPR TOOLS
  - Export contact data: Async BullMQ job: collect all docs referencing contactId across all collections. Zip as JSON. S3 upload. Email download link to admin.
  - Delete contact: Requires email confirmation token sent to admin. On confirm: delete Contact doc, anonymise Message.senderId for visitor messages, delete VisitorSessions.
  - Conversation retention policy: Business.retentionDays setting. BullMQ repeatable cron deletes Conversations older than N days.

AUDIT LOG
  - Audit logging: Middleware wrapping sensitive routes. On success: appends AuditLog doc with agentId, action, targetType, targetId, ip, metadata, timestamp.
  - Audit log viewer: GET /api/security/audit-log — cursor-paginated, filterable by agent, action, date range.

MODULE RULES
- AuditLog collection is append-only. No UPDATE or DELETE operations on it ever, anywhere in the codebase.
- Data deletion requires an email-confirmed token before executing. Never delete on a single API call.
