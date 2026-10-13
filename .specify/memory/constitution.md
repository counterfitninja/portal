# Developer & Team Productivity Portal — Project Constitution
**Version:** 1.0.0
**Scope:** Developer Portal, Internal CMDB, Monitoring Aggregator, & Azure Infrastructure

---

## 1. Foundational Architecture & Azure Standards
1. **Cloud-Native Azure Native Infrastructure:**
   - All platform resources MUST be provisioned using Infrastructure as Code (Bicep or Terraform). Manual Azure Portal configurations are strictly forbidden.
   - Core runtime defaults to **Azure Container Apps (ACA)** for containerized microservices/backends, backed by **Azure Container Registry (ACR)**.
   - Static web assets MUST be hosted on **Azure Static Web Apps** or **Azure App Service**.
   - Database persistence MUST use **Azure Database for PostgreSQL (Flexible Server)** or **Azure Cosmos DB** with managed identities enabled.

2. **Zero-Trust Identity & Access Control:**
   - Security relies on **Azure Active Directory / Microsoft Entra ID** via OpenID Connect (OIDC) / OAuth 2.0.
   - Service-to-service and service-to-Azure authentication MUST use **Azure Managed Identities** (System or User-Assigned). Hardcoded credentials, secrets, or connection strings in source code or environment variables are forbidden.
   - All secret management (API keys for third-party integrations) MUST be stored in **Azure Key Vault** and injected via CSI secret store or runtime key vault references.

---

## 2. Platform Core & CMDB Principles
1. **Single Pane of Glass (Aggregation over Replication):**
   - The portal serves as a unified abstraction layer for Product, Engineering, and SRE teams.
   - Telemetry, work items, and infrastructure state MUST be queried via live integration APIs or asynchronously cached via event buses (Azure Service Bus / Event Grid). The portal MUST NOT become the primary write store for third-party data.

2. **Internal CMDB Design & Schema Rules:**
   - The CMDB is the **Source of Truth** for Service Ownership, Team Mappings, Environment Relationships, and Dependency Graphs.
   - Every service registered in the CMDB MUST have a defined **Owner (ADO Team / Entra Group)**, **Criticality Tier (Tier 1–4)**, **Repository Link**, and **SLO targets**.
   - Relationship graphs (Service A -> Depends On -> Database B) MUST be queryable in real-time.

---

## 3. Integration Governance & Abstraction
1. **Azure DevOps (ADO) Integration:**
   - Sync work items, pull requests, sprint velocity, and CI/CD pipeline run status.
   - Access MUST use ADO OAuth / Service Principals with least-privilege Personal Access Tokens (PATs) stored in Azure Key Vault as a fallback.

2. **New Relic Integration:**
   - Aggregate APM metrics, active incident alerts, Golden Signals (Latency, Traffic, Errors, Saturation), and deployment markers.
   - API interactions MUST use REST/GraphQL endpoints with query-scoped API keys cached securely.

3. **Azure SRE & Azure Monitor Integration:**
   - Pull direct health states from **Azure Resource Health**, **Azure Monitor Alerts**, and **Log Analytics Workspace**.
   - Native integration with **Azure Site Reliability Engineering (SRE)** toolsets (e.g., Chaos Studio, Azure Advisor recommendations, and Workbooks).

4. **Extensibility & Third-Party Adapters:**
   - All third-party integrations MUST implement a standardized `IntegrationAdapter` interface (defining `healthCheck()`, `fetchMetrics()`, `syncEntities()`).
   - Third-party API failures MUST be isolated using circuit breakers to ensure a single failing vendor API does not degrade the core Developer Portal.

---

## 4. Operational, Security, & SRE Standards
1. **Observability & Health Monitoring:**
   - The Developer Portal itself MUST export metrics and traces directly to **Azure Application Insights** and **New Relic**.
   - Standard `/health/liveness` and `/health/readiness` endpoints are mandatory for every portal microservice.

2. **Data Residency & Auditability:**
   - Audit logs for all CMDB mutations, user role changes, and administrative actions MUST be streamed directly to an **Azure Log Analytics Workspace** with a minimum retention period of 90 days.

---

## 5. Development & Code Quality Guidelines
1. **API First Strategy:**
   - All frontend capabilities MUST interact with backend services via documented OpenAPI 3.0 (Swagger) REST endpoints or GraphQL.
2. **Resilience & Caching:**
   - High-volume external metrics (New Relic / Azure Monitor) MUST pass through an **Azure Cache for Redis** layer to adhere to API rate limits and deliver sub-second dashboard load times.