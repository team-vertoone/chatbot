MODULE SPEC: M3 — Agent inbox
Phase: Phase 1
Purpose: The main agent-facing UI. Everything agents do with conversations lives here.

MONGOOSE MODELS
Conversation (core)
Message (core)
CannedResponse (M3)
InternalNote (M3)

EVENTS EMITTED
inbox.conversation.closed
inbox.conversation.assigned
inbox.conversation.reopened
inbox.note.created

EVENTS CONSUMED
widget.conversation.created (M2)
widget.message.created (M2)
routing.conversation.assigned (M5)

API ROUTES
/api/inbox/conversations
/api/inbox/conversations/:id
/api/inbox/conversations/:id/messages
/api/inbox/conversations/:id/assign
/api/inbox/conversations/:id/status
/api/inbox/conversations/:id/snooze
/api/inbox/conversations/:id/tags
/api/inbox/notes
/api/inbox/canned-responses
/api/inbox/search

MONGODB SCHEMAS
// CannedResponse.model.ts
{ businessId, shortcut, title, body, createdBy }

// InternalNote.model.ts
{ conversationId, agentId, body, mentions:[AgentId], createdAt }

INBOX UI (REACT)
  - Conversation list panel: Left panel sorted by updatedAt. Cursor-paginated. Live updates via socket. Filter by status, agent, team, tag.
  - Conversation detail panel: Full message thread, cursor-paginated upward scroll. Real-time new messages via inbox:message.new socket.
  - Reply box: Textarea with / shortcut to search canned responses, attach files, send on Enter or button.
  - Internal notes: Toggle between Reply and Note mode. Notes saved to InternalNote collection, never shown to visitor.
  - @mention in notes: @agentName triggers mention, stored in InternalNote.mentions[], triggers notification in M6.
  - Canned responses: / triggers search dropdown over saved templates. Insert into reply box on select.
  - Assign conversation: Dropdown to pick agent or team. PATCH /api/inbox/conversations/:id/assign. Emits inbox.conversation.assigned.
  - Change status: Open → Pending → Resolved. Resolved emits inbox.conversation.closed. Sets Conversation.resolvedAt.
  - Tag conversation: Multi-select tag picker. Updates Conversation.tags array with $set.
  - Snooze: Sets Conversation.snoozedUntil. BullMQ delayed job re-opens at that time.
  - Bulk actions: Checkbox select multiple conversations → bulk assign, tag, resolve.
  - Collision detection: Track agents in socket room conv:{id}. If 2 agents join, emit inbox:agent.collision warning to both.
  - Search: MongoDB text index on Message.body. GET /api/inbox/search?q= returns matching conversations.
  - Contact panel (sidebar): Fetches /api/contacts/:id — M3 calls the API, does NOT import the contacts module service.

MODULE RULES
- Snooze re-open is a BullMQ notificationQueue delayed job — never use setTimeout.
- Collision detection uses socket room tracking — no DB write needed.
- Full-text search requires a text index on Message.body defined in the Message schema in M0.
- Conversation.firstResponseAt is set here when the first agent message is sent — used by M9 reporting.
