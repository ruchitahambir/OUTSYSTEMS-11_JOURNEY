# OutSystems Interview Answer Bank
## 20 Most Common Questions — Spoken Answer Format

Instructions for use:
- Read each answer ONCE
- Close this file
- Say the answer OUT LOUD to yourself or your phone camera
- Stumble through it — that's fine
- Say it again. By the 3rd time it's yours.

---

## 1. What is OutSystems and what makes it different from traditional development?

**Spoken answer:**
"OutSystems is a low-code platform that lets you build enterprise-grade web and mobile applications visually. What makes it different is the speed — you're building with a visual IDE, pre-built components, and one-click deployment, so what takes weeks in traditional development takes days. But it's not just drag-and-drop — you still write logic, design data models, and integrate with external systems. The platform handles the infrastructure, security, and scalability concerns, so developers focus entirely on business logic."

---

## 2. What is the difference between O11 and ODC?

**Spoken answer:**
"O11 is the established OutSystems platform — module-based, can run on-prem or cloud, follows a 4-layer canvas architecture. ODC is the next-generation cloud-native version built on Kubernetes with auto-scaling and zero infrastructure management. The biggest architectural shift is that ODC moves from module-based thinking to domain-driven design — each app owns its data and cross-domain communication happens only through service contracts. ODC also adds native feature toggles and breaking change management that O11 needs workarounds for."

---

## 3. What is the 4-layer canvas in O11?

**Spoken answer:**
"The 4-layer canvas is the O11 architecture pattern. At the bottom is the Foundation layer — reusable UI components and themes. Above that is the Core layer — business entities and logic. Then the End-User layer — screens facing specific user groups. And optionally an Orchestration layer for cross-domain processes. The key rule is that references only flow downward. A Core module never references an End-User module. This prevents circular dependencies and keeps the architecture clean and independently deployable."

---

## 4. What is the difference between Client Actions and Server Actions?

**Spoken answer:**
"Client actions run in the browser — they're fast, no server round trip, ideal for UI interactions and local data manipulation. Server actions run on the server — they access the database, call integrations, and handle sensitive business logic. The rule of thumb is: if you're touching data or an external system, it's a server action. If you're responding to user interaction without needing fresh data from the server, it's a client action. Mixing them up is a common mistake that causes either security issues or unnecessary server load."

---

## 5. How do you prevent SQL injection in OutSystems?

**Spoken answer:**
"OutSystems Aggregates are completely safe from SQL injection — they're parameterised under the hood. The risk only appears in Advanced SQL, and there I always use input parameters with the @param syntax rather than concatenating user input into the SQL string. For any dynamic string portions I use the EncodeSql function. The rule is simple: never build SQL by concatenating user-supplied values. If I'm ever tempted to do that, it means I should be using an input parameter instead."

---

## 6. How do you pass data between screens?

**Spoken answer:**
"I use input parameters on the destination screen. On the source screen, I set the destination and map the value — typically an entity ID — to the input parameter. On the destination screen's OnInitialize, I fetch the full record using that ID as a filter in an Aggregate. This keeps the URL clean, the data fresh, and the screen self-sufficient. For temporary UI state that doesn't need to survive navigation, I use local variables or client variables."

---

## 7. What are Roles and how do you implement them?

**Spoken answer:**
"Roles in O11 control what users can access. I define custom roles per business requirement — things like Admin, Manager, or Technician. I grant roles programmatically using Grant Role server actions, usually during login or user registration. I protect screens by setting the required role in screen properties, so OutSystems auto-redirects unauthorised users. For conditional UI — like showing a Delete button only to Admins — I use the Check Role function in expressions. Anonymous and Registered are the built-in roles for unauthenticated and any-authenticated access."

---

## 8. What are Aggregates and when would you use Advanced SQL instead?

**Spoken answer:**
"Aggregates are my default — they're visual, safe from SQL injection, and OutSystems optimises them automatically. I use them for the vast majority of queries. I only drop to Advanced SQL when I need something Aggregates can't express — like a UNION, a complex subquery, or window functions. Even then, I use input parameters and EncodeSql for any user-supplied values. Advanced SQL is a power tool for specific cases, not a replacement for Aggregates."

---

## 9. How do you handle file uploads in OutSystems without Forge?

**Spoken answer:**
"For multiple file uploads without Forge, I maintain a local list variable on the screen — a list of a custom structure holding filename, binary content, and MIME type. Each time the user selects a file, the OnChange action of the Upload widget appends it to the list. On Save, a server action iterates through the list with a For Each and persists each file to the database. I also validate file types and sizes on the client side before even hitting the server."

---

## 10. What is a Timer in OutSystems?

**Spoken answer:**
"Timers are scheduled background jobs that run server-side — similar to cron jobs. I've used them for sending daily digest emails, archiving old records, and syncing data with external systems. They run asynchronously, so they don't affect the user interface at all. You define the schedule in Service Studio and can also trigger them programmatically using Wake actions when you need on-demand background processing. In ODC, the concept is the same but configured through ODC Portal."

---

## 11. How do you consume a REST API in OutSystems?

**Spoken answer:**
"In Service Studio, I go to Logic, then Integrations, then Consume REST API. I can paste an OpenAPI spec or define the methods manually. OutSystems automatically generates the actions and data structures from the spec. I then call the generated action from a server action in my logic — never from client actions, to keep credentials server-side. For authentication I handle headers or query parameters in the OnBeforeRequest callback, which runs before every API call."

---

## 12. What is the screen lifecycle in Reactive Web?

**Spoken answer:**
"The main lifecycle events I work with are OnInitialize and OnReady. OnInitialize runs before the screen renders — this is where I fetch data, set default variable values, and do any server-side preparation. OnReady runs after the screen is rendered on the client — this is where I'd run any JavaScript, set focus, or initialise third-party widgets. A common mistake is trying to manipulate UI elements in OnInitialize — they don't exist yet at that point."

---

## 13. How do you handle errors and exceptions in OutSystems?

**Spoken answer:**
"I define User Exceptions for specific business rule violations — for example, 'TicketAlreadyClosed' or 'InsufficientBalance'. For database and integration errors I use exception handlers in my action flows. OutSystems automatically rolls back database transactions when an unhandled exception occurs, which I rely on for data integrity. I always log errors using the Log Message action and show user-friendly messages rather than technical details. Critical errors go to a monitoring dashboard via the Error Log entity."

---

## 14. What are Site Properties and when do you use them?

**Spoken answer:**
"Site Properties are environment-specific configuration values — things I don't want to hardcode because they differ between Development, QA, and Production. API base URLs, file size limits, timeout values, feature flags. I centralise them in a Core module so they're referenced from one place. The key benefit is that I can change a value per environment in Service Center without touching or redeploying code. In ODC the equivalent is Settings, which are managed per stage in ODC Portal."

---

## 15. What is Domain-Driven Design in ODC?

**Spoken answer:**
"In ODC, instead of the 4-layer canvas, I organise applications around domains — bounded business contexts that map to areas of the business like Orders, Customers, or Inventory. Each domain owns its data — its apps have private entity schemas that other domains cannot access directly. Cross-domain communication happens only through Service Actions, which act like API contracts between teams. This is stricter than O11's architecture but it scales much better in large multi-team environments because domains are truly independent."

---

## 16. How does deployment differ between O11 and ODC?

**Spoken answer:**
"In O11, you publish to a server and promote through environments — Development, QA, Production — using LifeTime. In ODC, you deploy to stages — Dev, QA, Production — through ODC Portal, and apps run in Kubernetes containers that auto-scale. The key difference is that in ODC there's zero infrastructure management — OutSystems handles all of that. You focus purely on the application. ODC also has native feature toggles so you can deploy code dark and enable it per stage without a new deployment."

---

## 17. What is a Service Action in ODC?

**Spoken answer:**
"A Service Action in ODC is a server action that's explicitly published for consumption by other apps or domains. It's the cross-domain contract. When I expose a Service Action, I'm saying: other teams can call this, and I own this contract. ODC enforces versioning — if I change the signature, consumers get notified and I can run old and new versions simultaneously during migration. This makes Service Actions the backbone of ODC's domain isolation model."

---

## 18. How do you optimise performance in OutSystems?

**Spoken answer:**
"My main performance patterns: first, fetch only what you need — use precise filters and Max Records in Aggregates. Second, avoid N+1 queries — never fetch related data inside a loop, always join in the Aggregate. Third, use client-side local storage for data the user needs repeatedly without server round trips. Fourth, lazy load long lists using the OnScrollEnding event. And fifth, cache configuration in Site Properties rather than querying the database for static config on every request."

---

## 19. How do you structure a portfolio app to show to interviewers?

**Spoken answer:**
"I structure portfolio apps to demonstrate breadth: a proper data model with relationships, screen-to-screen navigation with input parameters, role-based access control, at least one REST API integration, error handling with user-friendly feedback, and a timer or asynchronous process. I document the architecture decisions — why I chose certain patterns. For ODC, I add domain isolation and a Service Action to show I understand cross-domain design. The goal is to show I can build production-quality apps, not just hello-world demos."

---

## 20. How do you explain the career gap in your experience?

**Spoken answer:**
"I used this period as a deliberate transition into low-code development, with a specific focus on OutSystems ODC. I supplemented it with Azure and data fundamentals to become a stronger full-stack low-code developer. I built portfolio applications, studied architecture patterns, and developed a solid foundation in both O11 and ODC. I'm now fully prepared to apply these skills in a production environment and contribute from day one."

---

## Quick drill — say these 5 daily until automatic

1. "Aggregates are always safe; Advanced SQL needs EncodeSql and input parameters — never concatenate user input."
2. "Client actions run in the browser; server actions run on the server. Data operations always go server-side."
3. "References in the 4-layer canvas only flow downward — never upward, never sideways between cores."
4. "ODC replaces modules with apps and libraries, and replaces 4-layer canvas with domain-driven design."
5. "I grant roles programmatically during login, protect screens with role properties, and check roles in logic with Check Role functions."
