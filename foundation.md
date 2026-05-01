

| Tech stack | Description | 
| :--- | :---: |
| Runtime | Node.js 20+ with TypeScript (strict mode). ts-node-dev in dev, compiled JS in prod. |
| Backend | Express.js. Single app instance. Each module exports a router registered via app.use() in app.ts. |
| Database | MongoDB with Mongoose. Single mongoose.connect() in /core/db.ts. MongoDB Atlas in production. All models defined with Mongoose schemas + TypeScript interfaces. |
| Real-time | Socket.io attached to the same HTTP server. Two namespaces only: /widget (visitors) and /dashboard (agents). Never create a second Socket.io instance. |
| Cache/ pub-sub | Redis via ioredis. Used for sessions, presence, round-robin counters, rate limiting, and Socket.io adapter for horizontal scaling. |
| Background jobs | BullMQ on Redis. Four queues created once in M0: emailQueue, notificationQueue, webhookQueue, reportQueue. Modules only call queue.add() — never create new Queue instances. |
| Frontend | React 18 + Vite + TypeScript. Zustand for client state. TanStack Query for server state. TailwindCSS + shadcn/ui for UI. |
| Widget | Vanilla TypeScript compiled to a single bundle via Vite lib mode. No React. No external dependencies. CSS injected into Shadow DOM to prevent host-site conflicts. |
| Auth | JWT access tokens (15min) + refresh tokens (30 days) stored in httpOnly cookies. Single auth system in M0/M1 — never re-implement in any other module.|
| File storage | AWS S3 or Cloudflare R2. Presigned URLs for direct browser upload. All access through StorageService in /core/storage.ts.|
| Email | Resend or SendGrid. All sending through EmailService in /core/email.ts — never call SDK directly from modules. |
| Validation | Zod for all request validation and env config. Shared validate(schema) middleware in /core/middleware/validate.ts.|


Folder structure
```
/
├── server/
│   └── src/
│       ├── core/
│       │   ├── config.ts        ← zod env validation, single import point
│       │   ├── db.ts            ← mongoose.connect()
│       │   ├── redis.ts         ← ioredis client export
│       │   ├── socket.ts        ← socket.io init, /widget + /dashboard namespaces
│       │   ├── queue.ts         ← bullmq: emailQueue, notificationQueue, webhookQueue, reportQueue
│       │   ├── events.ts        ← typed EventBus + EventMap interface
│       │   ├── email.ts         ← EmailService class
│       │   ├── storage.ts       ← StorageService class (S3/R2)
│       │   └── middleware/
│       │       ├── auth.ts      ← authenticateAgent, requireRole
│       │       ├── widgetAuth.ts← visitor session validation
│       │       ├── rateLimiter.ts
│       │       └── validate.ts  ← validate(zodSchema)
│       ├── modules/
│       │   ├── business/        ← M1
│       │   ├── widget/          ← M2
│       │   ├── inbox/           ← M3
│       │   ├── contacts/        ← M4
│       │   ├── routing/         ← M5
│       │   ├── notifications/   ← M6
│       │   ├── ticketing/       ← M7
│       │   ├── helpcenter/      ← M8
│       │   ├── reporting/       ← M9
│       │   ├── csat-sla/        ← M10
│       │   ├── proactive/       ← M11
│       │   ├── integrations/    ← M12
│       │   ├── security/        ← M13
│       │   └── whitelabel/      ← M14
│       └── app.ts               ← registers all module routers + calls init()
├── client/                      ← React dashboard
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── store/               ← zustand stores
│       ├── hooks/               ← tanstack query hooks
│       ├── lib/                 ← api client, socket client
│       └── modules/             ← per-module UI
└── widget/                      ← standalone vanilla JS widget
    └── src/
        ├── widget.ts            ← entry, reads data-business-id
        ├── ui.ts                ← DOM / Shadow DOM
        ├── socket.ts            ← socket.io-client
        └── api.ts               ← fetch to /widget/*
```

Core Mongoose models (defined in M0 — never rename these)

```
Business   { name, slug, widgetConfig{color,position,welcomeMsg,requireEmail}, settings{timezone,workingHours}, createdAt }
Agent      { businessId, email, name, role(admin|agent|viewer), status(online|away|offline), passwordHash, refreshTokens[{token,expiresAt}] }
Team       { businessId, name, memberIds[AgentId], workingHours{enabled,schedule} }
Conversation { businessId, contactId, status(open|pending|snoozed|resolved), channel(widget|email), assignedAgentId, assignedTeamId, tags[String], snoozedUntil, firstResponseAt, resolvedAt, createdAt, updatedAt }
Message    { conversationId, senderType(visitor|agent|system), senderId, body, attachments[{url,name,type}], readAt, createdAt }
Contact    { businessId, email, name, phone, attributes(Mixed), notes, tags[String], companyId, createdAt }
VisitorSession { businessId, contactId, socketId, pageUrl, userAgent, ip, country, lastSeen }
```

Quick reminder on order:
* M0 first, fully working
* Then M1 → M2 → M3 → M4 → M5 → M6 in that order
* After Phase 1 is stable, M7 through M12 can be given to separate AI sessions in parallel
* M13 and M14 last
