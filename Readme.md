# Low-Level Design Document – Platform-Agnostic Solution Backend

## 1. Introduction

### 1.1 Purpose
This document provides a detailed low-level design for the Platform-Agnostic Solution Backend. It outlines the system architecture, components, interfaces, and implementation details for developers and technical stakeholders.

### 1.2 Scope
The backend supports AWS cloud resource management, including:

- Inventory Management
- Utilization Monitoring
- Cost Analysis
- Rightsizing
- Compliance Scanning
- Provisioning
- User Management
- Organization Management
- Audit Trails
- Reporting

### 1.3 Definitions and Acronyms

| Term | Meaning |
|------|---------|
| EC2 | Elastic Compute Cloud |
| RDS | Relational Database Service |
| ELB | Elastic Load Balancer |
| AMD | Advanced Micro Devices |
| API | Application Programming Interface |
| LLD | Low-Level Design |
| JWT | JSON Web Token |
| FinOps | Financial Operations |
| SecOps | Security Operations |

---

## 2. System Architecture

### 2.1 High-Level Architecture

REST API Layer (Controllers)
└── Feature Services (Inventory, Cost, Utilization, Rightsizing, etc.)
└── Shared Services (Security, AWS Integration, Config, Common Utils)
└── Data Access Layer (MongoDB Repositories)


### 2.2 Technology Stack

| Area | Technology |
|------|------------|
| Framework | Spring Boot |
| Database | MongoDB |
| Cloud SDK | AWS SDK |
| Auth | JWT Token Based |
| Build | Maven |
| Containers | Docker, Docker Compose |
| Logging | Logback |
| Monitoring | Prometheus, Loki, Tempo |
| Secrets | HashiCorp Vault |

### 2.3 Deployment Architecture

Docker Compose Environment
├── App Service (Spring Boot)
├── MongoDB (Data Store)
├── Vault (Secrets)
└── Monitoring Stack (Prometheus, Loki, Tempo)


---

## 3. Detailed Component Design

### 3.1 Core Modules

| Module | Description | Key Components |
|--------|-------------|----------------|
| Inventory | Manages cloud resource inventory | InventoryService, InventoryDetailService |
| Utilization | Monitors resource utilization | UtilizationController, UtilizationMetricsService |
| Cost | Tracks and analyzes cloud cost | CostService, CostReportGenerator |
| Rightsizing | Optimizes instance sizing | RightsizingService, EC2ResizingUtil |
| Compliance | Compliance scanning | ComplianceService, ComplianceScanner |
| Provisioning | Manages resource provisioning | ProvisioningService, ApprovalWorkflow |
| User | User identity & auth | UserService, AuthenticationService |
| Organization | Org grouping & hierarchy | OrganizationService |
| Client | Client configuration | ClientService, ClientValidator |
| Audit Trail | System activity logs | AuditTrailService |
| Reports | Generates reports | ReportService, ReportGenerator |
| Scheduler | Distributed scheduled tasks | SchedulerService, TaskScheduler |
| Modules | Feature toggle & enablement | ModuleService |

---

## 4. Service Layer Design

### 4.1 Inventory Service

**Key Components**
- InventoryController
- InventoryService
- InventoryRepository
- InventoryMapper

**Methods**
getInventory()
getInventoryDetails(resourceId)
extractEC2Inventory()
extractRDSInventory()
extractS3Inventory()
extractEBSInventory()
extractELBInventory()

### 4.2 Utilization Service

**Methods**
getUtilizationMetrics()
handleEC2Service()
handleRDSService()
getHistoricalMetrics()
aggregateMetrics()

### 4.3 Cost Service

**Methods**
getCostDistributionByFamily()
forecastCost()
getCostByService()
getCostByTagKey()
generateCostReport()

### 4.4 Rightsizing Service

**Methods**
getRecommendations()
processRecommendation()
recommendAmdInstanceType()
processResizingAsync()
calculateSavings()
createBackupSnapshot()
    
---

## 5. API Layer Design (Examples)

### 5.1 Client API
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/clients | Create new client |
| GET | /api/clients | List clients |
| PUT | /api/clients/{id} | Update client |
| DELETE | /api/clients/{id} | Delete client |

### 5.2 User API
| Method | Endpoint |
|--------|----------|
| POST | /api/users |
| POST | /api/auth/login |

---

## 6. AWS Integration Layer

| Component | Purpose |
|----------|---------|
| AwsClientFactory | Creates AWS service clients |
| AwsCredentialsProvider | Manages AWS credentials |
| AwsRequestSigner | Signs AWS API requests |

---

## 7. Data Access Layer

### MongoDB Collections
clients
users
organizations
inventory
utilization_metrics
cost_data
rightsizing_recommendations
compliance_issues
provision_resources
audit_trail
reports
modules
scheduled_tasks

### Sample Document (Client)
```json
{
  "name": "Client A",
  "awsAccountId": "123456789012",
  "enabledModules": ["INVENTORY", "COST_MONITORING"],
  "active": true
}
```
---
## 8. Security Considerations

### 8.1 Authentication & Authorization
- JWT-based authentication for secure access.
- Role-Based Access Control (RBAC) with roles such as:
  - `ADMIN`
  - `USER`
  - `VIEWER`
- Each API request includes a JWT token in the `Authorization` header.
- Tokens have expiration and support refresh flows.
- Access to resources is restricted by client/organization boundaries.

### 8.2 Sensitive Data Handling
- AWS access keys and sensitive configuration values are encrypted using **AES-256**.
- User passwords are stored using **BCrypt hashing**.
- Sensitive values are masked in logs to prevent accidental exposure.
- Secrets are **never stored in plain text**.
- Vault is used to securely fetch and rotate credentials.

### 8.3 API Security Controls
- Input validation is enforced across all API endpoints.
- All traffic must use **HTTPS/TLS**.
- Rate limiting and throttling are configured to prevent abuse.
- CSRF protection is enforced for UI workflows.
- Security headers are applied to prevent browser-based attacks:
  - Content-Security-Policy (CSP)
  - X-Frame-Options
  - X-XSS-Protection
- Protection against:
  - **SQL/NoSQL injection**
  - **Cross-Site Scripting (XSS)**
  - **Cross-Site Request Forgery (CSRF)**
  - **Clickjacking**

### 8.4 Vault Integration
The system integrates with **HashiCorp Vault** for secure secret management.

Stored values include:
- AWS credentials
- Database credentials
- API keys / tokens
- Encryption keys

Vault tokens and credentials are **rotated automatically** based on TTL policies.

---

## 9. Database Design

### 9.1 MongoDB Collections

| Collection | Description |
|-----------|-------------|
| clients | Stores client configurations |
| users | Stores user account information |
| organizations | Stores organization details |
| client_user_mappings | Mapping of users to clients |
| inventory | Stores cloud resource inventory data |
| utilization_metrics | Stores resource utilization metrics |
| cost_data | Stores cost and billing records |
| rightsizing_recommendations | Rightsizing decision logs |
| resizing_amd_data | AMD instance resizing reference mappings |
| compliance_issues | Compliance findings per scan |
| provision_resources | Provisioning workflow states |
| audit_trail | Records system actions for traceability |
| reports | Stores generated report data |
| modules | Module enablement/activation information |
| scheduled_tasks | Cron and scheduled task metadata |

### 9.2 Indexing Strategy

Indexes are created for optimal performance:

```json
clients: { organizationId: 1 }, { name: 1 }
users: { email: 1 }, { organizationId: 1 }
organizations: { name: 1 }
inventory: { clientId: 1, resourceId: 1, resourceType: 1 }
utilization_metrics: { clientId: 1, resourceId: 1, timestamp: -1 }
cost_data: { clientId: 1, date: -1, service: 1 }
rightsizing_recommendations: { clientId: 1, resourceId: 1 }
compliance_issues: { clientId: 1, severity: 1, status: 1 }
provision_resources: { clientId: 1, status: 1, createdAt: -1 }
audit_trail: { clientId: 1, timestamp: -1, userId: 1 }
reports: { clientId: 1, reportType: 1, createdAt: -1 }
modules: { clientId: 1 }
scheduled_tasks: { clientId: 1, taskType: 1, nextRunTime: 1 }
```
---


### 9.3 Sample Documents

**Client Document**
```json
{
  "name": "Client A",
  "organizationId": "64a8f001",
  "awsAccountId": "123456789012",
  "active": true,
  "enabledModules": ["INVENTORY", "COST_MONITORING"],
  "createdAt": "2024-01-01T10:00:00Z"
}
```
---

## User Document
```
{
  "email": "user@example.com",
  "role": "ADMIN",
  "organizationId": "64a8f001",
  "active": true,
  "lastLogin": "2024-01-02T14:44:00Z"
}
```
---
## 10. Performance Considerations
### 10.1 Caching Strategy

Caffeine cache for in-memory caching (local environment).

Redis recommended for distributed multi-node deployments.

Cache TTL configured based on module usage patterns.

Cache invalidation triggered upon data updates.
---

### 10.2 Asynchronous Processing

Used for tasks such as:

Inventory sync

Cost data refresh

Rightsizing evaluation

Compliance scanning

Example Executor Configuration:

```
@EnableAsync
@Bean(name = "taskExecutor")
public Executor taskExecutor() {
  ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
  executor.setCorePoolSize(4);
  executor.setMaxPoolSize(8);
  executor.initialize();
  return executor;
}

```
---
### 10.3 Database Optimization

Extensive indexing for query speed.

Document model optimized to avoid unnecessary nesting.

MongoDB connection pooling enabled by default.

---

### 10.4 AWS SDK Optimization

AWS client connections re-used.

Retry strategy uses exponential backoff.

Throttling errors handled gracefully.
---

## 11. Monitoring and Observability
11.1 Prometheus Metrics

Metrics exposed through:
```
/actuator/prometheus
```
Includes:

Request latency

Resource sync job durations

API call success/failure ratios
---

### 11.2 Loki Log Aggregation

Logs stored in JSON structured format.

Indexed for fast search by:

clientId

userId

requestId

### 11.3 Tempo Distributed Tracing

Tracing enabled for end-to-end transaction visibility.

Helps identify performance bottlenecks and latency issues.

---
## 12. Appendix
12.1 API Documentation

Swagger UI is available at:
```
http://localhost:8080/swagger-ui/index.html

```
---

### 12.2 Developer Setup Requirements

```
Java 11+
Maven 3.6+
Docker & Docker Compose

```
