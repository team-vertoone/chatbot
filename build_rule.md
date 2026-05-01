BUILD RULES — paste into every AI module prompt

ISOLATION
- Never import service/class from another module folder
- Never call mongoose.connect() — already done in /core/db.ts
- Never create new Socket.io Server — import namespace from /core/socket.ts
- Never create new BullMQ Queue — import from /core/queue.ts
- Never rewrite auth middleware — import from /core/middleware/
- Never use process.env — import from /core/config.ts
- Never call email/S3 SDK directly — use EmailService and StorageService
- Never create new Express app — export Router, register in init(app)

COMMUNICATION
- EventBus.emit('module.entity.action', payload) to broadcast
- EventBus.on('event', handler) inside init() only
- Add new event types to EventMap in /core/events.ts
- Modules may READ shared Mongoose models. Never write to another module's collection.
- ONLY allowed cross-module import: isTeamOnline(teamId) from M1 (pure utility)

MONGODB RULES
- Own models in /modules/[name]/models/
- Use findOneAndUpdate + $set. Never findOne then save.
- Index businessId on every collection. Index all query fields.
- _id is the only PK. Expose as id via toJSON transform.
- { timestamps: true } on every schema.
- Cursor pagination always — never load full collections.

API CONVENTIONS
- /api/[module]/* — authenticateAgent
- /widget/[module]/* — widgetAuth
- /public/[module]/* — no auth
- Socket events: [module]:[entity].[action]
- Socket rooms: business:{id} and conv:{id} only
- Never emit to socket IDs
