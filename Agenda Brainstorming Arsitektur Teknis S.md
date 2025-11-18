Agenda Brainstorming Arsitektur Teknis Sistem

Sistem Manajemen Uang Saku Digital Berbasis Biometrik - MVP Technical Architecture

Tujuan Meeting

Membahas dan memutuskan arsitektur teknis sistem untuk MVP, dengan fokus pada implementasi yang aman, scalable, dan reliable untuk pilot project di 1-2 sekolah.

CRITICAL - Arsitektur Keamanan Data

1. Data Biometrik Storage & Processing

Pertanyaan untuk Tech Lead/Security Engineer:

Bagaimana skema penyimpanan data biometrik?

Hash-only dengan template matching?

Encrypted database dengan AES-256?

Secure enclave/TEE (Trusted Execution Environment)?

Apakah data biometrik disimpan di server atau hanya di device?

Jika di server: bagaimana struktur database-nya?

Jika di device: bagaimana sync mechanism untuk backup?

Hash algorithm apa yang digunakan untuk fingerprint template?

Keputusan yang harus diambil:

Storage architecture: centralized vs edge computing

Encryption standard dan key management strategy

Data retention policy dari sisi teknis

2. Arsitektur Enkripsi & Security Layer

Pertanyaan untuk Tech Lead:

Layer mana yang akan di-encrypt?

Transport layer (HTTPS/TLS)?

Application layer (end-to-end)?

Database layer (encryption at rest)?

Key management: bagaimana store dan rotate encryption keys?

Certificate management untuk HTTPS?

Rate limiting dan DDoS protection strategy?

Session management dan token expiration policy?

Audit logging: apa saja yang di-log dan bagaimana storage-nya?

Keputusan yang harus diambil:

Tech stack untuk security layer

Authentication mechanism: JWT, OAuth2, atau custom?

Authorization model: RBAC, ABAC?

HIGH - System Architecture & Design

3. Overall System Architecture

Pertanyaan untuk System Architect:

Arsitektur sistem: Monolith atau Microservices?

Jika monolith: bagaimana structure module-nya?

Jika microservices: service apa saja yang akan ada?

Communication pattern antar service (jika microservices):

REST API?

gRPC?

Message queue (RabbitMQ, Kafka)?

API Gateway atau direct service-to-service?

Service mesh perlu atau tidak?

Keputusan yang harus diambil:

Architecture pattern yang akan digunakan

Communication protocol standard

API versioning strategy

4. Database Design & Data Model

Pertanyaan untuk Backend Lead:

Database choice: PostgreSQL, MongoDB, MySQL, atau hybrid?

Database schema design:

Table/collection apa saja yang diperlukan?

Relationship antar entity bagaimana?

Index strategy untuk query optimization?

Sharding strategy jika perlu scale horizontal?

Read replica untuk load distribution?

Caching layer: Redis, Memcached?

Database migration strategy dan tool?

Keputusan yang harus diambil:

Primary database technology

Data normalization level

Backup frequency dan retention

5. Backend Services Architecture

Pertanyaan untuk Backend Lead:

Backend framework: Node.js (Express/NestJS), Python (Django/FastAPI), Go, Java (Spring Boot)?

Service structure:

Authentication Service

Transaction Service

User Management Service

Fingerprint Processing Service

Notification Service

Reporting/Analytics Service

Background job processing: Celery, Bull, Sidekiq?

Scheduled tasks: Cron jobs, job scheduler?

File storage untuk reports/exports: S3, MinIO, local filesystem?

Keputusan yang harus diambil:

Backend tech stack

Service separation strategy

Background job infrastructure

6. Frontend Architecture

Pertanyaan untuk Frontend Lead:

Platform target:

Mobile app: React Native, Flutter, Native (Swift/Kotlin)?

Web app: React, Vue, Angular, atau pure HTML/JS?

Desktop app perlu atau tidak?

State management: Redux, MobX, Context API, Zustand?

Offline capability requirement?

Local storage strategy: SQLite, IndexedDB, AsyncStorage?

Sync mechanism dengan server bagaimana?

UI component library atau custom?

Build dan deployment pipeline?

Keputusan yang harus diambil:

Frontend tech stack

Platform priority: mobile-first atau web-first?

Offline mode implementation strategy

HIGH - Scalability & Performance

7. Skalabilitas & Load Handling

Pertanyaan untuk System Architect:

Ekspektasi load untuk pilot:

Berapa concurrent users maksimal?

Berapa transaksi per detik (TPS) target?

Peak time: 500 transaksi dalam 15 menit = ~0.5-1 TPS

Horizontal scaling strategy:

Stateless application design?

Load balancer: Nginx, HAProxy, cloud LB?

Container orchestration: Kubernetes, Docker Swarm, atau manual?

Vertical scaling limit: sampai mana sebelum horizontal?

Auto-scaling policy dan trigger?

Connection pooling strategy?

Keputusan yang harus diambil:

Target SLA untuk MVP: 99% atau 99.9%?

Load testing plan dan tools (JMeter, K6, Locust)

Scaling trigger thresholds

8. Caching Strategy

Pertanyaan untuk Backend Lead:

Cache layer mana yang perlu?

Application level (in-memory)?

Distributed cache (Redis)?

CDN untuk static assets?

Cache apa yang akan di-cache?

User session?

Frequently accessed data (user profile, balance)?

Query results?

Cache invalidation strategy?

TTL (Time To Live) untuk setiap cache type?

Keputusan yang harus diambil:

Caching technology

Cache eviction policy

Cache warming strategy

HIGH - Reliability & Fault Tolerance

9. High Availability & Redundancy

Pertanyaan untuk DevOps/Infrastructure Lead:

Single Point of Failure (SPOF) apa saja yang ada?

Redundancy untuk critical components:

Database: primary-replica setup?

Application servers: multi-instance?

Load balancer: active-passive atau active-active?

Health check mechanism untuk setiap service?

Automatic failover strategy?

Circuit breaker pattern implementation?

Keputusan yang harus diambil:

Minimum redundancy level untuk MVP

Health check intervals

Failover automation level

10. Offline Mode & Fallback Mechanism

Pertanyaan untuk System Architect:

Offline mode scope:

Read-only access?

Queue transactions locally?

Full offline capability?

Conflict resolution jika ada data mismatch saat sync?

Fallback mechanism jika fingerprint scanner fail:

PIN backup?

QR code temporary?

Manual override oleh admin?

Timeout handling untuk setiap operation?

Retry policy: exponential backoff?

Keputusan yang harus diambil:

Offline mode implementation level

Sync strategy dan conflict resolution

Fallback mechanism priority

11. Backup & Disaster Recovery

Pertanyaan untuk DevOps Lead:

Backup frequency:

Database: continuous, hourly, daily?

Application config: version controlled?

User data: incremental atau full?

Backup storage location: same region, different region, multi-region?

RTO (Recovery Time Objective): berapa lama untuk restore?

RPO (Recovery Point Objective): data loss maksimal berapa menit/jam acceptable?

Disaster recovery testing schedule?

Backup encryption dan access control?

Keputusan yang harus diambil:

Backup strategy dan schedule

DR plan dan testing frequency

RTO dan RPO targets

MEDIUM - Hardware Integration

12. Fingerprint Scanner Integration

Pertanyaan untuk Hardware/IoT Lead:

Communication protocol dengan scanner:

USB?

Bluetooth?

Network-based (IP)?

SDK atau library dari vendor yang akan digunakan?

Bagaimana error handling jika scanner tidak responding?

Fingerprint matching: di device atau di server?

Template format dari scanner: proprietary atau standard (ISO/ANSI)?

Liveness detection: hardware-based atau software-based?

Calibration dan testing protocol untuk scanner?

Keputusan yang harus diambil:

Integration architecture dengan scanner

Template matching location (edge vs cloud)

Error handling strategy

MEDIUM - Infrastructure & Deployment

13. Cloud Infrastructure

Pertanyaan untuk DevOps Lead:

Cloud provider: AWS, GCP, Azure, atau local hosting?

Region selection: Jakarta, Singapore?

Compute resources:

VM-based (EC2, Compute Engine)?

Container-based (ECS, GKE)?

Serverless (Lambda, Cloud Functions)?

Network architecture:

VPC setup?

Subnet design?

Security groups/firewall rules?

CDN perlu atau tidak?

DNS management?

Keputusan yang harus diambil:

Cloud provider dan region

Compute architecture (VM, container, serverless)

Network topology

14. CI/CD Pipeline

Pertanyaan untuk DevOps Lead:

Version control: Git workflow (GitFlow, trunk-based)?

CI/CD tool: GitHub Actions, GitLab CI, Jenkins, CircleCI?

Pipeline stages:

Build?

Unit test?

Integration test?

Security scan?

Deploy?

Environment setup:

Development?

Staging?

Production?

Deployment strategy:

Blue-green deployment?

Rolling update?

Canary deployment?

Rollback mechanism?

Keputusan yang harus diambil:

CI/CD tooling

Deployment strategy

Environment architecture

15. Monitoring & Observability

Pertanyaan untuk DevOps/Backend Lead:

Monitoring stack:

Metrics: Prometheus, CloudWatch, Datadog?

Logging: ELK Stack, CloudWatch Logs, Loki?

Tracing: Jaeger, Zipkin, OpenTelemetry?

Metrics yang akan di-monitor:

Application metrics (latency, error rate, throughput)?

Infrastructure metrics (CPU, memory, disk, network)?

Business metrics (transaction count, success rate)?

Alerting strategy:

Alert threshold untuk setiap metric?

Alert channel: email, Slack, PagerDuty?

On-call rotation?

Log retention policy?

APM (Application Performance Monitoring) perlu atau tidak?

Keputusan yang harus diambil:

Observability stack

Key metrics dan SLIs (Service Level Indicators)

Alerting rules dan escalation

DELIVERABLES DARI MEETING INI

Architecture Decision Records (ADR)

Decision Area

Options Considered

Decision

Rationale

Owner

Status

Architecture Pattern

Monolith / Microservices

TBD

TBD

System Architect

Pending

Database

PostgreSQL / MongoDB / MySQL

TBD

TBD

Backend Lead

Pending

Cloud Provider

AWS / GCP / Azure

TBD

TBD

DevOps Lead

Pending

Backend Framework

Node.js / Python / Go / Java

TBD

TBD

Backend Lead

Pending

Mobile Framework

React Native / Flutter / Native

TBD

TBD

Frontend Lead

Pending

Encryption

AES-256 / RSA / Hybrid

TBD

TBD

Security Engineer

Pending

Technical Documentation To Create

System Architecture Diagram (C4 Model: Context, Container, Component, Code)

Database Schema Design (ERD)

API Design Document (OpenAPI/Swagger spec)

Security Architecture Document

Infrastructure Architecture Diagram

Data Flow Diagram

Sequence Diagrams untuk critical flows

Deployment Architecture Diagram

POC/Prototype Requirements

Fingerprint scanner integration POC (3 vendor testing)

Load testing untuk target TPS

Offline mode sync mechanism POC

Security penetration testing scope

Technical Stack Decision Deadline

Target: 1 minggu dari meeting ini

Critical Path:

Architecture pattern decision (Day 1-2)

Tech stack selection (Day 3-4)

Database design (Day 5-6)

Infrastructure planning (Day 6-7)

Architecture review dan finalization (Day 7)

Reference Documents

PDF: Analisis Resiko - Bagian Resiko Teknis

ISO/IEC 27001 Security Standards

OWASP Top 10 Security Risks

Cloud Provider Best Practices (AWS Well-Architected, GCP Best Practices)

Database Design Patterns

Microservices Patterns (if applicable)

Meeting Duration: 4-5 jam (bisa dipecah menjadi 2 session)

Required Participants:

System Architect (Lead)

Backend Lead

Frontend Lead

DevOps/Infrastructure Lead

Security Engineer

Hardware/IoT Integration Lead (jika applicable)

Output Expected:

Completed Architecture Decision Records

Clear tech stack untuk seluruh layer

Timeline untuk technical documentation

Action items dengan owner dan deadline
