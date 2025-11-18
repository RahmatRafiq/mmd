# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MMD (Money Management Digital)** - School Cashless Payment Platform with biometric fingerprint authentication.

This is a digital wallet system with fingerprint authentication designed specifically for schools. Students use fingerprint scanners to make cashless payments at school canteens, while parents can remotely top-up wallets and monitor their children's spending.

**Current Phase**: MVP Planning - Pre-development (no code implementation yet)
**Target**: Pilot deployment to 1-2 schools with 500-1000 students

## Core Concepts

### School-Centric Architecture
- Each SCHOOL has multiple CANTEENs (kantin pusat, koperasi siswa, etc.)
- STUDENTs belong to one school but can buy at any canteen within the school
- Subscription tiers: Trial, Basic, Premium

### Key Actors
- **SCHOOL**: Sekolah yang menggunakan platform
- **CANTEEN**: Kantin/koperasi dalam sekolah
- **STUDENT**: Siswa dengan wallet dan fingerprint
- **PARENT**: Orang tua/wali yang top-up dan monitor spending
- **STAFF**: Platform users (super_admin, school_admin, canteen_admin, cashier)

### Critical Flows
1. **Top-up**: Parent → Payment Gateway → Wallet Balance Updated → Notifications sent
2. **Purchase**: Student scans fingerprint → System verifies → Cashier adds items → Check spending limits → Deduct from wallet → Complete purchase
3. **Spending Control**: Parent sets daily limits and time windows (e.g., only 07:00-15:00, max Rp 50,000/day)

## Database Architecture

The system uses **14 core tables** organized into functional groups (school-specific naming):

### Core School Management (6 tables)
- **SCHOOL** - Sekolah with subscription management (Trial, Basic, Premium)
- **STAFF** - System users with roles: super_admin, school_admin, canteen_admin, cashier
- **CANTEEN** - Kantin/koperasi within a school
- **PARENT** - Orang tua/wali who manage student wallets
- **STUDENT** - Siswa with wallets and fingerprints
- **STUDENT_PARENT** - Many-to-many relationship with permissions (can_topup, can_view_transactions, can_set_limits)

### Financial Management (3 tables)
- **WALLET** - Multiple wallets per student (main, savings) with balance tracking
- **WALLET_TOPUP** - Top-up transactions with payment gateway integration
- **SPENDING_LIMIT** - Advanced limits: daily_limit, time windows, category restrictions, date ranges

### Transaction Management (3 tables)
- **MENU_ITEM** - Products sold by canteens with stock tracking
- **PURCHASE** - Payment records with verification data and refund support
- **PURCHASE_ITEM** - Line items within each purchase

### Device & Security (2 tables)
- **DEVICE** - Fingerprint scanners and POS terminals
- **FINGERPRINT** - Encrypted biometric templates with security features (failed_attempts, locked_until)

### Communication (1 table)
- **NOTIFICATION** - Alerts sent to parents and staff

### Key Relationships
- STUDENT ||--o{ WALLET (one student can have multiple wallets: main + savings)
- STUDENT ||--o{ STUDENT_PARENT ||--o{ PARENT (many-to-many with relationship type)
- STUDENT ||--o{ FINGERPRINT (multiple fingers per student)
- PURCHASE ||--o{ PURCHASE_ITEM ||--o{ MENU_ITEM

**Full ERD**: See `ERD-V3-SCHOOL-SPECIFIC.md` for complete schema with all fields and constraints.

## Technical Decisions Pending

The following architecture decisions are documented in `Agenda Brainstorming Arsitektur Teknis S.md` and need to be finalized:

### Critical Priorities
1. **Architecture Pattern**: Monolith vs Microservices
2. **Database**: PostgreSQL vs MongoDB vs MySQL (with multi-tenancy considerations)
3. **Backend Framework**: Node.js (Express/NestJS) vs Python (Django/FastAPI) vs Go vs Java
4. **Mobile Framework**: React Native vs Flutter vs Native
5. **Cloud Provider**: AWS vs GCP vs Azure (target: Jakarta/Singapore region)

### Security Architecture
- **Biometric Storage**: Hash-only vs Encrypted DB vs TEE (Trusted Execution Environment)
- **Encryption**: AES-256 for data at rest, TLS 1.3 for transit
- **Authentication**: JWT vs OAuth2 for API access
- **Authorization**: RBAC (Role-Based Access Control) implementation
- **Template Matching**: Edge (device-based) vs Cloud (server-based) fingerprint verification

### Fingerprint Scanner Integration
- POC required for 3 vendors before selection
- Communication protocol: USB vs Bluetooth vs Network-based
- Template format: Proprietary vs ISO/ANSI standard
- Liveness detection requirements

### Performance Targets (MVP)
- Concurrent users: 500
- Target TPS: 0.5-1 transactions per second
- Peak load: 500 transactions in 15 minutes (school recess)
- Fingerprint verification: < 2 seconds
- Transaction response time: < 500ms
- Uptime SLA: 99%-99.9%

## Security Considerations

### Critical Requirements
1. **No Raw Biometric Storage**: Only encrypted/hashed templates stored
2. **Encryption Layers**:
   - Transport: TLS 1.3 for all API calls
   - Application: End-to-end encryption for sensitive data
   - Database: Encryption at rest (AES-256)
3. **Audit Trail**: All financial transactions and data access must be logged
4. **Rate Limiting**: Prevent abuse of top-up, authentication, and transaction endpoints
5. **Privacy Compliance**: GDPR-ready with data deletion/export capabilities

### Fallback Mechanisms
- Fingerprint scanner fails → PIN backup authentication
- Network offline → Queue transactions locally, sync when restored
- Payment gateway failure → Manual verification flow

## Development Phases

### Current Status: Week 0 (Planning)
The repository contains planning documentation only. No code has been written yet.

### Planned Timeline
- **Week 1**: Finalize architecture decisions and tech stack
- **Week 2-4**: Development environment setup, CI/CD pipeline, database migrations
- **Week 5-8**: MVP feature development
- **Week 9-10**: Load testing, security testing, optimization
- **Week 11-12**: Pilot deployment to 1-2 schools

## Important Files

- **`ERD-V3-SCHOOL-SPECIFIC.md`** - ✅ AUTHORITATIVE - Complete database schema with 14 tables (school-specific naming)
- `ERD-V2-IMPROVED.md` - Enhanced ERD with V1 improvements (generic naming - superseded by V3)
- `ERD-COMPARISON-V1-V2.md` - Detailed comparison of V1 vs V2 changes and migration guide
- `ERD-FINAL-SIMPLE.md` - Original V1 ERD (deprecated - use V3)
- `erDiagram.mmd` - Early draft ERD (deprecated)
- `mmd.code-workspace.md` - Project overview and stakeholder information
- `Agenda Brainstorming Arsitektur Teknis S.md` - 15 critical architecture questions and decision framework

## Design Principles

1. **School Data Isolation**: Every query must be scoped to school_id to prevent data leakage between schools
2. **Security by Default**: All sensitive operations require authentication and authorization checks
3. **Offline Resilience**: Support offline purchase queuing with conflict resolution
4. **Audit Everything**: Financial operations and biometric access must be fully traceable
5. **Graceful Degradation**: System should have fallback mechanisms for critical flows

## Common Patterns to Follow

### School Data Isolation
```sql
-- Every query should filter by school_id:
SELECT * FROM student WHERE school_id = ? AND id = ?
SELECT * FROM purchase WHERE school_id = ? AND purchase_time >= ?

-- Prevent cross-school data access at the database/ORM level
```

### Spending Limit Enforcement
```
Before processing purchase:
1. Check wallet.is_active AND NOT wallet.is_frozen
2. Get all active SPENDING_LIMIT for this wallet (matching today's date)
3. Apply most restrictive limit (based on priority)
4. Verify:
   - Current time within allowed_start_time and allowed_end_time
   - Daily spending total + purchase_amount <= daily_limit
   - purchase_amount <= per_transaction_limit
   - Menu item categories NOT IN blocked_categories
5. If any check fails → REJECT purchase
```

### Fingerprint Verification Flow
```
1. Student scans fingerprint on DEVICE
2. Check fingerprint.is_active
3. Verify NOT locked (failed_attempts < threshold OR locked_until has passed)
4. Perform template matching
5. On failure: increment failed_attempts, lock if threshold exceeded
6. On success: reset failed_attempts, update last_verified_at, return student_id
```

### Purchase Idempotency
```
All purchases must have unique purchase_code
Format: PU[YYYYMMDD][SEQUENCE] → PU20251117001
Implement idempotency to prevent duplicate charges if request is retried
Store verification_data JSON for audit purposes (fingerprint quality, device info)
```

## Future Implementation Notes

### When implementing backend services:
- Implement robust logging for all financial operations
- Use database transactions for wallet balance updates (prevent race conditions)
- Implement job queues for async operations (notifications, report generation)
- Add health check endpoints for all services
- Implement circuit breaker pattern for external service calls (payment gateways)

### When implementing frontend apps:
- Support offline mode with local transaction queue
- Implement optimistic UI updates with rollback on failure
- Cache frequently accessed data (member profiles, menu items)
- Real-time balance updates via WebSocket or polling
- Handle biometric enrollment and re-enrollment flows

### When implementing DevOps:
- Separate environments: development, staging, production
- Database migration strategy with rollback capability
- Automated backup with tested restore procedures
- Monitoring for key metrics: transaction_success_rate, wallet_balance_consistency, fingerprint_verification_latency
- Alert on: failed_transaction_spike, database_connection_errors, high_api_latency

## Monitoring & Observability

### Key Metrics to Track
- **Transaction Success Rate**: Target > 99%
- **Fingerprint Verification Latency**: Target < 2s
- **Wallet Balance Consistency**: Regular reconciliation jobs
- **Daily Transaction Volume**: Business metric
- **Failed Authentication Attempts**: Security metric
- **API Response Time P95**: Target < 500ms

### Critical Alerts
- Transaction failure rate > 5%
- Database connection pool exhausted
- Payment gateway integration errors
- Unusual spending patterns (fraud detection)
- Biometric verification failures spike

## Risks & Mitigations

1. **Fingerprint Scanner Vendor Lock-in**: Create hardware abstraction layer to support multiple vendors
2. **Peak Load Performance**: Implement caching, load testing before pilot, auto-scaling infrastructure
3. **Offline Transaction Conflicts**: Design robust conflict resolution (server wins vs client wins vs merge strategies)
4. **Biometric Data Breach**: Never store raw fingerprints, encrypt templates, regular security audits
5. **Regulatory Compliance**: GDPR compliance from day one, legal review before pilot

## Notes for AI Assistants

- This project is in **planning phase** - no code exists yet
- Major architectural decisions are still pending (see Agenda document)
- When making implementation suggestions, consider multi-tenant security implications
- Performance targets are modest for MVP (0.5-1 TPS) but design should support future scale
- Financial accuracy and biometric security are the highest priorities
- Offline capability is required, not optional
