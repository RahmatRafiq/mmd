# ERD Final - Multi-Tenant Cashless Payment Platform

```mermaid
erDiagram

    ORGANIZATION {
        uuid id PK
        string name "Nama sekolah/perusahaan"
        string slug "UNIQUE"
        enum type "school, company, cooperative"
        string address
        string city
        string phone
        string email "UNIQUE"
        string logo_url
        enum subscription_tier "trial, basic, premium"
        datetime subscription_start
        datetime subscription_end
        boolean is_active
        json settings
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    USER {
        uuid id PK
        uuid organization_id FK "NULL untuk super_admin"
        string username "UNIQUE"
        string email "UNIQUE"
        string phone
        string password_hash
        string avatar_url
        enum role "super_admin, org_admin, tenant_admin, cashier"
        boolean is_active
        datetime last_login_at
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    TENANT {
        uuid id PK
        uuid organization_id FK
        string name "Kantin Pusat, Tenant Makanan, Tenant Minuman, Koperasi"
        string slug
        string location
        boolean is_active
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    GUARDIAN {
        uuid id PK
        uuid organization_id FK
        string name
        string email "UNIQUE"
        string phone "UNIQUE"
        string password_hash
        string nik
        boolean is_verified
        datetime email_verified_at
        datetime phone_verified_at
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    MEMBER {
        uuid id PK
        uuid organization_id FK
        uuid guardian_id FK "primary guardian"
        string member_number "unique per organization"
        string name
        string photo_url
        date birth_date
        enum gender "male, female"
        string class_group
        string phone
        string email
        boolean is_active
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    MEMBER_GUARDIAN {
        uuid id PK
        uuid member_id FK
        uuid guardian_id FK
        enum relationship "father, mother, sibling, relative, other"
        boolean is_primary
        boolean can_topup
        boolean can_view_transactions
        datetime created_at
        datetime updated_at
        UNIQUE member_id_guardian_id
    }

    WALLET {
        uuid id PK
        uuid member_id FK "UNIQUE"
        decimal balance
        string currency
        boolean is_active
        boolean is_frozen
        datetime created_at
        datetime updated_at
    }

    TOPUP {
        uuid id PK
        uuid wallet_id FK
        uuid guardian_id FK
        string topup_code "UNIQUE"
        decimal amount
        enum payment_method "bank_transfer, ewallet, qris"
        string payment_reference
        enum status "pending, completed, failed, expired"
        json payment_metadata
        datetime processed_at
        datetime created_at
        datetime updated_at
    }

    SPENDING_LIMIT {
        uuid id PK
        uuid wallet_id FK
        decimal daily_limit
        decimal per_transaction_limit
        time allowed_start_time
        time allowed_end_time
        boolean is_active
        datetime created_at
        datetime updated_at
    }

    MENU_ITEM {
        uuid id PK
        uuid tenant_id FK
        string name
        text description
        decimal price
        string category
        string image_url
        boolean is_available
        int stock_quantity
        boolean track_stock
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    TRANSACTION {
        uuid id PK
        uuid member_id FK
        uuid wallet_id FK
        uuid tenant_id FK
        uuid device_id FK
        uuid cashier_id FK
        string transaction_code "UNIQUE"
        decimal total_amount
        enum status "completed, failed, refunded"
        enum payment_method "wallet, cash"
        datetime transaction_time
        json verification_data
        text notes
        datetime created_at
        datetime updated_at
    }

    TRANSACTION_ITEM {
        uuid id PK
        uuid transaction_id FK
        uuid menu_item_id FK
        string item_name
        decimal item_price
        int quantity
        decimal subtotal
        datetime created_at
    }

    DEVICE {
        uuid id PK
        uuid organization_id FK
        uuid tenant_id FK
        string device_code "UNIQUE"
        string device_serial
        enum device_type "fingerprint_scanner, pos_terminal"
        string ip_address
        enum status "active, inactive, maintenance"
        datetime last_heartbeat_at
        datetime created_at
        datetime updated_at
    }

    FINGERPRINT {
        uuid id PK
        uuid member_id FK
        enum finger_position "thumb_left, thumb_right, index_left, index_right"
        blob encrypted_template
        string template_hash
        float quality_score
        datetime enrolled_at
        boolean is_active
        int failed_attempts
        datetime locked_until
        datetime created_at
        datetime updated_at
        UNIQUE member_id_finger_position
    }

    NOTIFICATION {
        uuid id PK
        enum recipient_type "guardian, user"
        uuid recipient_id
        string title
        text message
        enum type "topup, transaction, low_balance, alert"
        boolean is_read
        datetime read_at
        json data
        datetime created_at
    }

    %% Relationships
    ORGANIZATION ||--o{ USER : has
    ORGANIZATION ||--o{ TENANT : has
    ORGANIZATION ||--o{ GUARDIAN : has
    ORGANIZATION ||--o{ MEMBER : has
    ORGANIZATION ||--o{ DEVICE : owns

    MEMBER ||--o{ MEMBER_GUARDIAN : linked_via
    GUARDIAN ||--o{ MEMBER_GUARDIAN : linked_via
    GUARDIAN ||--|| MEMBER : primary_guardian_of

    MEMBER ||--|| WALLET : has
    WALLET ||--o{ TOPUP : receives
    WALLET ||--o{ TRANSACTION : spends
    WALLET ||--o{ SPENDING_LIMIT : has
    GUARDIAN ||--o{ TOPUP : creates

    TENANT ||--o{ MENU_ITEM : sells
    TENANT ||--o{ TRANSACTION : processes
    TENANT ||--o{ DEVICE : uses

    TRANSACTION ||--o{ TRANSACTION_ITEM : contains
    MENU_ITEM ||--o{ TRANSACTION_ITEM : sold_in
    MEMBER ||--o{ TRANSACTION : makes
    USER ||--o{ TRANSACTION : processes_as_cashier
    DEVICE ||--o{ TRANSACTION : processes

    MEMBER ||--o{ FINGERPRINT : uses

    GUARDIAN ||--o{ NOTIFICATION : receives
    USER ||--o{ NOTIFICATION : receives
```

---

## Struktur Hierarki

```
ORGANIZATION (Sekolah/Perusahaan)
├── TENANT 1 (Kantin Pusat)
│   ├── MENU_ITEM (Nasi Goreng, Es Teh, dll)
│   └── DEVICE (Fingerprint Scanner)
├── TENANT 2 (Tenant Makanan Berat)
│   ├── MENU_ITEM (Nasi Uduk, Soto, dll)
│   └── DEVICE (POS Terminal)
├── TENANT 3 (Tenant Minuman & Snack)
│   └── MENU_ITEM (Aqua, Chitato, dll)
└── MEMBER (Siswa/Karyawan)
    ├── WALLET
    └── FINGERPRINT
```

---

## Total: 14 Entities

**Core (6)**:
- ORGANIZATION
- USER
- TENANT
- GUARDIAN
- MEMBER
- MEMBER_GUARDIAN

**Financial (3)**:
- WALLET
- TOPUP
- SPENDING_LIMIT

**Transaction (3)**:
- MENU_ITEM
- TRANSACTION
- TRANSACTION_ITEM

**Device (2)**:
- DEVICE
- FINGERPRINT

**Communication (1)**:
- NOTIFICATION
