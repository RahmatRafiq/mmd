# MMD - Multi-Tenant Cashless Payment Platform

**Sistem Manajemen Uang Saku Digital Berbasis Biometrik**

Platform pembayaran cashless multi-tenant dengan autentikasi biometrik (fingerprint) untuk sekolah, perusahaan, dan koperasi.

---

## Ringkasan Proyek

**Nama Proyek**: Multi-Tenant Cashless Payment Platform
**Singkatan**: MMD (Money Management Digital)
**Fase**: MVP - Pilot Project (1-2 sekolah)
**Target Users**: Sekolah, Perusahaan, Koperasi

### Fitur Utama

1. **Multi-Tenant Architecture**
   - Satu platform untuk banyak organisasi (sekolah/perusahaan)
   - Setiap organisasi punya tenant sendiri (kantin, koperasi, dll)
   - Subscription tier: Trial, Basic, Premium

2. **Biometric Authentication**
   - Fingerprint scanner integration
   - Liveness detection
   - Encrypted template storage
   - Fallback mechanism (PIN, QR code)

3. **Digital Wallet System**
   - Member wallet dengan balance tracking
   - Top-up via bank transfer, e-wallet, QRIS
   - Spending limits (daily, per-transaction, time-based)
   - Real-time balance updates

4. **Transaction Management**
   - POS terminal integration
   - Menu item management per tenant
   - Transaction tracking dan reporting
   - Refund mechanism

5. **Guardian Portal**
   - Parent/Guardian dapat monitor child spending
   - Top-up wallet remotely
   - Spending limit configuration
   - Real-time notifications

6. **Multi-Role Access**
   - Super Admin (platform level)
   - Organization Admin
   - Tenant Admin
   - Cashier

---

## Struktur Data Utama

### Core Entities (6)
- **ORGANIZATION** - Sekolah/Perusahaan induk
- **USER** - Admin, cashier, staff
- **TENANT** - Kantin, koperasi, tenant makanan
- **GUARDIAN** - Orang tua/wali
- **MEMBER** - Siswa/karyawan
- **MEMBER_GUARDIAN** - Relasi member-guardian (many-to-many)

### Financial Entities (3)
- **WALLET** - Dompet digital member
- **TOPUP** - Riwayat pengisian saldo
- **SPENDING_LIMIT** - Batas pengeluaran

### Transaction Entities (3)
- **MENU_ITEM** - Produk/menu yang dijual
- **TRANSACTION** - Transaksi pembayaran
- **TRANSACTION_ITEM** - Detail item per transaksi

### Device Entities (2)
- **DEVICE** - Fingerprint scanner, POS terminal
- **FINGERPRINT** - Template sidik jari member

### Communication (1)
- **NOTIFICATION** - Notifikasi ke guardian/user

**Total: 14 Entities**

---

## Struktur Hierarki

```
ORGANIZATION (Sekolah/Perusahaan)
├── USER (Admin, Cashier)
├── GUARDIAN (Orang Tua)
├── MEMBER (Siswa/Karyawan)
│   ├── WALLET
│   │   ├── TOPUP
│   │   ├── SPENDING_LIMIT
│   │   └── TRANSACTION
│   └── FINGERPRINT (multiple fingers)
├── TENANT (Kantin, Koperasi)
│   ├── MENU_ITEM
│   ├── TRANSACTION
│   └── DEVICE (POS, Scanner)
└── DEVICE (Organization-level devices)
```

---

## Use Case Scenarios

### 1. Top-up Wallet
```
Guardian → Login → Pilih Member → Top-up →
Pilih payment method (Bank/E-wallet/QRIS) →
Payment gateway → Webhook → Update wallet balance →
Notifikasi ke Guardian & Member
```

### 2. Transaction via Fingerprint
```
Member → Scan fingerprint → Verifikasi →
Cashier pilih items → Calculate total →
Check spending limit → Deduct wallet →
Transaction completed → Notifikasi
```

### 3. Daily Spending Limit
```
Guardian → Set daily limit Rp 50.000 →
Set time window 07:00-15:00 →
Member transaksi jam 16:00 → REJECTED →
Notifikasi guardian
```

---

## Target Performance (MVP)

### Load Capacity
- **Concurrent users**: 500 users
- **TPS (Transactions Per Second)**: 0.5-1 TPS
- **Peak load**: 500 transaksi dalam 15 menit (jam istirahat sekolah)

### SLA Target
- **Uptime**: 99% - 99.9%
- **Response time**: < 500ms untuk transaction
- **Fingerprint scan**: < 2 detik untuk verify

### Pilot Scope
- **Sekolah**: 1-2 sekolah
- **Total siswa**: ~500-1000 siswa
- **Tenant**: 2-4 kantin per sekolah

---

## Arsitektur Teknis (Rencana)

### Critical Decisions (TBD)

1. **Architecture Pattern**
   - [ ] Monolith vs Microservices
   - [ ] Communication protocol (REST, gRPC, Message Queue)

2. **Database**
   - [ ] PostgreSQL / MongoDB / MySQL
   - [ ] Sharding strategy
   - [ ] Read replica setup

3. **Backend Framework**
   - [ ] Node.js (Express/NestJS)
   - [ ] Python (Django/FastAPI)
   - [ ] Go / Java (Spring Boot)

4. **Frontend Platform**
   - [ ] Mobile: React Native / Flutter / Native
   - [ ] Web: React / Vue / Angular
   - [ ] Desktop: Electron (optional)

5. **Cloud Provider**
   - [ ] AWS (Jakarta/Singapore region)
   - [ ] GCP
   - [ ] Azure
   - [ ] Local hosting

6. **Security**
   - [ ] Biometric storage: Hash-only / Encrypted DB / TEE
   - [ ] Encryption: AES-256 / RSA / Hybrid
   - [ ] Authentication: JWT / OAuth2
   - [ ] Authorization: RBAC / ABAC

7. **Fingerprint Scanner**
   - [ ] Vendor selection (POC 3 vendors)
   - [ ] Communication: USB / Bluetooth / Network
   - [ ] Template matching: Device-based / Server-based

---

## Keamanan Data

### Critical Security Requirements

1. **Biometric Data Protection**
   - Encrypted at rest (AES-256)
   - Encrypted in transit (TLS 1.3)
   - Template hashing (irreversible)
   - No raw fingerprint storage

2. **Financial Data Security**
   - Transaction logging & audit trail
   - Wallet balance encryption
   - Rate limiting untuk top-up/withdraw
   - Anti-fraud detection

3. **Authentication & Authorization**
   - Multi-factor authentication untuk admin
   - Session timeout & token expiration
   - Role-Based Access Control (RBAC)
   - IP whitelisting untuk critical operations

4. **Privacy Compliance**
   - GDPR-ready (data deletion, export)
   - Audit logs untuk data access
   - Data retention policy
   - Consent management

---

## Reliability & Fault Tolerance

### High Availability Strategy

1. **Redundancy**
   - Database: Primary-Replica setup
   - Application: Multi-instance deployment
   - Load balancer: Active-Active

2. **Offline Mode**
   - Queue transactions locally
   - Sync saat network restored
   - Conflict resolution strategy

3. **Fallback Mechanisms**
   - Fingerprint fail → PIN backup
   - Scanner offline → QR code temporary
   - Payment gateway fail → Manual verification

4. **Backup & Disaster Recovery**
   - Database backup: Daily (minimum)
   - RTO (Recovery Time Objective): < 4 jam
   - RPO (Recovery Point Objective): < 1 jam

---

## Monitoring & Observability

### Key Metrics

1. **Application Metrics**
   - Transaction success rate
   - Fingerprint verification latency
   - API response time
   - Error rate per endpoint

2. **Business Metrics**
   - Daily transaction volume
   - Total GMV (Gross Merchandise Value)
   - Active users (DAU, MAU)
   - Top-up success rate

3. **Infrastructure Metrics**
   - CPU, Memory, Disk, Network usage
   - Database connection pool
   - Queue length
   - Cache hit ratio

4. **Security Metrics**
   - Failed login attempts
   - Suspicious transaction patterns
   - API rate limit violations
   - Unauthorized access attempts

---

## Dokumentasi Terkait

1. **ERD (Entity Relationship Diagram)**
   - File: `ERD-FINAL-SIMPLE.md`
   - 14 entities dengan relasi lengkap

2. **Agenda Brainstorming Arsitektur Teknis**
   - File: `Agenda Brainstorming Arsitektur Teknis S.md`
   - 15 critical questions untuk tech decision

3. **Architecture Decision Records (ADR)** *(Coming Soon)*
   - System architecture pattern
   - Tech stack selection
   - Database design
   - Security architecture
   - Infrastructure planning

4. **API Design Document** *(Coming Soon)*
   - OpenAPI/Swagger specification
   - Authentication flow
   - Error handling standards

5. **Security Architecture Document** *(Coming Soon)*
   - Encryption standards
   - Key management strategy
   - Audit logging requirements

---

## Timeline Rencana

### Week 1: Architecture & Design
- Day 1-2: Architecture pattern decision
- Day 3-4: Tech stack selection
- Day 5-6: Database design finalization
- Day 6-7: Infrastructure planning
- Day 7: Architecture review & approval

### Week 2-4: Development Setup
- CI/CD pipeline setup
- Development environment
- Database migration scripts
- Security layer implementation

### Week 5-8: MVP Development
- Core features implementation
- Fingerprint integration
- Transaction flow
- Guardian portal

### Week 9-10: Testing & Optimization
- Load testing
- Security penetration testing
- Bug fixes
- Performance optimization

### Week 11-12: Pilot Deployment
- Deploy ke 1-2 sekolah
- User training
- Monitoring & support
- Feedback collection

---

## Stakeholders & Roles

| Role | Responsibility | Contact |
|------|---------------|---------|
| System Architect | Overall architecture design | TBD |
| Backend Lead | Backend services, API design | TBD |
| Frontend Lead | Mobile & Web app | TBD |
| DevOps Lead | Infrastructure, deployment | TBD |
| Security Engineer | Security architecture | TBD |
| Hardware/IoT Lead | Fingerprint scanner integration | TBD |
| Product Manager | Requirements, roadmap | TBD |
| QA Lead | Testing strategy | TBD |

---

## Risks & Mitigation

### Critical Risks

1. **Fingerprint Scanner Compatibility**
   - Risk: Vendor lock-in, hardware failure
   - Mitigation: POC 3 vendors, abstraction layer

2. **Data Security Breach**
   - Risk: Biometric data leak
   - Mitigation: Encryption, security audit, penetration testing

3. **Performance at Peak Load**
   - Risk: System slow/crash saat jam istirahat
   - Mitigation: Load testing, auto-scaling, caching

4. **Offline Transaction Sync Conflicts**
   - Risk: Data inconsistency
   - Mitigation: Robust conflict resolution, transaction queue

5. **Regulatory Compliance**
   - Risk: Privacy law violations
   - Mitigation: Legal review, GDPR compliance, data governance

---

## Contact & Support

- **Project Repository**: TBD
- **Documentation**: `/docs` folder
- **Issue Tracker**: TBD
- **Slack/Discord**: TBD

---

**Last Updated**: 2025-11-17
**Version**: 0.1.0 (MVP Planning Phase)
