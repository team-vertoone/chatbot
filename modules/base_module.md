MODULE SPEC: M0 — Project base
Phase: Foundation
Purpose: Shared infrastructure. No business features — just the foundation every module builds on.

MONGOOSE MODELS
Business
Agent
Team
Conversation
Message
Contact
VisitorSession

EVENTS EMITTED
none

EVENTS CONSUMED
none

API ROUTES
No routes — infrastructure only

MONGODB SCHEMAS
// All 7 core models defined here
// See Foundation tab for full schemas

WHAT TO BUILD
  - mongoose.connect(): Single connection in /core/db.ts using MONGODB_URI from config.
  - ioredis client: Single client in /core/redis.ts. Exported and shared across all modules.
  - Socket.io server: /core/socket.ts — attach to HTTP server, create two namespaces, Redis adapter, export both.
  - BullMQ queues: /core/queue.ts — emailQueue, notificationQueue, webhookQueue, reportQueue. Exported.
  - Typed EventBus: /core/events.ts — EventEmitter + TypeScript EventMap interface for all events.
  - Core Mongoose models: Business, Agent, Team, Conversation, Message, Contact, VisitorSession — all with indexes and toJSON id transform.
  - Auth helpers: /core/auth.ts — signAccessToken(), signRefreshToken(), verifyAccessToken(), verifyRefreshToken().
  - Shared middleware: authenticateAgent, requireRole, widgetAuth, validate(zodSchema), rateLimiter.
  - Response helpers: sendSuccess(res,data,meta?) and sendError(res,code,msg). Every route uses these.
  - Config loader: /core/config.ts — zod validates all env vars on startup. Crash-fast if any missing.
  - Module init system: app.ts imports every module and calls init(app,io) in order at startup.
  - EmailService: /core/email.ts — sendEmail({to,subject,template,data}) wrapping Resend or SendGrid.
  - StorageService: /core/storage.ts — upload(), getPresignedUrl(), deleteFile() wrapping S3 or R2.

MODULE RULES
- Build and fully test this before touching any module.
- Add new event types to EventMap in /core/events.ts as modules are built.
- Core model field names are immutable after M0 is set — modules may add fields via their own logic but never rename core fields.
