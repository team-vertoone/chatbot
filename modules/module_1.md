MODULE SPEC: M1 — Business & auth
Phase: Phase 1
Purpose: Businesses signing up, agent accounts, roles, JWT login/refresh, and team management. Your super-admin panel lives here too.

MONGOOSE MODELS
Business (core)
Agent (core)
Team (core)
Invitation (M1-owned)

EVENTS EMITTED
business.created
agent.created
agent.deactivated
team.created

EVENTS CONSUMED
none

API ROUTES
/api/auth/login
/api/auth/logout
/api/auth/refresh
/api/auth/me
/api/businesses (CRUD)
/api/agents (CRUD)
/api/teams (CRUD)
/api/invitations

MONGODB SCHEMAS
// Invitation.model.ts (M1-owned)
{ businessId, email, role, token (unique), expiresAt, acceptedAt }

BUSINESS MANAGEMENT
  - Create / edit business: Name, slug (unique), timezone, logo URL, widgetConfig object in Business doc.
  - Widget config editor: PATCH /api/businesses/:id/widget — updates widgetConfig fields served to widget on init.
  - Super-admin view: Platform-owner route to list all businesses, toggle active, view usage. Protected by role=superadmin.

AUTH
  - Email + password login: bcrypt (12 rounds). Returns JWT access token + sets httpOnly refresh cookie.
  - JWT access token (15min): Payload: { agentId, businessId, role }. Verified by authenticateAgent middleware.
  - Refresh token rotation: POST /api/auth/refresh — validates cookie, issues new pair, removes old token from Agent.refreshTokens array.
  - Logout: Removes refresh token from Agent.refreshTokens, clears cookie.
  - Invite agent by email: Creates Invitation doc with one-time token, sends email via EmailService.
  - Accept invitation: Agent sets password, Invitation.acceptedAt set, Agent doc created.

AGENT MANAGEMENT
  - List / edit agents: Admin updates name, role, status per agent.
  - Deactivate agent: Sets Agent.status=deactivated, clears all refreshTokens. History preserved.
  - Agent working hours: Per-agent availability stored in Agent — consumed by M5 routing logic.

TEAMS
  - Create / edit teams: Team name, add/remove agents in Team.memberIds array.
  - Team working hours: Team.workingHours schedule — exported as isTeamOnline(teamId) helper for M5 and M10.

MODULE RULES
- M1 is the ONLY module that calls signAccessToken() and signRefreshToken().
- Export isTeamOnline(teamId):boolean as a pure utility — this is the one function M5 and M10 are allowed to import from another module.
