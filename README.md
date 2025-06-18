***ATHENA BUDGET***

*Budget Approval System for a Large Multi-national Sales Organization.*

**Description**

Athena Budget is a secure and scalable budget approval platform designed for a large multi-national sales organization with thousands of employees across multiple regions. It centralizes and streamlines budget request and approval workflows within the company, supporting multi-level hierarchical approvals (L1 to L3) and a 24-hour rollback window for flexible decision management.

Sales employees submit budget requests via a React Native mobile app featuring biometric authentication. Approvers also use the mobile app with biometric authentication to manage approvals and rejections. Admin users control budget categories, reasons, and approver assignments through the React web portal.

Built on a microservices architecture, Athena Budget leverages Kafka for event-driven communication and mTLS for secure inter-service authentication. Notifications are sent through push (Firebase Cloud Messaging), email (AWS SES), and SMS (Twilio) channels. The system is containerized with Docker and orchestrated using Kubernetes, providing a robust, modern solution tailored to the complex needs of a global sales enterprise.

**Services**

**User Service**
Responsibilities: Login, register, manage roles, departments.

```
POST   /api/v1/auth/login                 // Login (returns JWT)
POST   /api/v1/auth/logout                // Logout
GET    /api/v1/users/me                   // Get current user info
GET    /api/v1/users/{id}                 // Get user by ID
GET    /api/v1/users?role=manager         // List users filtered by role
POST   /api/v1/users                      // Create user (admin)
PUT    /api/v1/users/{id}                 // Update user
DELETE /api/v1/users/{id}                 // Delete user (admin)
GET    /api/v1/departments                // List all departments
POST   /api/v1/departments                // Create department
```

**Budget Request Service**
Responsibilities: Create and view budget requests (sales user only).
```
POST   /api/v1/requests                   // Create new budget request (locked after submit)
GET    /api/v1/requests                   // List my requests
GET    /api/v1/requests/{id}             // View specific request
GET    /api/v1/requests/{id}/status      // View status history
```

**Approval Service**
Responsibilities: Manage approval steps (L1, L2, L3) with rollback window.
```
GET    /api/v1/approvals/incoming        // Approver: see pending approvals
POST   /api/v1/approvals/{id}/approve    // Approve a budget request
POST   /api/v1/approvals/{id}/reject     // Reject a request
POST   /api/v1/approvals/{id}/rollback   // Rollback approval (within 24h)
GET    /api/v1/approvals/history         // View my approval actions
```

**Admin Portal Service**
Responsibilities: Manage categories, reasons, and assign approvers.
```
GET    /api/v1/categories                // List budget categories
POST   /api/v1/categories                // Add new category
PUT    /api/v1/categories/{id}           // Update category
DELETE /api/v1/categories/{id}           // Delete category

GET    /api/v1/reasons                   // List predefined spending reasons
POST   /api/v1/reasons                   // Add reason
PUT    /api/v1/reasons/{id}              // Edit reason

POST   /api/v1/assignments               // Assign approvers to categories
GET    /api/v1/assignments               // View assignments
```

**Audit / History Service**
Responsibilities: Record every action (immutable log).
```
GET    /api/v1/audit/logs                // Admin view of all action logs
GET    /api/v1/audit/logs?userId=xxx     // Filter by user
GET    /api/v1/audit/logs?requestId=yyy  // Filter by budget request
```

**Notification Service**
Responsibilities: Internal service, consumes Kafka topic.
```
POST   /api/v1/notify/push               // Trigger mobile push (FCM)
POST   /api/v1/notify/email              // Send email
POST   /api/v1/notify/sms                // Send SMS
GET    /api/v1/notify/status/{id}        // Check notification delivery status
Most notification endpoints are internal — they respond to events, not user calls.
```

**Securing API**
Use JWT bearer tokens (OAuth2 or Firebase).

Each microservice validates JWT locally or via shared JWKS.

API Gateway can enforce route-level roles (e.g., role: admin).


**Tech Stack**
```
Layer:	Technology / Tool
Frontend:	React (Web Portal), React Native (Mobile App)
Backend Microservices:	Java Spring Boot, Spring Security (JWT, OAuth2)
API Gateway / BFF:	Spring Cloud Gateway or Custom API Gateway
Communication:	Kafka (Event Bus), REST over HTTPS
Service Security:	mTLS (Mutual TLS) between microservices
Databases:	PostgreSQL (per microservice schema)
Caching & Session:	Redis (optional for caching/session storage)
Notifications:	Firebase Cloud Messaging (Push), AWS SES (Email), Twilio (SMS)
Containerization:	Docker
Orchestration:	Kubernetes
CI/CD:	GitHub Actions / Jenkins / GitLab CI
Cloud (may terminate all services right after project completed):	AWS
Monitoring & Logging:	Prometheus, Grafana, ELK Stack / CloudWatch
```

**Tech Stack Diagram**

```
[React Native App]             [React Web Portal]
        |                              |
        |        HTTPS (JWT Auth)      |
        +------------------------------+
                           |
                      [API Gateway / BFF]
                           |
                           +---------------------------------------------------+
                           |                                                   |
                 [User Service]                                    [Admin Portal Service]
             (Login, Role, Department)                 (Manage Category, Reasons, Assign Approvers)
                           |
      +--------------------+-------------------+-------------------------------+
      |                    |                   |                               |
[Budget Request Service]   |          [Approval Service]               [Audit / History Service]
(Create, View, Lock)       |        (L1–L3 approval, rollback 24h)     (Track all actions, immutable)
                           |
                   Kafka Event Bus
                           |
                           +-------------------------------+
                           |                               |
                [Notification Service]             [Rate Limiter Service] (optional)
     (Push via FCM, Email, SMS notifications)         (Per-user or per-IP API limits)
                           |
         +-----------------+----------------+
         |                                  |
 [Email Gateway Integration]      [SMS Gateway Integration]
 (e.g., SES, Mailgun, SMTP)       (e.g., Twilio, Nexmo)

Internal Communication:
- All microservices use **mTLS**
- Client communication uses **JWT (OAuth2)**

                           +-----------------------------+
                           |                             |
                    [PostgreSQL DBs]               [Redis (cache/session)]
              (One DB per service or shared schema)     (Optional, for token/session caching)
```

**Testing**
- Unit Testing: JUnit + Mockito — essential for every microservice’s core logic.

- Integration Testing: Spring Boot Test + Testcontainers — to test your microservice with DB and dependencies.

- End-to-End Testing: Cypress (for React) + REST-assured (for backend APIs) — to validate critical user flows and APIs.

- (Optional) Contract Testing (Pact or Spring Cloud Contract): Helps avoid integration breaks between microservices.

- Automate unit, integration, and E2E tests in your pipeline (GitHub Actions or Jenkins).

**Devops**

```
[Developer] 
     |
     v
[Git Repository] <-- Push code / PR
     |
     v
[Jenkins Pipeline Trigger]
     |
     v
+----------------------+
|       Build          |
| - Compile backend    |
| - Build frontend     |
| - Build Docker images|
| - Push images to ECR |
+----------------------+
     |
     v
+----------------------+
|      Test            |
| - Unit tests         |
| - Integration tests  |
| - (Optional) Contract|
| - Frontend tests     |
+----------------------+
     |
     v
+----------------------+
| Static Code Analysis  |
| (Optional, SonarQube)|
+----------------------+
     |
     v
+----------------------+
| Deploy to Staging    |
| - Deploy Docker images|
| - Run DB migrations  |
+----------------------+
     |
     v
+----------------------+
| End-to-End Tests     |
| - Cypress (UI)       |
| - REST-assured / API |
+----------------------+
     |
     v
+----------------------+
| Manual / Auto Approval|
+----------------------+
     |
     v
+----------------------+
| Deploy to Production |
| - Deploy Docker images|
| - Run DB migrations  |
+----------------------+
     |
     v
+----------------------+
| Monitoring & Alerts  |
| - Jenkins notify     |
| - CloudWatch, etc.   |
+----------------------+

- Jenkins pulls code from the repository (e.g., GitHub).

- Runs build and automated tests.

- Builds Docker images and pushes them to Amazon Elastic Container Registry (ECR).

- Deploys updated images to ECS with Fargate or EKS cluster.

- Triggers automated integration and deployment tests in staging environment.
```

**AWS**

```
                                +--------------------------+
                                |       React Native App    |
                                |  (Mobile, biometric auth) |
                                +-------------+------------+
                                              |
                                +-------------|------------+
                                |      React Web Portal     |
                                +-------------+------------+
                                              |
                               HTTPS (JWT OAuth2 / OpenID Connect)
                                              |
                                +-------------v------------+
                                |    Elastic Load Balancer  |  <-- ALB with SSL cert from ACM
                                +-------------+------------+
                                              |
                                +-------------v------------+
                                |    Amazon ECS / EKS       |  <-- Backend microservices containerized with Docker
                                |  (API Gateway / BFF +     |
                                |   User, Budget, Approval, |
                                |   Notification, Admin)    |
                                +------+------+-------------+
                                       |      |
         +-----------------------------+      +-----------------------------+
         |                                                            |
+--------v---------+                                        +---------v--------+
| Amazon RDS       |                                        | AWS Secrets      |
| PostgreSQL       |                                        | Manager / SSM    |
| (Per service DB) |                                        | (Store JWT keys, |
+------------------+                                        | DB credentials)  |
                                                            +-----------------+

               |
               +-------------------------------+
               |                               |
       +-------v--------+              +-------v--------+
       | Kafka Cluster  | (Optional)   | Redis Cache    | (Optional)
       +----------------+             +----------------+

               |
+--------------v-----------------+
| Notification System (Push)     |
| Firebase Cloud Messaging (FCM) |
| + AWS SES (Email)              |
| + Twilio (SMS)                 |
+-------------------------------+

              |
+-------------v---------------+
| Monitoring & Logging        |
| Amazon CloudWatch           |
+----------------------------+

              ^
              |
              |
      +-------v---------+
      |     Jenkins     |  <-- CI/CD Pipeline: Build, Test, Dockerize, Push to ECR, Deploy to ECS/EKS
      +-----------------+

```

- Frontend Apps (React Native & React Web) communicate over HTTPS secured with JWT tokens issued by authentication system (can be a service integrated or AWS Cognito or Keycloak externally).

- Incoming client traffic hits the ALB (Application Load Balancer), which routes requests securely (TLS via ACM cert).

ALB forwards traffic to the microservices running in Amazon ECS with Fargate or Amazon EKS cluster for Kubernetes skills demonstration.

- Each microservice uses its own PostgreSQL database managed by Amazon RDS.

- Secrets like DB credentials and JWT private keys are securely stored and retrieved from AWS Secrets Manager or optionally SSM Parameter Store.

- Kafka can be deployed externally or self-managed (or AWS MSK) for event-driven communication, and Redis optionally for caching or session management.

- The Notification Service sends push notifications via Firebase Cloud Messaging (FCM), emails via AWS SES, and SMS via Twilio.

- CloudWatch monitors logs, metrics, and alerts for the whole system.

- CI/CD (not shown in diagram) runs on GitHub Actions pushing Docker images to ECR and deploying to ECS/EKS.

***Post-AWS Deployment: Laptop-Based High Availability Setup with Kubernetes***

After decommissioning AWS services, the system can be deployed locally on a laptop or workstation using Kubernetes, simulating a production-like environment. To enhance availability and reliability, two identical application server instances are run locally, and a Load Balancer distributes traffic between them.

- Kubernetes Cluster:	k3s (recommended for local prod-like setup): Minikube (great for testing and demos), Kind (lightweight CI/testing)
- Container Runtime:	 Docker – required for running containers and building images
- API Gateway:	 NGINX Ingress Controller or Traefik as ingress (mTLS-ready, supports routing, TLS termination)
- Secrets Store:	 Kubernetes Secrets (for JWT keys, DB creds, etc.)
- Optional: HashiCorp Vault for advanced secrets mgmt
- CI/CD:	 Jenkins (locally installed or containerized) – Build → Dockerize → Push to local registry → Deploy
- Monitoring / Logging:	 Prometheus + Grafana + Loki (for logs) – optional but excellent for showcasing
- Local Databases:	 Each microservice gets its own PostgreSQL DB (can run in Docker Compose or k8s StatefulSets)
- Kafka (Event Bus):	 Docker container of bitnami/kafka or confluentinc/cp-kafka
- Redis (Optional):	 Docker container (for caching, token session store, etc.)

***Optiona Note***

Security Risks & Mitigations
Risk | Threat	Mitigation Strategy
--- | ---
Expired JWTs | Short-lived access tokens + long-lived refresh tokens; store refresh tokens securely.
Token Theft / Replay Attacks | Use HTTPS everywhere. Implement JWT token binding to client (e.g., via IP/device fingerprinting).
Insecure Secrets | Store secrets in Kubernetes Secrets or use HashiCorp Vault. Avoid hardcoding secrets in code.
Privilege Escalation | Enforce role-based access control (RBAC) at both API Gateway and service levels.
Insecure Inter-Service Communication | Use mTLS between microservices to ensure both parties are authenticated.
Lack of Auditability | Log all critical actions to an immutable Audit Service (e.g., approvals, deletions).
Denial of Service (DoS) | Implement rate limiting using tools like Istio, NGINX, or API Gateway filters.
CSRF / XSS in Admin U | Use CSRF tokens in forms, Content Security Policy (CSP), and input validation.
Exposed APIs | Secure APIs with JWT (OAuth2), validate tokens per service, and limit scopes.


Observability Strategy
Component | Strategy
-- | --
Metrics | Collected via Prometheus. Track request rates, error rates, DB latency, Kafka queue length, and memory/CPU usage per pod.
Logs | Aggregate logs using Loki (or Fluentd/Filebeat → Elasticsearch). Structure logs with correlation IDs (trace ID per request).
Tracing | Use OpenTelemetry + Jaeger or Tempo to trace request flows across services.
Dashboards | Visualized in Grafana: service health, budget request volumes, approvals per region, etc.
Alerting | Alert via Grafana or Prometheus AlertManager: high error rates, DB down, Kafka lag, etc.
Who Monitors? | DevOps or SRE team (or on-call rotation) gets alerts via Slack, email, or PagerDuty.


Deployment Strategy | Element	Choice
-- | --
Strategy | Use Rolling Updates for regular deploys. Blue-Green for risky upgrades (schema changes).
Rollback | Rollback via Kubernetes kubectl rollout undo, or swap Ingress route in Blue-Green setup.
Staging/Prod | Parity	Ensure 100% config + data parity via shared Helm charts and secrets templating.
Canary Releases (optional) | Deploy a subset (e.g., 5%) of traffic to new version to detect regressions.
Zero Downtime | Ensure readiness and liveness probes are configured correctly for graceful rolling.

Optional: Use Helm or Kustomize to manage environments cleanly.

Failover & Recovery
Failure Scenario | Response Strategy
-- | --
Kafka Down | Services that produce events retry with exponential backoff. Consumer services cache & retry when Kafka is available. Monitor Kafka health.
PostgreSQL | Down	Trigger alerts. Services queue writes if applicable, or fail fast with retries. Implement automated DB failover (e.g., Patroni or cloud RDS Multi-AZ).
Pod Crash / Node Failure | Kubernetes auto-restarts failed pods. Use anti-affinity rules and replica sets. Ensure persistent volumes are attached correctly.
Notification Failure | Retry logic in Notification Service for FCM, SES, or Twilio. Log undelivered messages for future retry.
Ingress Controller Crash | Auto-heal via replica count >1. Use readiness probes. Ensure separate failover LB setup for HA.
JWT Key Rotation Issue | Keep JWKS key cache with short TTL. Sync key rotation through CI/CD pipelines and notify all services of changes.

Optional: Periodic backups of DBs, audit logs, and Kafka topics to offsite storage (e.g., S3 or local volumes).
