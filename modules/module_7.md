MODULE SPEC: M7 — Ticketing
Phase: Phase 2
Purpose: For complex issues that need a structured workflow, ownership, and deadline tracking.

MONGOOSE MODELS
Ticket (M7)
TicketComment (M7)
TicketHistory (M7)

EVENTS EMITTED
ticketing.ticket.created
ticketing.ticket.overdue

EVENTS CONSUMED
inbox.conversation.closed (M3)

API ROUTES
/api/tickets
/api/tickets/:id
/api/tickets/:id/comments
/api/tickets/:id/conversations
/public/tickets/:publicId

MONGODB SCHEMAS
// Ticket.model.ts
{ businessId, contactId, title, status, priority, assignedAgentId, conversationIds:[ObjectId], dueAt, publicId, createdAt }

// TicketComment.model.ts
{ ticketId, agentId, body, isInternal:Boolean }

// TicketHistory.model.ts (append-only)
{ ticketId, field, oldValue, newValue, changedBy, changedAt }

TICKETS
  - Create from conversation: POST /api/tickets with conversationId — creates Ticket doc, stores conversationId in ticket.conversationIds[].
  - Custom statuses: Per-business status list in Business.ticketStatuses array (added to Business doc). Default: open, in-progress, blocked, resolved.
  - Priority levels: low / normal / high / urgent enum on Ticket doc. Color-coded in UI.
  - Assign + due date: assignedAgentId and dueAt fields. BullMQ repeatable job checks overdue tickets every hour.
  - Internal comments: TicketComment docs. isInternal:Boolean flag separates private vs customer-visible comments.
  - History log: Immutable TicketHistory doc appended on every status/assignment/priority change.
  - Customer status page: GET /public/tickets/:publicId — shows status with no login required. publicId is nanoid(12).
  - Ticket templates: Pre-fill title and description for common issue types. Stored in Business.ticketTemplates[].

MODULE RULES
- Customer-facing public URL uses publicId (nanoid, not MongoDB _id) — never expose internal IDs publicly.
- Overdue check is a BullMQ repeatable job (every 1 hour) — not a cron expression, not setTimeout.
