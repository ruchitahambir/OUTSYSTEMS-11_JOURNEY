# Interview Q&A Reference

## How to use this file
Every answer here is a SPOKEN answer template.
Read it once. Then close this file and say it out loud. Stumble through it. Say it again.
By the third time it should feel like yours.

---

## Architecture Questions

### Q: Explain the OutSystems 11 architecture canvas.
"OutSystems O11 uses a 4-layer architecture to organize modules and prevent circular dependencies. Bottom layer is Foundation — utilities like logging, no business logic. Above that is Core — business entities and logic, no UI. Then End-User — the screens and interfaces. Top is Orchestration — cross-app dashboards and processes. The rule is simple: you can only reference modules in the same or lower layer. This keeps the codebase maintainable and scalable."

### Q: What is the difference between O11 and ODC?
"ODC is OutSystems' cloud-native next-generation platform. The biggest difference is the architecture — O11 uses a module-based 4-layer canvas, ODC uses domain-driven design where each app behaves more like a microservice. In ODC, Service Actions run in their own transaction — they're essentially internal REST calls. The monitoring tool shifts from Service Center and LifeTime to the ODC Portal. The database changes from SQL Server to Aurora PostgreSQL. And ODC is cloud-only — no self-hosted option."

### Q: How do you handle circular dependencies in O11?
"A circular dependency means Module A references Module B and Module B references Module A — OutSystems won't allow this. The fix is to extract the shared logic into a lower-level module — usually a Core or Foundation module — so both A and B can reference it without referencing each other. I use the Architecture Dashboard in LifeTime to detect these early."

---

## Actions Questions

### Q: What is the difference between Client, Server, and Service Actions?
"Client Actions run in the browser — used for UI logic and validation. They can't directly access the database. Server Actions run on the server — this is where all database operations and business logic live. They're tightly coupled — part of the same transaction as the caller. Service Actions also run on the server but in their own transaction. They're used to expose reusable logic across modules without tight coupling. In ODC, Service Actions are even more like REST calls between microservices."

### Q: When would you use a Service Action over a Server Action?
"I use a Service Action when the logic needs to be reused across multiple modules or applications, and I want to avoid a direct module reference. It gives loose coupling — consumers depend on the service contract, not the internal implementation. The trade-off is that it runs in its own transaction, so I need to handle rollback scenarios explicitly — especially in ODC."

### Q: Can a Client Action query the database?
"No, Client Actions run in the browser and have no direct database access. If I need to fetch data from a Client Action, I call a Server Action from it, which does the database work and returns the data. The Client Action then handles the UI update."

---

## SQL & Security Questions

### Q: How do you prevent SQL injection in OutSystems?
"OutSystems makes this mostly automatic — Aggregates are fully parameterized under the hood so they're safe by default. The risk comes with Advanced SQL queries. There I always use Input Parameters for any user-supplied values — they get automatically escaped. If I absolutely must concatenate dynamic text, I use the EncodeSql() function which escapes single quotes and special characters. I never use string concatenation directly. Basically my rule is: user input never touches SQL strings directly."

### Q: When would you use Advanced SQL instead of an Aggregate?
"Aggregates cover most needs — joins, filters, sorting — and are always safe. I reach for Advanced SQL when I need something an Aggregate can't do: a UNION query, a complex subquery, calling a stored procedure, or a bulk operation. Advanced SQL gives full flexibility but requires extra care on security."

### Q: What is EncodeSql() and when do you use it?
"EncodeSql() is a built-in OutSystems function that escapes special characters in a string to make it safe for use in SQL. Specifically it escapes single quotes and other characters that could be interpreted as SQL syntax. I use it in Advanced SQL when I have a case where I must concatenate dynamic text — like a dynamic LIKE search — but even then I prefer to restructure the query to use Input Parameters if possible."

---

## SOAP & REST Questions

### Q: How do you consume a SOAP web service in OutSystems O11?
"In O11, consuming SOAP is straightforward. I go to the module's Logic tab, find Integrations, and choose Consume SOAP Web Service. I provide the WSDL URL or upload the file. OutSystems automatically reads the WSDL and generates all the data structures and actions for each operation. Then I just call those generated actions in my logic like any other Server Action. If the WSDL is behind a firewall, I download the file and import it manually."

### Q: What is the difference between consuming SOAP in O11 vs ODC?
"In O11, SOAP support is native and built into Service Studio — import WSDL, get actions, done. In ODC, there's no native SOAP in ODC Studio. Instead you use Integration Builder, which is a separate tool. You import the WSDL there, it generates an OutSystems library with all the SOAP actions, and then you add that library as a dependency in your ODC app. One extra step, but the end result is similar."

### Q: How do you handle errors when calling an external REST API?
"I always wrap external API calls in a try-catch pattern. First I check the response status code — 200 means success, 404 means resource not found, 500 means server error. Each case gets its own handling. For unexpected errors I catch AllExceptions, log the full details — action name, input parameters, error message — to the Integration Log or General Log. I never expose raw error messages to users. Instead I raise a friendly UserException like 'External service is temporarily unavailable, please try again.'"

---

## Roles & Security Questions

### Q: How do roles work in OutSystems O11?
"Roles in O11 control access to screens and logic at runtime. You define custom roles in the module's Security tab — things like TicketAgent or Supervisor. At login or registration, you assign roles to users using the built-in GrantRole Server Actions. To protect a screen, you set its Roles property to the required roles — unauthenticated users get redirected to login, unauthorized users to a NoPermission screen. In logic, I use CheckRole() functions to conditionally show or hide features."

### Q: How do you secure data so users only see their own records?
"I combine roles with data-level filtering in aggregates. For example, if someone has a Supervisor role, they see all tickets. If they're a TicketAgent, they only see tickets assigned to them. The aggregate filter would look like: CheckSupervisorRole() OR Ticket.AssignedTo = GetUserId(). This way the security is in the data layer, not just the UI — so even if someone bypasses the UI, they can't get data they shouldn't see."

---

## Service Center & Debugging Questions

### Q: How do you debug a production issue in OutSystems?
"I start with Service Center's Monitoring section. I check the Error Log for the timestamp the issue was reported — it gives me the error message, stack trace, module, and which user was affected. If it's an integration issue, I check the Integration Log to see the exact HTTP request and response. I correlate logs using the RequestKey. Once I identify the failing action and line, I reproduce it in Development with the debugger — setting breakpoints and inspecting variables in the Watch panel. I never debug in Production."

### Q: What is a RequestKey and why does it matter?
"RequestKey is a unique identifier OutSystems assigns to each top-level web request. It gets propagated through all actions, Service Actions, and integration calls within that request. In Service Center's logs, I can filter by RequestKey to see every log entry related to one user's action — across multiple modules. It's essential for tracing a complex flow that touches many modules or external services."

### Q: What logs are available in Service Center and what does each show?
"There are four main logs. Error Log shows runtime exceptions with full stack traces. General Log shows custom entries written with LogMessage() — I use this to log important business events or debug info. Integration Log shows all REST and SOAP calls — the full request and response — which is invaluable for diagnosing API issues. Extension Log shows errors from custom Extensions built with Integration Studio."

---

## ODC-Specific Questions

### Q: What is an Agent in ODC and how does it work?
"ODC Agents are AI-powered automated workflows introduced with Agent Workbench. An agent has instructions — which is basically a system prompt defining its goal — and a set of tools, which are OutSystems Server Actions exposed to the AI. When triggered, the agent uses an LLM to reason about which tool to call next, calls it, processes the result, and continues until the goal is achieved. It's useful for automating complex workflows that would normally require human judgment — like triaging support tickets or orchestrating multi-step approval processes."

### Q: How is ODC Portal different from Service Center?
"ODC Portal replaces both LifeTime and Service Center. It's a single web interface for everything: deploying apps across environments, monitoring logs, managing users and roles, configuring app settings, and accessing ODC Forge. The monitoring is more powerful — ODC Portal shows distributed traces, so you can trace a request across multiple apps in one view, which was harder to do in Service Center which showed per-module logs."

### Q: What should you know about transactions in ODC Service Actions?
"This is one of the most important ODC differences. In O11, Service Actions run in the same transaction as the caller by default — if the caller rolls back, the Service Action's changes roll back too. In ODC, every Service Action runs in its own separate transaction. So if a Service Action commits successfully but the caller fails afterward, that Service Action's data is already committed and will NOT be rolled back. This means I need to implement compensating transactions — if step B fails after step A succeeded, I have to explicitly undo step A in an error handler."

---

## Gap Story (practice saying this out loud)

"I used this period to deliberately transition into low-code development, focusing on OutSystems ODC. I supplemented it with Azure and data fundamentals to be a stronger full-stack low-code developer. I'm now ready to apply that in a production environment."

---

## Quick confidence builders — say these before an interview

"I have hands-on experience building applications in OutSystems O11."
"I understand the architecture canvas and how to prevent circular dependencies."
"I know the difference between Client, Server, and Service Actions and when to use each."
"I can consume SOAP and REST APIs and handle errors properly."
"I understand SQL injection prevention and always use parameterized queries."
"I have worked with roles, user management, and screen-level security."
"I know how to read Service Center logs and debug using Service Studio."
"I am actively learning ODC and understand how it differs from O11."

Say each one out loud three times. Make them feel natural. They are all TRUE.
