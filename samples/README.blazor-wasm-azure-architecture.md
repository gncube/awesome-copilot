# Sample: Blazor WASM + Azure Functions Architecture

**File**: `blazor-wasm-azure-architecture.puml`

A production-ready C4 Container diagram demonstrating best practices for the **PlantUML Architect** agent, featuring a modern web application architecture with Blazor WebAssembly frontend and serverless Azure Functions backend.

---

## Architecture Overview

### What This Diagram Shows

A three-tier, cloud-native application stack:

1. **Frontend Tier** – Blazor WASM app (single-page app) deployed to Azure Static Web Apps + CDN
2. **API Tier** – Microservices implemented as Azure Functions (HTTP-triggered and event-triggered)
3. **Data & Security Tier** – SQL Database, Key Vault, Service Bus for async operations
4. **Observability** – Application Insights for monitoring and telemetry

### Real-World Use Case

This architecture is ideal for:
- ✅ Enterprise web applications with real-time data binding
- ✅ SaaS platforms requiring global CDN distribution
- ✅ Microservices with async task processing
- ✅ Organizations adopting serverless on Azure

---

## Boundary Integrity Analysis

### ✅ Boundary 1: Frontend Tier (Container_Boundary)

```
Container_Boundary(frontend_boundary, "Frontend Tier - Azure CDN/Static Web Apps") {
    Container(blazor_app, "Blazor WASM App", ...)
    Container(cdn, "Azure CDN", ...)
}
```

**Validation:**
- ✅ Contains 2 substantive elements (Blazor App + CDN)
- ✅ Both are real Container definitions (not just Rel calls)
- ✅ Boundary name reflects content ("Frontend Tier")
- ✅ Related elements grouped together (both frontend delivery)

**No aliases-only boundaries** ❌ Example (INVALID):
```plantuml
Container_Boundary(frontend, "Frontend") {
    Rel(user, someElement, "calls")  # Only a relationship, no definition!
}
```

---

### ✅ Boundary 2: API Tier (Container_Boundary)

```
Container_Boundary(api_boundary, "API Tier - Azure Functions") {
    Container(api_gateway, "API Gateway Function", ...)
    Container(auth_func, "Authentication Function", ...)
    Container(business_logic, "Business Logic Function", ...)
    Container(async_processor, "Async Processor Function", ...)
    
    Rel(api_gateway, auth_func, "delegates auth", ...)
    Rel(api_gateway, business_logic, "routes requests", ...)
}
```

**Validation:**
- ✅ Contains 4 substantive Container elements
- ✅ Internal relationships documented with specific verbs ("delegates", "routes", "publishes")
- ✅ Mixed trigger types (HTTP and Service Bus) – architectural variety within tier
- ✅ Boundary name is descriptive ("API Tier - Azure Functions")

**Boundary Strategy:**
All API services grouped together because:
1. Deployed as single unit (Azure Functions app)
2. Share infrastructure (Managed Identity, Key Vault)
3. Serve single responsibility (API surface)

---

### ✅ Boundary 3: Data & Security Tier (Container_Boundary)

```
Container_Boundary(data_boundary, "Data & Security Tier") {
    Container(secrets_vault, "Azure Key Vault", ...)
    ContainerDb(sql_db, "Azure SQL Database", ...)
    Container(message_queue, "Service Bus", ...)
}
```

**Validation:**
- ✅ Contains 3 elements: 2 Container + 1 ContainerDb
- ✅ All define substantive data/security resources
- ✅ NOT just storage; includes security and messaging
- ✅ Boundary name reflects tier (Data & Security)

**Architectural Rationale:**
Grouped together because:
1. Restricted access (Managed Identity authentication)
2. High-security zone
3. Shared by API layer services

---

### ✅ Boundary 4: Observability (Container_Boundary)

```
Container_Boundary(observability_boundary, "Observability & Monitoring") {
    Container(app_insights, "Application Insights", ...)
}
```

**Validation:**
- ✅ Contains 1 substantive element (App Insights)
- ✅ Single-element boundaries are valid if they serve clear purpose
- ✅ Boundary name reflects concern ("Observability & Monitoring")
- ✅ Not grouped with data tier (different security/access model)

---

## Branding & Tag Application

### GPW Enterprise Branding

All primary elements tagged with `$tags="gpw"` or `$tags="gpw-external"`:

```plantuml
AddElementTag("gpw", $bgColor="#4F46E5", $fontColor="#FFFFFF", $borderColor="#3730A3")

Container(api_gateway, "API Gateway Function", ..., $tags="azure-func+prod+managed-identity")
```

**Visual Result:** Consistent purple branding across all diagrams

### Azure Service Tagging

Each Azure service tagged with specific Azure tag:

```plantuml
AddContainerTag("azure-func", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurefunctions")
AddContainerTag("azure-sql", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azuresqldatabase")
```

**Benefits:**
- ✅ Consistent Azure blue color scheme
- ✅ Sprites render service icons
- ✅ Legend displays all Azure services used
- ✅ Easy to identify services in multi-diagram sets

### Environment Tagging

Production elements tagged with `$tags="prod"`:

```plantuml
Container(blazor_app, "Blazor WASM App", ..., $tags="azure-staticweb+prod")
Container(auth_func, "Authentication Function", ..., $tags="azure-func+prod+managed-identity")
```

**Result in Legend:**
```
┌─────────────────────────────┐
│ Production               🔴  │
│ Azure Function           ⚡  │
│ SQL Database             💾  │
│ Managed Identity         🔐  │
└─────────────────────────────┘
```

---

## Relationship Quality Analysis

### ✅ Specific, Meaningful Verbs

This diagram uses **specific verb phrases**, not generic labels:

| ✅ Good | ❌ Bad |
|--------|--------|
| `"calls API"` | `"uses"` |
| `"delegates auth"` | `"talks to"` |
| `"routes requests"` | `"communicates with"` |
| `"publishes to"` | `"sends data to"` |
| `"queries/updates data"` | `"accesses"` |
| `"consumes messages"` | `"receives from"` |
| `"retrieves secrets"` | `"gets"` |

### Protocol Documentation

Each relationship specifies protocol/technology:

```plantuml
Rel(blazor_app, api_gateway, "calls API", "HTTPS REST")
Rel(business_logic, sql_db, "queries/updates data", "TDS/JDBC")
Rel(async_processor, message_queue, "consumes messages", "AMQP 1.0")
Rel(api_gateway, secrets_vault, "retrieves secrets", "HTTPS + Managed Identity")
```

**Benefits:**
- Developers know exact protocols and connection strings
- Architects understand integration patterns
- Security teams can validate authentication methods
- Operations can configure firewall rules

---

## Cross-Boundary Integration

### Frontend ↔ API Communication

```plantuml
Rel(user, blazor_app, "navigates to", "HTTPS")
Rel(blazor_app, api_gateway, "calls API", "HTTPS REST")
```

**Pattern:** Browser → Static Web → API Gateway

### API ↔ Data Communication

```plantuml
Rel(business_logic, sql_db, "queries/updates data", "TDS/JDBC")
Rel(api_gateway, secrets_vault, "retrieves secrets", "HTTPS + Managed Identity")
```

**Pattern:** Functions → Database (transactional) + Key Vault (auth)

### Async Processing

```plantuml
Rel(business_logic, async_processor, "publishes to", "AMQP")
Rel(async_processor, message_queue, "consumes messages", "AMQP 1.0")
Rel(async_processor, sql_db, "writes results", "TDS/JDBC")
```

**Pattern:** Business logic → Service Bus queue → Async function → Database

### Observability

```plantuml
Rel(api_gateway, app_insights, "logs requests/responses", "HTTPS")
Rel(business_logic, app_insights, "logs telemetry", "HTTPS")
Rel(async_processor, app_insights, "logs events", "HTTPS")
```

**Pattern:** All services report to central telemetry hub

---

## Best Practices Demonstrated

### 1. **Container_Boundary-First Design**
- ✅ Boundaries defined based on logical/architectural concerns (not just visual grouping)
- ✅ Every boundary contains ≥1 substantive element
- ✅ Related services grouped together

### 2. **Security & Identity**
- ✅ Managed Identity tagged consistently (`$tags="managed-identity"`)
- ✅ Key Vault access documented explicitly
- ✅ External identity provider shown (Entra ID)
- ✅ Admin access separately modeled

### 3. **Async Patterns**
- ✅ Service Bus queue shown as messaging intermediary
- ✅ Function trigger types varied (HTTP vs. trigger-based)
- ✅ Long-running operations isolated

### 4. **Observability**
- ✅ Application Insights as dedicated boundary
- ✅ Telemetry relationships explicit
- ✅ Admin monitoring role modeled

### 5. **Azure Service Integration**
- ✅ All services tagged with Azure-specific sprites
- ✅ Managed service advantages highlighted
- ✅ Serverless model emphasized

---

## Validation Against Hard Rules

### ✅ HARD RULE: Boundary Integrity

**Every boundary contains element definitions:**
- Frontend: Blazor App + CDN ✅
- API: 4 Functions ✅
- Data: Key Vault + SQL + Service Bus ✅
- Observability: App Insights ✅

**No aliases-only boundaries:** ✅ All boundaries have real definitions

---

## How to Use This Sample

### 1. **Template for Similar Projects**
Copy structure for other Blazor + Functions projects:
```
Frontend Tier → API Tier → Data Tier → Observability
```

### 2. **Validation Reference**
Use as checklist for your own diagrams:
- [ ] Every boundary has elements?
- [ ] Relationships have specific verbs?
- [ ] Branding applied consistently?
- [ ] Protocols documented?

### 3. **Architecture Review**
Present to stakeholders:
- Executives: Understand cloud consumption
- Architects: Discuss design decisions
- Developers: Implement against spec
- Operations: Plan infrastructure

### 4. **Documentation**
Commit alongside architecture decision records (ADRs):
```
docs/adr/0001-blazor-wasm-azure.md  (decision)
samples/blazor-wasm-azure-architecture.puml  (visual)
```

---

## Extending This Diagram

To add more elements, follow this pattern:

### Add CI/CD Tier
```plantuml
Container_Boundary(cicd_boundary, "CI/CD & Deployment") {
    Container(github_actions, "GitHub Actions", "Workflow Automation", ...)
    Container(acr, "Azure Container Registry", "Image Registry", ...)
}
```

### Add Additional Services
```plantuml
Container(email_func, "Email Service Function", "Azure Function", ..., $tags="azure-func+prod")
Rel(async_processor, email_func, "triggers email", "AMQP")
```

### Add Performance Optimization
```plantuml
Container(cache, "Azure Cache for Redis", "Caching Layer", ..., $tags="azure-cache+prod")
Rel(api_gateway, cache, "queries cache first", "Redis Protocol")
```

---

## Quick Reference: Key Patterns

| Pattern | When to Use |
|---------|------------|
| Container_Boundary | Logical grouping of related services |
| System_Ext | External third-party systems |
| Rel() | Synchronous or documented communication |
| Service Bus queue | Async, decoupled communication |
| Managed Identity | Authentication without secrets in code |
| CDN + Static Web | Global distribution of frontend assets |

---

## Legend Reference

This diagram generates a legend with:
- **Production** (prod tag) – red
- **Azure Function** (azure-func tag) – light blue
- **SQL Database** (azure-sql tag) – light blue
- **Key Vault** (azure-keyvault tag) – light blue
- **Managed Identity** – purple

---

**Want to generate similar diagrams?** Use the PlantUML Architect agent with this prompt:

```
@plantuml-architect
Create a C4 Container diagram for a web application using Blazor WASM frontend and Azure Functions API.

Use Container_Boundary-first strategy:
1. Frontend Tier: Blazor WASM + CDN
2. API Tier: API Gateway, Auth, Business Logic, Async Processor functions
3. Data & Security: Key Vault, SQL Database, Service Bus
4. Observability: Application Insights

Apply GPW branding (enterprise purple) with prod tags. 
Include managed identity, security zones, and specific verb labels.
```

For additional examples and templates, see the **plantuml-c4-branding** skill.
