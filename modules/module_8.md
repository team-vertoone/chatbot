MODULE SPEC: M8 — Help center
Phase: Phase 2
Purpose: Public self-service knowledge base businesses host for their customers.

MONGOOSE MODELS
HcCollection (M8)
HcArticle (M8)
HcFeedback (M8)

EVENTS EMITTED
none

EVENTS CONSUMED
none

API ROUTES
/api/helpcenter/collections (CRUD)
/api/helpcenter/articles (CRUD)
/public/hc/:businessSlug
/public/hc/:businessSlug/articles/:slug
/public/hc/:businessSlug/search

MONGODB SCHEMAS
// HcCollection.model.ts
{ businessId, title, slug, orderIndex:Number, isInternal:Boolean }

// HcArticle.model.ts
{ businessId, collectionId, title, slug, contentJson:Object, metaTitle, metaDescription, status, isInternal, viewCount }

// HcFeedback.model.ts
{ articleId, rating:Number, visitorId:String }

CONTENT MANAGEMENT
  - Collections + articles: Two-level hierarchy: HcCollection → HcArticle. orderIndex field, drag to reorder in UI.
  - Rich article editor: Tiptap editor in React. Content stored as JSON in HcArticle.contentJson. Rendered server-side to HTML for public pages.
  - Draft / published / archived: HcArticle.status enum. Only published + non-internal articles show on public site.
  - SEO fields: metaTitle, metaDescription, canonicalUrl per article.
  - Internal articles: HcArticle.isInternal=true — visible in agent search only, hidden from public site.

PUBLIC SITE
  - Public routes — no auth: Express serves /public/hc/:businessSlug/* with no middleware. These are the only fully public routes.
  - Full-text search: MongoDB text index on HcArticle.title + body text. /public/hc/:slug/search?q= returns results.
  - Article feedback: POST /public/hc/feedback — stores HcFeedback doc (rating 1 or -1, visitorId from session).
  - In-widget suggestions: Widget calls /public/hc/:slug/search?q= with visitor's typed query. Shows top 3 results in widget before they send message.

ANALYTICS
  - View counter: Redis INCR hc:views:{articleId} on each page load. BullMQ job flushes to HcArticle.viewCount every 60s.

MODULE RULES
- Public help center routes (/public/*) have NO auth middleware — only routes in the system with no auth besides /widget/*.
- View count uses Redis buffer + periodic flush — never increment MongoDB on every page view directly.
