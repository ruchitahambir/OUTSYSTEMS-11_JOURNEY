# ODC Core Reference

## Table of Contents
1. ODC Architecture vs O11
2. ODC Studio & Apps vs Libraries
3. Domain-Driven Design in ODC
4. REST & SOAP in ODC
5. Roles & Admin in ODC
6. SQL & Data in ODC
7. Error Handling in ODC (Screens, Agents, Libraries)
8. Agents: Architecture, Components, Flow
9. ODC Portal
10. ODC Forge

---

## 1. ODC Architecture vs O11

### What changed
ODC is cloud-native, Kubernetes-based, built for microservices-style development.
NOT an upgrade from O11 — it is a completely different product.

### Key architectural differences
```
O11:                          ODC:
Module-based                  App-based (each app = microservice-like)
4-Layer Canvas                Domain-Driven Design
SQL Server / Oracle           Aurora PostgreSQL (managed, cloud-only)
Service Studio                ODC Studio (browser-based)
Service Center                ODC Portal
LifeTime                      ODC Portal (also handles deployment)
eSpace/Module                 App or Library
Traditional Web support       Reactive Web ONLY (no Traditional)
SOAP full support             SOAP consume via Integration Builder only
Self-hosted option            Cloud-only (AWS)
```

### O11 concepts mapped to ODC
| O11 | ODC equivalent |
|-----|---------------|
| Module | App |
| Service Module | Library (shared logic/UI) |
| Server Action (cross-module) | Service Action (own transaction, REST-like) |
| LifeTime | ODC Portal |
| Service Center | ODC Portal |
| Site Properties | Settings (in ODC Portal) |
| Extension | External Library |
| Forge | ODC Forge (separate catalog) |

### Interview spoken answer
"ODC is OutSystems' cloud-native next-generation platform. The biggest shift from O11 is the move from module-based to app-based architecture, where each app behaves like a microservice. Service Actions in ODC run in their own transaction, making them more like REST calls between services. The monitoring and deployment tool is now ODC Portal instead of LifeTime and Service Center. The database is Aurora PostgreSQL — managed, you don't touch it directly."

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/

---

## 2. ODC Studio & Apps vs Libraries

### Apps
- Standalone deployable unit with its own screens, logic, and data
- Has its own database scope
- Can expose Service Actions for other apps to consume

### Libraries
- Reusable logic/UI shared across apps
- No database — cannot have entities
- Can contain: UI blocks, client/server actions, structures, resources
- Think of it like a shared utility package

### When to use each
```
Library: Shared UI theme, common validation functions, utility formatters
App: A full application with screens, users, and data (e.g. Ticket Manager, HR Portal)
```

### Module structure in ODC Studio
```
App:
  ├── UI Flows (screens)
  ├── Logic (Client Actions, Server Actions, Service Actions, Timers)
  ├── Data (Entities, Structures, Static Entities)
  └── Dependencies (other Apps, Libraries, System)

Library:
  ├── Logic (Client/Server Actions — no Service Actions)
  ├── UI Blocks
  └── Structures (no Entities)
```

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/building_apps/

---

## 3. Domain-Driven Design in ODC

### What it means in practice
ODC encourages organizing apps around BUSINESS DOMAINS, not technical layers.

```
Example: SmartServe in ODC

Domain: Ticketing
  App: TicketCore           → Ticket entities, business logic
  App: TicketPortal         → User-facing screens
  Library: TicketComponents → Shared UI blocks, validators

Domain: UserManagement
  App: UserCore             → User entities, auth logic
  Library: UserUtils        → Shared user helpers

Cross-domain:
  TicketPortal consumes UserCore via Service Actions
  Both use TicketComponents library
```

### Service Actions in ODC — key difference from O11
- In O11: Service Actions break the module dependency but still same DB context
- In ODC: Service Actions run in their OWN transaction — truly separate
- This means: caller cannot roll back what a Service Action already committed
- Design implication: handle errors explicitly in both caller and the service action

### Practical impact
```
Scenario: CreateTicket calls AssignAgent (Service Action in another App)
- AssignAgent commits its own DB changes
- If CreateTicket fails AFTER AssignAgent succeeds:
  → AssignAgent change is NOT rolled back
  → You must implement compensating logic (e.g., UnassignAgent on failure)
```

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/building_apps/app_architecture/

---

## 4. REST & SOAP in ODC

### REST in ODC (consuming)
Same concept as O11 but done in ODC Studio.
ODC Studio → Logic → Integrations → REST → Add REST API
Works identically to O11 with same structure/mapping approach.

### REST in ODC (exposing)
ODC apps can expose REST APIs via Service Actions that are published.
ODC Portal → App → API tab → view exposed endpoints.

### SOAP in ODC — IMPORTANT DIFFERENCE
ODC does NOT have native SOAP integration in ODC Studio.
SOAP is available via Integration Builder (separate tool).

How to consume SOAP in ODC:
  Step 1: Go to Integration Builder (integrationbuilder.outsystems.com)
  Step 2: Create new SOAP integration, provide WSDL
  Step 3: Integration Builder generates an OutSystems app/library with the SOAP actions
  Step 4: Add that generated library as dependency in your ODC app
  Step 5: Call the generated actions in your logic

This is a key ODC limitation vs O11 — O11 has native SOAP, ODC needs Integration Builder.

Interview answer on SOAP in ODC:
"In ODC, SOAP is not natively available in ODC Studio like in O11. Instead, you use Integration Builder to import the WSDL, which generates a ready-to-use library. You add that library as a dependency and call the actions normally. It adds one extra step compared to O11 but the end result is similar."

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/integration_with_external_systems/

---

## 5. Roles & Admin in ODC

### How roles work in ODC
Similar concept to O11 but managed through ODC Portal.

```
Role types:
  Built-in: Anonymous, Registered
  Custom: Defined per app (e.g. TicketAgent, Supervisor)

Assign at runtime:
  AssignRole(UserId, RoleName) — built-in action
  RevokeRole(UserId, RoleName) — built-in action

Check in logic:
  HasRole("TicketAgent") → returns Boolean (uses current session user)
```

### ODC Portal admin practices
```
ODC Portal:
  Users section → manage users, assign built-in roles
  App-specific roles → assigned per app in Portal
  Groups → batch role management
  Audit logs → user activity tracking
```

### Key difference from O11
- In O11: roles defined per module, assigned via Server Actions using System APIs
- In ODC: roles defined per app, managed via ODC Portal UI or programmatically

### Protecting screens in ODC
Screen properties → Accessible by → set required roles (same as O11 concept)

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/user_management/

---

## 6. SQL & Data in ODC

### Database: Aurora PostgreSQL
- Managed by OutSystems — you don't have DBA access
- PostgreSQL syntax differences from SQL Server (O11 default):
  - String concat: || instead of +
  - Boolean: true/false instead of 1/0
  - Case sensitive by default (use ILIKE for case-insensitive LIKE)

### Aggregates in ODC
Work the same as O11 — visual query builder, auto-parameterized, safe from injection.
Limitation: no UNION, no complex subqueries.

### Advanced SQL in ODC
```
SAFE pattern:
  SELECT {Ticket}.[Id], {Ticket}.[Title]
  FROM {Ticket}
  WHERE {Ticket}.[Status] = @StatusFilter
  -- @StatusFilter = Input Parameter → auto-escaped

PostgreSQL-specific syntax in ODC:
  -- Case-insensitive search:
  WHERE {Ticket}.[Title] ILIKE '%' || @SearchText || '%'
  
  -- Boolean check (not 1/0 like SQL Server):
  WHERE {Ticket}.[IsActive] = true
```

### SQL injection prevention in ODC
Same principles as O11:
- Use Input Parameters — never concatenate user input
- Use EncodeSql() for dynamic parts
- Aggregates are always safe

### Entity relationships in ODC
- Foreign keys defined via entity attributes (Identifier type)
- Cross-app data: use Service Actions — ODC apps don't share DB access
- External entities: connect via Integration Builder to external DBs

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/data/

---

## 7. Error Handling in ODC (Screens, Agents, Libraries)

### Error types and where they occur

#### Screen errors
```
Type: User-triggered (validation, permission, not found)
Where: Screen actions, on-click handlers, data fetch

Example: Ticket not found when navigating by ID
  Solution:
    → In OnInitialize or Data Action:
    → Fetch ticket by ID
    → If list is empty → navigate to NotFound screen or show message widget
    → Use IsDataFetched + IsEmpty(GetTicket.List) pattern
```

#### Logic/Action errors
```
Type: Business logic failures, unexpected data states

Example: Division by zero, null reference, constraint violation
  Solution:
    → Wrap in Try-Catch (Exception Handler in flow)
    → Log with LogMessage or custom logger
    → Raise meaningful UserException: "Could not save ticket. Please try again."
    → Never expose technical error messages to users
```

#### Integration errors (REST/SOAP)
```
Type: Network timeout, 4xx/5xx responses, malformed data

Pattern:
  Try:
    → Call external API
    → Check response
  Catch: AllExceptions or specific (e.g. HTTPRequestException)
    → Log: method name, input params, error message
    → Return structured error response to screen
    → Screen shows: "External service unavailable. Please try later."
```

#### SQL errors
```
Type: Constraint violation, data type mismatch, query failure

Example: Unique constraint violation (duplicate ticket number)
  Solution:
    → Catch DatabaseException (or check before insert with aggregate)
    → Raise UserException: "Ticket number already exists"
    
Example: Query timeout
  Solution:
    → Check Service Center / ODC Portal for slow query
    → Add index on filtered entity attributes
    → Optimize aggregate — remove unnecessary joins
```

#### Agent errors (ODC-specific)
```
Type: Step failure, tool call failure, infinite loop, timeout

Where: Agent flows, tool actions

Solution:
  → Each agent step should have error handling
  → Use OnError flow in agent to capture failures
  → Log agent execution failures to ODC Portal
  → Set maximum iteration limits on loops
  → Test with edge-case inputs before deployment
```

### Error handling best practices (applies to all)
1. Never show raw exception messages to end users
2. Always log errors with enough context: action name, input values, user ID
3. Use UserException for expected business errors (user can act on it)
4. Use AllExceptions handler for unexpected technical errors
5. Return IsSuccess + ErrorMessage pattern from service actions so callers can handle gracefully
6. Test error paths deliberately — not just the happy path

---

## 8. Agents: Architecture, Components, Flow

### What are ODC Agents
ODC Agents = AI-powered automated workflows that combine LLM reasoning with OutSystems actions.
Introduced via Agent Workbench (GA 2025).
Think of them as: an AI model that can call your OutSystems logic as tools.

### Agent components
```
Agent:
  ├── Instructions        → System prompt defining agent behavior and goal
  ├── Tools               → OutSystems Server Actions exposed to the agent
  ├── Memory              → Optional context storage between runs
  ├── Input               → What triggers the agent (screen action, timer, event)
  └── Output              → What the agent returns to the caller

Tool:
  → A Server Action decorated as a Tool
  → Agent calls it when it decides the tool is needed
  → Must have clear name, description, and typed parameters
  → Example: SearchTickets(Keywords), AssignTicket(TicketId, AgentId), GetUserDetails(UserId)
```

### Agent flow architecture
```
Trigger (User message OR Timer OR Event)
  ↓
Agent receives input + context
  ↓
LLM reasoning: "What tool should I call next?"
  ↓
Call Tool (OutSystems Server Action)
  ↓
Tool returns result
  ↓
LLM processes result, decides next step
  ↓
Loop until goal achieved OR max iterations
  ↓
Return final output to caller
```

### Practical example: SmartServe AI Agent
```
Goal: Automatically triage incoming support tickets

Instructions:
  "You are a support ticket triage agent. Analyze the ticket description,
   categorize it by priority (Low/Medium/High/Critical), and assign it
   to the appropriate team based on category."

Tools:
  - GetTicketDetails(TicketId) → returns ticket description, submitter info
  - GetTeamByCategory(Category) → returns available team
  - AssignTicket(TicketId, TeamId, Priority) → assigns and updates ticket
  - NotifyTeam(TeamId, Message) → sends notification

Flow:
  Trigger: New ticket created (event)
  Agent: GetTicketDetails → analyze → GetTeamByCategory → AssignTicket → NotifyTeam
```

### Agent best practices
- Tools should do ONE thing clearly — single responsibility
- Write clear tool descriptions: the LLM uses them to decide which tool to call
- Set max iterations to prevent infinite loops
- Log every tool call for debugging
- Test agents with edge-case inputs and unusual ticket descriptions
- Handle tool failures gracefully — agent should not crash if one tool fails

### Agent errors and debugging
```
Error: Agent calls wrong tool repeatedly
  Cause: Unclear tool descriptions or overlapping tool names
  Fix: Rewrite tool descriptions to be distinct and specific

Error: Agent loop never terminates
  Cause: Goal condition never satisfied or max iterations not set
  Fix: Set MaxIterations, define clear completion criteria in instructions

Error: Tool action fails
  Cause: Input parameter mismatch or business rule violation
  Fix: Add exception handling in Tool Server Action, return structured error

Error: Agent gives wrong answer
  Cause: LLM hallucination or insufficient context
  Fix: Add more specific instructions, provide example inputs/outputs in prompt
```

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/building_apps/use_ai_within_your_app/

---

## 9. ODC Portal

### What ODC Portal replaces
Replaces: LifeTime (deployment) + Service Center (monitoring) + some admin functions

### Key sections
```
ODC Portal:
  ├── Apps                → Deploy, manage versions, view app health
  ├── Users               → Manage end users, assign roles
  ├── Monitoring          → Logs: errors, traces, integration logs
  ├── Settings            → App settings (like Site Properties in O11)
  ├── Environments        → Dev / QA / Prod pipeline management
  └── Forge               → Install ODC Forge components
```

### Deploying an app in ODC
```
Step 1: Publish from ODC Studio → deploys to Development
Step 2: ODC Portal → Apps → [app] → Deploy to QA → select version
Step 3: Test in QA
Step 4: ODC Portal → Apps → [app] → Deploy to Production
```

### Monitoring in ODC Portal
```
Monitoring → Logs:
  - Filter by app, severity, time range
  - Click log entry → full trace with action names and parameters
  - Integration logs: request/response for REST calls
  - Agent logs: each tool call, LLM reasoning steps

Key difference from Service Center:
  - ODC Portal has distributed traces → see full request path across apps
  - Service Center shows per-module logs → can be harder to correlate
```

### Settings (Site Properties equivalent)
```
ODC Portal → Apps → [app] → Settings → App Settings
  - Set per environment (Dev/QA/Prod can have different values)
  - Example: ExternalAPIBaseURL, MaxUploadSizeMB, FeatureFlag_NewUI
```

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/managing_apps/

---

## 10. ODC Forge

### What is ODC Forge
Separate from O11 Forge — ODC-specific component catalog.
Located at: ODC Portal → Forge tab

### Key differences from O11 Forge
- Components are ODC-native (cannot use O11 Forge components in ODC)
- Smaller catalog than O11 Forge (ODC is newer)
- Growing rapidly — check regularly for new components
- Trust levels: OutSystems Supported, Trusted, Community (same system as O11)

### Installing ODC Forge components
```
Step 1: ODC Portal → Forge → search for component
Step 2: Click Install → select target environment
Step 3: ODC Studio → Manage Dependencies → add the component
Step 4: Use component elements in your app
```

### Wrapping pattern (same as O11)
Always wrap ODC Forge actions in your own Library actions:
  Your Library: DocumentUtils.GeneratePDF() → internally calls ForgePDFLib.CreatePDF()
  If Forge component changes, only DocumentUtils needs updating.

Doc: https://success.outsystems.com/documentation/outsystems_developer_cloud/forge/
