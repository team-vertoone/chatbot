MODULE SPEC: M6 — Notifications
Phase: Phase 1
Purpose: Ensures agents never miss a message — across browser push, in-app bell, email, and Slack.

MONGOOSE MODELS
NotificationPreference (M6)
NotificationLog (M6)
PushSubscription (M6)

EVENTS EMITTED
none

EVENTS CONSUMED
routing.conversation.assigned (M5)
widget.message.created (M2)
inbox.conversation.closed (M3)
inbox.note.created (M3)

API ROUTES
/api/notifications/preferences
/api/notifications/history
/api/notifications/push-subscribe

MONGODB SCHEMAS
// NotificationPreference.model.ts
{ agentId, channel:String, eventType:String, enabled:Boolean }

// NotificationLog.model.ts
{ agentId, channel, eventType, payload:Object, status, sentAt }

// PushSubscription.model.ts
{ agentId, subscription:Object }

DELIVERY CHANNELS
  - In-app notification bell: Socket emit notification:new to room business:{id}. Frontend shows badge + dropdown of recent notifications.
  - Browser push (Web Push): VAPID keys generated in M0. Agent subscribes via browser API. Subscription stored in PushSubscription doc. Sends via web-push library in BullMQ job.
  - Email notification: On new assigned conversation — BullMQ emailQueue job via EmailService.
  - Sound alert: Client-side only. Frontend plays audio file on notification:new socket event.
  - Unread tab title count: Client-side counter updated from socket events. Updates document.title.
  - Slack webhook: Per-business Slack incoming webhook URL stored in Business settings. BullMQ job POSTs on new unassigned conversations.

PREFERENCES
  - Per-agent settings: Toggle each channel (push, email, sound, slack) per event type (assigned, mention, new_message).
  - Daily digest email: BullMQ repeatable cron job (8am per business timezone) — summarises open conversations per agent.

MODULE RULES
- All sends go through BullMQ notificationQueue — never send synchronously inside an EventBus handler.
- Only notify the assigned agent (or admins for unassigned). Never broadcast to all agents.
