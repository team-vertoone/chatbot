MODULE SPEC: M10 — CSAT & SLA
Phase: Phase 2
Purpose: Customer satisfaction ratings and service level agreement enforcement with automated alerts.

MONGOOSE MODELS
CsatSurvey (M10)
CsatResponse (M10)
SlaPolicy (M10)
SlaBreach (M10)

EVENTS EMITTED
csat.survey.sent
sla.breach.warning
sla.breach.occurred

EVENTS CONSUMED
inbox.conversation.closed (M3)
widget.message.created (M2)
routing.conversation.assigned (M5)

API ROUTES
/api/csat/settings
/api/csat/responses
/api/sla/policies
/api/sla/report
/public/csat/:token

MONGODB SCHEMAS
// CsatSurvey.model.ts
{ businessId, conversationId, token, questionText, sentAt, used:Boolean }

// CsatResponse.model.ts
{ surveyId, rating:Number, comment, submittedAt }

// SlaPolicy.model.ts
{ businessId, name, firstReplyMinutes, resolutionMinutes, appliesTo:Object }

// SlaBreach.model.ts
{ conversationId, policyId, breachType, breachedAt, responseMinutes }

CSAT
  - Post-chat survey send: Listen to inbox.conversation.closed. Queue BullMQ notificationQueue job with 5-min delay to send survey email.
  - Custom question text: Per-business CsatSurvey.questionText. Configurable in dashboard settings.
  - Public survey form: GET /public/csat/:token — no auth. One-time token renders rating form. POST submits CsatResponse.
  - 1–5 rating + comment: Rating integer and optional comment text in CsatResponse doc.
  - Low CSAT alert: If rating <= 2, emit via EventBus → M6 notifies business admin.
  - CSAT in reporting: CsatResponse data feeds into M9 agent performance snapshot as csatAvg.

SLA
  - SLA policies: SlaPolicy doc per business: firstReplyMinutes, resolutionMinutes, and which conversations it applies to.
  - SLA timer on assignment: On routing.conversation.assigned — create BullMQ delayed job with jobId sla:{convId}:first-reply at firstReplyMinutes deadline.
  - Breach warning at 80%: Second BullMQ delayed job at 80% of target — emits sla.breach.warning, M6 notifies agent.
  - Breach record on exceed: If target exceeded, write SlaBreach doc, add slaBreached:true to Conversation via $set.
  - Business hours pause: Use isTeamOnline() from M1. If outside hours, store elapsed time in Redis and resume clock when team comes online.
  - SLA compliance report: Reads SlaBreach collection. % conversations meeting target. Feeds into M9 snapshots.

MODULE RULES
- SLA BullMQ jobs use jobId sla:{conversationId}:{type} so they can be cancelled by ID when conversation resolves (queue.remove(jobId)).
- CSAT token is nanoid(16). Mark CsatSurvey.used=true immediately on form load — reject resubmissions.
