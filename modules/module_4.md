MODULE SPEC: M4 — Contacts
Phase: Phase 1
Purpose: Visitor and customer records. Auto-created from widget events, enrichable by agents.

MONGOOSE MODELS
Contact (core)
Company (M4)

EVENTS EMITTED
contacts.contact.created
contacts.contact.merged

EVENTS CONSUMED
widget.visitor.identified (M2)
widget.conversation.created (M2)

API ROUTES
/api/contacts
/api/contacts/:id
/api/contacts/:id/conversations
/api/contacts/:id/notes
/api/companies
/api/contacts/import
/api/contacts/export

MONGODB SCHEMAS
// Company.model.ts (M4-owned)
{ businessId, name, domain, attributes:Object }

CONTACT RECORDS
  - Auto-create on conversation: Listen to widget.conversation.created — upsert Contact by email+businessId. Idempotent.
  - Custom attributes: Stored as Mixed (flexible Object) in Contact.attributes. Agent adds/edits any key-value pair.
  - Contact notes: Free text in Contact.notes field. Editable by any agent.
  - Contact tags: String array in Contact.tags. Filterable in search.
  - Conversation history: GET /api/contacts/:id/conversations — queries Conversation by contactId.
  - Search and filter: MongoDB text index on name+email. Filter by tags and attributes fields.
  - CSV import: Upload CSV to S3. BullMQ reportQueue job processes rows, upserts by email+businessId.
  - CSV export: Async BullMQ job. Streams cursor to CSV. S3 upload. EmailService sends download link.
  - Merge duplicates: Copies attributes from source, re-parents all Conversations via $set on contactId, deletes source Contact doc. Emits contacts.contact.merged.

COMPANIES
  - Company records: Company doc: name, domain, attributes. Contact links via Contact.companyId.
  - Auto-assign by email domain: On contact upsert, check if Contact.email domain matches any Company.domain — auto-link.

MODULE RULES
- Contact upsert from M2 events must use findOneAndUpdate with {upsert:true} — never findOne then save.
- Contact.attributes is a Mixed type — do not put a fixed schema on it. Let businesses store anything.
