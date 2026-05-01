MODULE SPEC: M2 — Chat widget
Phase: Phase 1
Purpose: The embeddable JS widget and the backend that powers real-time chat between visitors and agents.

MONGOOSE MODELS
VisitorSession (core)
Conversation (core)
Message (core)

EVENTS EMITTED
widget.conversation.created
widget.message.created
widget.visitor.identified

EVENTS CONSUMED
routing.conversation.assigned (M5)

API ROUTES
/widget/init
/widget/conversations
/widget/conversations/:id/messages
/widget/identify
/widget/upload-url
/widget/proactive/check

MONGODB SCHEMAS
// No new models — uses VisitorSession, Conversation, Message from M0

WIDGET BUNDLE (VANILLA TS)
  - Self-initializing script: <script src="widget.js" data-business-id="xxx"> reads attribute, calls /widget/init on load.
  - Shadow DOM isolation: All widget HTML/CSS injected into a Shadow DOM — zero conflict with host website styles.
  - Launcher button: Floating button. Color, position, icon read from widgetConfig returned by /widget/init.
  - Chat window UI: Open/close animation, message list, input box, send button — pure DOM manipulation.
  - Pre-chat form: If widgetConfig.requireEmail=true, show name+email form before first message.
  - Offline form: If init response says no agents online, show leave-a-message form instead of live chat.
  - Unread badge: Unread count stored in localStorage, shown on launcher on revisit.
  - Typing indicator: Debounced socket emit on keypress. Agent sees visitor typing via /dashboard namespace.
  - File upload: Gets presigned URL from /widget/upload-url, uploads directly to S3, sends URL as message body.
  - Session persistence: sessionToken stored in localStorage — re-used on revisit to restore conversation.
  - Sound on new message: Plays short audio file on incoming message.new socket event.

BACKEND
  - POST /widget/init: Validates business widget token. Returns widgetConfig, agentsOnline status, existing session if any. Creates VisitorSession doc.
  - widgetAuth middleware: Validates sessionToken (different secret from agent JWT). Attaches req.session. Used on all /widget/* routes.
  - POST /widget/identify: Verifies HMAC(userId, widgetSecret) sent from business server. Links VisitorSession to Contact by userId/email. Emits widget.visitor.identified.
  - Message send + persist: POST /widget/conversations/:id/messages — saves Message doc, emits widget.message.created, broadcasts to conv:{id} room on /dashboard.
  - Socket /widget namespace: On connect: verify sessionToken, join room conv:{conversationId}. Handle message.send events in real time.

MODULE RULES
- Widget bundle is a SEPARATE Vite build outputting widget.js — never import dashboard React components into it.
- sessionToken is signed with WIDGET_SECRET, not JWT_SECRET. widgetAuth and authenticateAgent are completely separate.
- Widget CSS must live inside Shadow DOM — never inject styles into the host page.
