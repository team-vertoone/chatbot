MODULE SPEC: M5 — Routing
Phase: Phase 1
Purpose: Automatically assigns new conversations to the right agent or team based on configurable rules.

MONGOOSE MODELS
RoutingRule (M5)
AgentCapacity (M5)

EVENTS EMITTED
routing.conversation.assigned

EVENTS CONSUMED
widget.conversation.created (M2)
agent.deactivated (M1)

API ROUTES
/api/routing/rules
/api/routing/rules/:id
/api/routing/rules/reorder
/api/routing/test

MONGODB SCHEMAS
// RoutingRule.model.ts
{ businessId, name, priority:Number, conditions:Object, action:Object, enabled:Boolean }

// AgentCapacity.model.ts
{ agentId, businessId, currentCount:Number, maxCount:Number }

ASSIGNMENT LOGIC
  - Round-robin: Redis INCR counter key routing:rr:{businessId}:{teamId}. Picks next agent. Skips offline or at-capacity agents.
  - Rule-based routing: RoutingRule docs evaluated in priority order. Conditions: pageUrl contains X, contactTag is Y, keyword in body. Action: assign to team or agent.
  - Team-first routing: Assign Conversation.assignedTeamId — any online team member can pick up.
  - Auto-assign on first reply: If conversation arrives unassigned, assign to first agent who replies via M3.
  - Agent capacity limiting: AgentCapacity.currentCount tracked. Skip agents at maxCount (default: 10).
  - Working hours check: Call isTeamOnline(teamId) from M1. If nobody online, leave unassigned and notify via M6.
  - Priority routing: If Contact.tags includes "vip", assign to agent with role=admin first.
  - Overflow routing: RoutingRule action can specify a fallbackTeamId if primary team is full.

ADMIN UI
  - Rule builder: Visual list of rules. Drag to reorder (updates priority field). Condition + action editor form.
  - Test a rule: POST /api/routing/test with sample conversation data — returns which rule matches and action taken.

MODULE RULES
- Routing runs inside the widget.conversation.created event handler. Must complete in <50ms. Redis reads only in hot path. Single findByIdAndUpdate at the end to set assignedAgentId.
- Round-robin counter is Redis INCR — atomic, no DB write.
- isTeamOnline() imported from M1 is the only cross-module service import allowed in the entire codebase.
