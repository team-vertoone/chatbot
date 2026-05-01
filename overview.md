BUILD ORDER OVERVIEW

FOUNDATION (build before any module)
  M0 — Project base: Project setup, DB schema, auth system, shared utilities, event bus

PHASE 1 — Core product (build in order)
  * M1 — Business & auth: Handles businesses signing up, agents being invited, roles, and login. Your platform admin can see all businesses here too.
  * M2 — Chat widget: The embeddable JS snippet + backend that powers real-time chat between visitors and agents. The heart of the whole product.
  * M3 — Agent inbox: The main agent-facing UI. Everything agents do with conversations lives here.
  * M4 — Contacts: Visitor and customer records. Built automatically from widget sessions, enrichable by agents.
  * M5 — Routing & assignment: Decides which agent or team gets each new conversation automatically.
  * M6 — Notifications: Alerts agents about new messages and events across all channels.

PHASE 2 — Growth (can build in parallel after Phase 1)
  * M7 — Ticketing: For complex issues that need tracking, deadlines, and status workflows beyond a simple conversation.
  * M8 — Help center: Public self-service knowledge base businesses host for their customers.
  * M9 — Reporting: Dashboards and charts so businesses can understand team performance and conversation patterns.
  * M10 — CSAT & SLA: Collect customer satisfaction ratings and enforce response time commitments.
  * M11 — Proactive messaging: Let businesses reach out to visitors first, based on behaviour triggers or scheduled campaigns.
  * M12 — Integrations & API: Let businesses connect your platform to their own tools via REST API, webhooks, and pre-built integrations.

PHASE 3 — Polish
  * M13 — Security & compliance: Enterprise-grade security features and GDPR compliance tooling.
  * M14 — White-label & billing: Let businesses customize the platform fully and give you a way to track usage for billing.
