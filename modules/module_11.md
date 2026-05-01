MODULE SPEC: M11 — Proactive messaging
Phase: Phase 2
Purpose: Let businesses reach out to visitors first based on behaviour triggers or scheduled campaigns.

MONGOOSE MODELS
ProactiveRule (M11)
ProactiveCampaign (M11)
CampaignSend (M11)

EVENTS EMITTED
proactive.message.sent

EVENTS CONSUMED
widget.visitor.session.active (M2)

API ROUTES
/api/proactive/rules
/api/proactive/campaigns
/widget/proactive/check

MONGODB SCHEMAS
// ProactiveRule.model.ts
{ businessId, name, conditions:Object, message:String, enabled:Boolean }

// ProactiveCampaign.model.ts
{ businessId, name, filterConditions:Object, message, channel, scheduledAt, status }

// CampaignSend.model.ts
{ campaignId, contactId, sentAt, openedAt, repliedAt }

TRIGGER-BASED
  - Trigger rules: ProactiveRule conditions: timeOnPage > N seconds, scrollPct > N%, pageUrl matches pattern, exitIntent is true.
  - Widget polls /widget/proactive/check: Every 10 seconds, widget sends { timeOnPage, scrollPct, pageUrl, exitIntent }. Server evaluates rules from Redis cache.
  - Rules cached in Redis: On rule save, JSON.stringify rules to Redis key proactive:rules:{businessId}. Check endpoint reads Redis — no DB query.
  - One trigger per session: VisitorSession.shownProactiveRuleIds[] tracks which rules have been shown. Never repeat in same session.
  - Exit-intent detection: Widget detects mouse leaving viewport top (mouseleave event on document). Sends exitIntent:true on next poll.

CAMPAIGNS
  - Targeted message campaigns: Filter contacts by attributes. Matching contacts receive proactive message on next widget visit or via email.
  - Scheduled campaigns: ProactiveCampaign.scheduledAt field. BullMQ delayed job triggers the send.
  - Email campaign: BullMQ job iterates matching contacts in batches of 100 using MongoDB cursor. Sends via EmailService.
  - Campaign analytics: CampaignSend doc per contact: sentAt, openedAt (pixel or link click), repliedAt.

MODULE RULES
- Proactive check endpoint must respond in <50ms — always serve from Redis cache, never query MongoDB on each poll.
- Campaign sends iterate contacts using a MongoDB cursor in batches of 100 — never load all contacts into memory.
