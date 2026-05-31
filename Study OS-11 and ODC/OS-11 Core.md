# O11 Core Reference

## Table of Contents
1. Architecture & 4-Layer Canvas
2. Actions: Client, Server, Service
3. Logic Flow Best Practices
4. REST API Integration
5. SOAP Integration (critical)
6. Advanced SQL & Injection Prevention
7. Roles & User Management
8. Service Center & Debugging
9. Forge & Module Management
10. Timers & Site Properties

---

## 1. Architecture & 4-Layer Canvas

### What it is
O11 uses a 4-layer architecture to enforce separation of concerns and prevent circular dependencies.

### The 4 Layers (bottom to top)
```
[Foundation] - Non-functional utilities (logging, theming, error handlers)
[Core]        - Business entities, business logic, no UI
[End-User]    - Screens and UI, consumes Core
[Orchestration] - Cross-app processes, dashboards, admin views
```

Rules:
- Modules only reference modules in LOWER or SAME layer
- NEVER reference upward (Core referencing End-User = violation)
- Foundation modules have NO dependencies on other custom modules

### Practical example (SmartServe)
```
SmartServeFoundation (Foundation) - Logging utility, custom error messages
SmartServeCore (Core)             - Ticket entity, TicketService, GetTicketById
SmartServeWeb (End-User)          - Screens for users to view/edit tickets
SmartServeAdmin (Orchestration)   - Admin dashboard across ticket + user modules
```

### Common violations and fixes
- Core referencing End-User: Extract shared logic to Foundation
- Circular dependency: Use Service Actions to break the cycle
- Tool: Architecture Dashboard in LifeTime shows violations

Doc: https://success.outsystems.com/documentation/11/app_architecture/
Doc: https://success.outsystems.com/documentation/11/app_architecture/the_architecture_canvas/

Q&A:
1. If a Core module needs a Foundation template, which layer does email logic live in?
2. Can an End-User module reference another End-User module? What is the risk?
3. What tool detects architecture violations in O11?

---

## 2. Actions: Client, Server, Service

### Client Actions
- Run in the BROWSER (JavaScript runtime)
- Access: local variables, screen widgets, client-side logic
- Cannot access: database, server resources directly
- Use for: UI validation, local state changes, calling server actions

Example: ValidateTicketForm (Client Action)
- Check if Title is empty → show inline error message
- Check if Priority is set → show warning
- If valid → call SaveTicket (Server Action)

### Server Actions
- Run on the SERVER
- Access: database, entities, integrations, platform APIs
- Tightly coupled — same transaction as the caller
- Use for: CRUD operations, business logic, calling external APIs

Example: SaveTicket (Server Action)
  Input: TicketRecord (Ticket entity type)
  Logic:
    1. If TicketRecord.Id = NullIdentifier() → CreateTicket
    2. Else → UpdateTicket
    3. Return TicketId

### Service Actions
- Run on SERVER but in their OWN transaction
- Loosely coupled — exposed via internal service catalog
- Used for: cross-module/cross-app reusable logic
- Consumer does not need a direct module reference

Example:
  TicketService module exposes:
    GetTicketById (Service Action) → consumed by Web and Admin modules
    CloseTicket (Service Action) → consumed by integration layer

### Best Practices
- Always use Server Actions for data operations — never DB logic in Client Actions
- Use Service Actions when logic is reused across multiple modules
- Keep Server Actions focused — one responsibility per action
- Name clearly: verb + noun (GetTicket, SaveUser, ValidateForm)

Doc: https://success.outsystems.com/documentation/11/logic/action_flow/
Doc: https://success.outsystems.com/Documentation/11/Developing_an_Application/Reuse_and_Refactor/Use_Services_to_Expose_Functionality

Q&A:
1. User clicks Save on a form. Where does validation run vs DB write?
2. Why use a Service Action instead of a Server Action?
3. Can a Client Action directly query the database?

---

## 3. Logic Flow Best Practices

### Decision nodes (If)
- Always handle BOTH True and False branches
- Use meaningful condition names: IsNewTicket, HasValidEmail

### Exception handling in flows
```
Server Action flow:
  Try:
    → CreateOrUpdateTicket
    → If error → Exception Handler
  Exception Handler:
    → LogMessage (custom logger)
    → Raise UserException with friendly message
    → OR return error flag to caller
```

### Common mistakes
- Forgetting to handle the False branch → runtime flow error
- Mixing UI logic into Server Actions — keep them pure business logic
- Not using transactions correctly — all DB ops in one Server Action share a transaction

### Naming conventions
Good: GetActiveTicketsByUser, ValidateTicketPriority, NotifyAssignee
Bad: Action1, DoStuff, MyFlow

Doc: https://success.outsystems.com/documentation/11/logic/action_flow/handling_exceptions/

---

## 4. REST API Integration (Consuming)

### How to consume a REST API in O11
Step 1: Module → Logic → Integrations → REST → Add REST API
Step 2: Paste base URL, set authentication
Step 3: Add Methods — each method = one endpoint
Step 4: Map input/output parameters to OutSystems structures

### Authentication patterns
- API Key: Add as header in OnBeforeRequest handler
- Bearer Token: Use OnBeforeRequest to add Authorization: Bearer {token}
- OAuth 2.0: Use Forge OAuth2 connector or custom implementation

### Error handling for REST
```
Server Action: CallExternalAPI
  Try:
    → GetExternalTicket(ticketId)
    → If StatusCode = 200 → process
    → If StatusCode = 404 → raise "Ticket not found"
    → If StatusCode = 500 → raise "External system error"
  Exception AllExceptions:
    → Log error details
    → Return IsSuccess=False, ErrorMessage
```

### Common REST errors
| Error | Cause | Fix |
|-------|-------|-----|
| SSL certificate error | Invalid cert | Add cert to trusted store |
| Timeout | Slow service | Set timeout in method properties |
| Serialization error | Structure mismatch | Check JSON field names match exactly |
| 401 Unauthorized | Wrong auth token | Verify token format and header name |

Doc: https://success.outsystems.com/documentation/11/integration_with_external_systems/rest/consume_rest_apis/

---

## 5. SOAP Integration (CRITICAL)

### What SOAP is
SOAP = XML-based web service protocol. Common in enterprise/banking/government.
O11 has strong native SOAP support via WSDL import.

### How to consume SOAP in O11
Step 1: Module → Logic → Integrations → SOAP → Consume SOAP Web Service
Step 2: Provide WSDL URL or upload WSDL file
Step 3: OutSystems auto-generates Structures and Actions for each operation
Step 4: Call generated actions like any Server Action

### Practical example
```
WSDL: https://bankingapi.example.com/AccountService?wsdl

Auto-generated:
  Structure: AccountDetails (AccountNumber, Balance, HolderName)
  Action: GetAccountDetails(AccountNumber) → AccountDetails
  Action: TransferFunds(TransactionRequest) → TransactionResult

Usage in Server Action: ProcessBankTransfer
  → Call TransferFunds(TransactionRequest)
  → If result.Success → update local DB
  → Else → raise exception with result.ErrorCode
```

### SOAP authentication
- Basic auth: Set Username/Password in SOAP properties
- WS-Security: Use custom header via OnBeforeRequest
- Certificate-based: Configure in Service Center → Administration → Certificates

### Interview spoken answer
"SOAP is contract-driven via WSDL, strongly typed, and uses XML. It is common in enterprise banking and legacy systems. REST is lightweight, JSON-based, and more flexible. In OutSystems, SOAP is consumed by importing the WSDL which auto-generates strongly typed actions — much faster than hand-coding. I use SOAP when the external system mandates it, REST for modern APIs."

### Common SOAP errors
| Error | Cause | Fix |
|-------|-------|-----|
| WSDL import fails | WSDL behind firewall | Download and import WSDL file manually |
| SOAPFaultException | Server-side error | Catch exception, log fault code and message |
| Element not recognized | WSDL version mismatch | Re-import WSDL after vendor update |
| Timeout | Slow SOAP endpoint | Increase timeout in method properties |
| Certificate error | SSL/TLS mismatch | Import server cert into O11 trusted certs |

Doc: https://success.outsystems.com/documentation/11/integration_with_external_systems/soap/

---

## 6. Advanced SQL & SQL Injection Prevention

### Aggregates vs Advanced SQL
Aggregates: visual, auto-parameterized, ZERO injection risk — use by default
Advanced SQL: raw SQL, required for UNION, subqueries, stored procs — injection risk zone

### SAFE pattern
```sql
SELECT {Ticket}.[Id], {Ticket}.[Title]
FROM {Ticket}
WHERE {Ticket}.[Status] = @StatusFilter
-- @StatusFilter is an Input Parameter → automatically escaped
```

### UNSAFE pattern (NEVER do this)
```sql
-- DANGEROUS: string concatenation
"SELECT * FROM {Ticket} WHERE Title = '" + UserInput + "'"
-- Attacker inputs: ' OR '1'='1 → returns all records
-- Attacker inputs: '; DROP TABLE Ticket; -- → destroys data
```

### EncodeSql() for dynamic parts
```
"SELECT * FROM {Ticket} WHERE Title LIKE '%" + EncodeSql(SearchText) + "%'"
-- EncodeSql escapes single quotes and special characters
```

### Best practices
1. Always use Input Parameters for user-supplied values
2. Use EncodeSql() if you must concatenate dynamic text
3. Use {Entity} syntax for table names — never hardcode
4. Test with: ' OR 1=1, '; DROP TABLE, <script>
5. Limit DB user permissions — principle of least privilege

### Common SQL errors
| Error | Cause | Fix |
|-------|-------|-----|
| Invalid column name | Attribute renamed, SQL not updated | Update SQL to match new name |
| Open DataReader error | Nested DB calls | Use separate actions |
| 0 rows unexpectedly | Wrong filter or NullIdentifier | Debug with Service Center SQL log |
| String truncated | Data longer than field | Increase attribute length |

Doc: https://success.outsystems.com/documentation/11/logic/sql/
Doc: https://success.outsystems.com/documentation/11/security/application_security/preventing_sql_injection/

Q&A:
1. A developer writes: "WHERE Name = '" + UserInput + "'" — what is the risk and fix?
2. When would you use Advanced SQL over an Aggregate?
3. What does EncodeSql() do?

---

## 7. Roles & User Management

### System Roles (built-in)
- Anonymous — not logged in
- Registered — logged in (any user)
- Manager — built-in admin role

### Custom Roles
Created in: module → Security tab → Roles
Examples: TicketAgent, Supervisor, AdminUser

### Assigning roles
```
Server Action: SetupUserRole
  Input: UserId
  Logic:
    If User.UserType = "Agent" → GrantTicketAgentRole(UserId)
    If User.UserType = "Supervisor" → GrantSupervisorRole(UserId)
```

### Protecting screens
Screen properties → Roles → check required roles
- Anonymous user → redirected to login
- User without role → redirected to NoPermission screen

### Checking roles in logic
```
If CheckTicketAgentRole(GetUserId()) Then → Allow edit
Else → Show read-only view
```

### Admin practices
- Never assign all roles during development — test with real role setup
- Use Groups in Service Center for bulk role assignment
- Audit: Service Center → Users → User detail
- Dedicated Admin role for Service Center — never use built-in admin for app logic

### Role-based data filtering
```
Aggregate filter:
  CheckSupervisorRole(GetUserId()) OR Ticket.AssignedTo = GetUserId()
```

Doc: https://success.outsystems.com/documentation/11/security/user_roles/
Doc: https://success.outsystems.com/documentation/11/managing_the_applications_lifecycle/manage_users/

Q&A:
1. How do you prevent a screen from being accessible to unauthenticated users?
2. What is the difference between CheckRole() and GrantRole()?
3. Where in Service Center do you manage user role assignments at runtime?

---

## 8. Service Center & Debugging

### Key Service Center areas
```
Factory    → All apps and modules, publish history
Monitoring → Error, General, Integration, Extension logs
Administration → DB connections, email, certificates, licensing
Users      → User management, role assignment
```

### Reading error logs
```
Monitoring → Error Log:
  Message: short error description
  Detail: full stack trace + action name + line number
  Module: which eSpace threw the error
  Integration Log: REST/SOAP call details (request/response body)
```

### Debugging in Service Studio
```
1. Set breakpoint on suspect node
2. Run Debug mode (F5)
3. Step through: F10 (step over), F11 (step into), F6 (continue)
4. Watch panel: inspect variable values in real time
```

### Debug best practices
- Always debug in Development — never Production
- Use LogMessage() to write custom entries to General Log
- Use Integration Log to see exact HTTP request/response
- Use RequestKey to correlate all logs for one user request
- Timer failures → check Cyclic Job log in Service Center

Doc: https://success.outsystems.com/documentation/11/managing_the_applications_lifecycle/monitor_and_troubleshoot/
Doc: https://success.outsystems.com/documentation/11/debugging_apps/

---

## 9. Forge & Module Management

### Installing Forge components
```
Step 1: forge.outsystems.com → find component
Step 2: Install → installs via LifeTime
Step 3: Service Studio → Manage Dependencies → find component
Step 4: Add needed elements, publish module
```

### Best practices
DO:
- Wrap Forge actions in your own Service Actions (abstraction layer)
  Example: Create DocumentService.GenerateTicketPDF() that internally calls PDFGenerator
  If you swap Forge component, only one place to change
- Prefer OutSystems Supported components for critical functionality
- Check license before using in commercial apps

DON'T:
- Reference Forge components directly from multiple modules
- Use untrusted Community components in production without code review
- Ignore version updates

### Updating Forge components
Service Center → Factory → Solutions → [component] → Check for updates
After update → republish all dependent modules

### Common Forge issues
| Issue | Cause | Fix |
|-------|-------|-----|
| Missing dependency | Component not in target env | Install in each environment (Dev/QA/Prod) |
| Behaves differently | Version mismatch | Ensure same version across environments |
| Breaking change after update | Forge API changed | Wrap calls in service layer |

Doc: https://success.outsystems.com/support/forge_components/forge_components_best_practices/

---

## 10. Timers & Site Properties

### Timers
Timers = scheduled background jobs running Server Actions.

Example: CleanupExpiredTickets
  Schedule: Daily at 2:00 AM
  Action: CleanExpiredTicketsJob
    → Delete Tickets where ExpiryDate < Today
    → Log count of deleted records

Setup: Module → Processes tab → Timers → Add Timer

Timer errors → Service Center → Monitoring → Cyclic Jobs

### Site Properties
Configuration values changeable per environment without republishing.

Examples:
  MaxTicketAttachmentSizeMB = 10
  SupportEmailAddress = "support@company.com"
  EnableDebugMode = False

Usage:
  If FileSize > GetMaxTicketAttachmentSizeMB() * 1024 * 1024 Then
    Raise "File too large"

Change without republish:
  Service Center → Factory → Modules → [module] → Site Properties → Edit value

Best practices:
- Use Site Properties for anything environment-specific (URLs, limits, flags)
- Never hardcode email addresses or thresholds
