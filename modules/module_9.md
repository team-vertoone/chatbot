MODULE SPEC: M9 — Reporting
Phase: Phase 2
Purpose: Dashboards and charts for businesses to understand team performance and conversation patterns.

MONGOOSE MODELS
ReportSnapshot (M9)

EVENTS EMITTED
none

EVENTS CONSUMED
widget.conversation.created (M2)
widget.message.created (M2)
inbox.conversation.closed (M3)

API ROUTES
/api/reports/live
/api/reports/volume
/api/reports/agents
/api/reports/tags
/api/reports/export

MONGODB SCHEMAS
// ReportSnapshot.model.ts
{ businessId, metric:String, periodDate:Date, value:Number, breakdown:Object }

LIVE DASHBOARD (REDIS-BASED)
  - Active conversations count: Redis counter reports:active:{businessId}. M2 increments on create, M3 decrements on resolve.
  - Agents online count: Redis presence keys set by /dashboard socket on connect/disconnect.
  - Current avg wait time: MongoDB aggregation on open conversations: avg(now minus updatedAt) where last sender = visitor. Run on demand — small dataset.

HISTORICAL REPORTS (PRECOMPUTED)
  - Conversation volume chart: Reads ReportSnapshot docs by metric='volume' and periodDate. BullMQ reportQueue cron builds daily.
  - First response time: Reads from Conversation.firstResponseAt (set by M3). Aggregated into daily snapshot.
  - Resolution time: Conversation.resolvedAt minus Conversation.createdAt. Aggregated into daily snapshot.
  - Agent performance table: Per-agent snapshot: conversationsHandled, avgFirstResponse, avgResolution, csatAvg (joined from M10 data).
  - Tag report: Count conversations grouped by tags array values. Aggregated into daily snapshot.
  - Custom date range: All report endpoints accept ?from= and ?to= ISO params. Query ReportSnapshot by periodDate range.
  - CSV export: BullMQ reportQueue async job. Streams MongoDB cursor to CSV. Uploads to S3. Emails download link.

MODULE RULES
- NEVER run heavy aggregations on live API calls for historical data. All historical reports read from ReportSnapshot.
- ReportSnapshot is built by a BullMQ repeatable job (daily at midnight, per business timezone).
- Live counters always read from Redis — never MongoDB.
