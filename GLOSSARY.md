# Glossary - MMD Platform

Dokumen ini berisi definisi istilah, tabel, dan field yang digunakan dalam sistem MMD (Money Management Digital).

---

## Konsep Utama

### Organization
Entitas utama yang menggunakan platform. Bisa berupa sekolah, perusahaan, korporasi, retail, atau instansi pemerintah.

### Member
Pengguna akhir yang memiliki wallet dan melakukan transaksi. Contoh: siswa (school), karyawan (industry), pegawai (corporate).

### Guardian
Wali atau orang tua yang memiliki hak untuk mengelola wallet member (khusus sektor school). Dapat melakukan top-up, melihat transaksi, dan mengatur spending limit.

### Tenant
Unit usaha dalam organization yang menjual produk/jasa. Contoh: kantin pusat, koperasi siswa, cafeteria, toko.

### Staff
Pengguna sistem yang mengelola operasional. Memiliki role seperti super_admin, org_admin, tenant_admin, cashier.

### Wallet
Dompet digital yang menyimpan saldo member. Setiap member dapat memiliki beberapa tipe wallet (main, savings, meal_allowance, transport).

---

## Sektor (Sector Types)

| Sector | Deskripsi | Member Type | Managed By |
|--------|-----------|-------------|------------|
| **school** | Sekolah (SD, SMP, SMA, SMK) | student | Guardian |
| **industry** | Pabrik, manufaktur | employee/worker | Self |
| **corporate** | Kantor, perusahaan | employee | Self |
| **retail** | Toko, outlet | customer | Self |
| **government** | Instansi pemerintah | employee | Self |

---

## Tabel dan Field

### Core Organization & User Management

#### organization
Entitas organisasi yang menggunakan platform.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| name | string | Nama organisasi |
| slug | string | URL-friendly identifier (unique) |
| sector_type | enum | Jenis sektor: school, industry, corporate, retail, government |
| address | string | Alamat lengkap |
| city | string | Kota |
| province | string | Provinsi |
| postal_code | string | Kode pos |
| phone | string | Nomor telepon |
| email | string | Email organisasi (unique) |
| logo_url | string | URL logo |
| subscription_tier | enum | Tier langganan: trial, basic, premium, enterprise |
| subscription_start | datetime | Tanggal mulai langganan |
| subscription_end | datetime | Tanggal berakhir langganan |
| is_active | boolean | Status aktif organisasi |
| sector_settings | json | Konfigurasi spesifik per sektor |

#### user
Akun login untuk mengakses sistem.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| username | string | Username untuk login (unique) |
| email | string | Email untuk login (unique) |
| password_hash | string | Hash password (bcrypt/argon2) |
| email_verified_at | datetime | Waktu verifikasi email (NULL = belum verified) |
| remember_token | string | Token untuk fitur "remember me" |
| last_password_change_at | datetime | Waktu terakhir ganti password |
| is_active | boolean | Status aktif akun |
| last_login_at | datetime | Waktu login terakhir |

#### profile
Informasi profil pengguna.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| user_id | uuid | FK ke user (one-to-one) |
| name | string | Nama lengkap |
| phone | string | Nomor telepon |
| photo_url | string | URL foto profil |
| address | string | Alamat |
| is_public | boolean | Profil publik atau privat |

#### staff
Pengelola sistem dengan role tertentu.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| user_id | uuid | FK ke user (unique) |
| organization_id | uuid | FK ke organization (NULL untuk super_admin) |
| is_active | boolean | Status aktif staff |

#### guardian
Wali/orang tua yang mengelola member (khusus school).

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| user_id | uuid | FK ke user (unique) |
| organization_id | uuid | FK ke organization |
| nik | string | Nomor Induk Kependudukan (KTP) |
| phone | string | Nomor telepon |
| is_verified | boolean | Status verifikasi oleh admin |
| phone_verified_at | datetime | Waktu verifikasi OTP telepon |

#### tenant
Unit usaha dalam organization.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| organization_id | uuid | FK ke organization |
| name | string | Nama tenant (Kantin Pusat, Cafeteria) |
| slug | string | URL-friendly identifier (unique per org) |
| location | string | Lokasi (Lantai 1 Gedung A) |
| description | string | Deskripsi tenant |
| is_active | boolean | Status aktif tenant |

#### member
Pengguna akhir yang memiliki wallet.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| user_id | uuid | FK ke user (NULL untuk school, REQUIRED untuk industry) |
| organization_id | uuid | FK ke organization |
| member_number | string | NIS/Employee ID (unique per org) |
| name | string | Nama lengkap |
| photo_url | string | URL foto |
| birth_date | date | Tanggal lahir |
| gender | enum | Jenis kelamin: male, female, other |
| member_type | enum | Tipe: student, employee, worker, customer |
| self_managed | boolean | true = kelola sendiri (industry), false = via guardian (school) |
| grade | string | Kelas untuk school (7, 8, 9, 10, 11, 12) |
| class_name | string | Nama kelas untuk school (7A, 7B) |
| department | string | Departemen untuk industry (IT, Finance, HR) |
| position | string | Jabatan untuk industry (Manager, Staff) |
| address | string | Alamat |
| phone | string | Nomor telepon |
| email | string | Email (REQUIRED untuk industry) |
| pin_hash | string | Hash PIN 6 digit (backup jika fingerprint fail) |
| can_set_own_limits | boolean | Izin set spending limit sendiri |
| can_topup_own_wallet | boolean | Izin top-up wallet sendiri |
| is_active | boolean | Status aktif member |

#### member_guardian
Relasi many-to-many antara member dan guardian dengan permissions.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| member_id | uuid | FK ke member |
| guardian_id | uuid | FK ke guardian |
| relationship | enum | Hubungan: father, mother, grandfather, grandmother, sibling, uncle, aunt, guardian, emergency_contact, other |
| is_primary | boolean | Guardian utama (satu per member) |
| can_topup | boolean | Izin top-up wallet |
| can_view_transactions | boolean | Izin lihat riwayat transaksi |
| can_set_limits | boolean | Izin set spending limit |

---

### Financial Management

#### wallet
Dompet digital member.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| member_id | uuid | FK ke member |
| wallet_type | enum | Tipe: main, savings, meal_allowance, transport |
| wallet_name | string | Nama custom (Uang Jajan, Tunjangan Makan) |
| balance | decimal | Saldo saat ini |
| pending_balance | decimal | Saldo pending (top-up belum confirmed) |
| currency | string | Mata uang (IDR, USD) |
| is_primary | boolean | Wallet utama (satu per member) |
| is_active | boolean | Status aktif wallet |
| is_frozen | boolean | Wallet dibekukan |
| frozen_reason | string | Alasan pembekuan |
| frozen_at | datetime | Waktu pembekuan |

#### wallet_history
Riwayat semua mutasi wallet.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| wallet_id | uuid | FK ke wallet |
| organization_id | uuid | FK ke organization (denormalized) |
| entry_type | enum | Tipe: credit (masuk), debit (keluar) |
| amount | decimal | Jumlah (selalu positif) |
| balance_before | decimal | Saldo sebelum transaksi |
| balance_after | decimal | Saldo setelah transaksi |
| transaction_type | enum | Jenis: topup, purchase, refund, adjustment, transfer_in, transfer_out |
| reference_type | string | Tipe referensi (wallet_topup, purchase) |
| reference_id | uuid | ID referensi |
| description | text | Deskripsi transaksi |
| processed_at | datetime | Waktu diproses |

#### wallet_topup
Transaksi pengisian saldo wallet.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| wallet_id | uuid | FK ke wallet |
| organization_id | uuid | FK ke organization (denormalized) |
| topped_up_by_guardian_id | uuid | FK ke guardian (untuk school) |
| topped_up_by_member_id | uuid | FK ke member (untuk industry self topup) |
| topped_up_by_staff_id | uuid | FK ke staff (admin manual topup) |
| topup_code | string | Kode unik (TU20251117001) |
| amount | decimal | Jumlah top-up |
| payment_method | enum | Metode: bank_transfer, ewallet, qris, cash, payroll_deduction, bulk_transfer |
| payment_reference | string | Referensi dari payment gateway |
| status | enum | Status: pending, completed, failed, expired |
| payment_metadata | json | Response gateway, URL receipt |
| processed_at | datetime | Waktu diproses |

#### spending_limit
Pengaturan batas pengeluaran wallet.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| wallet_id | uuid | FK ke wallet |
| set_by_type | enum | Siapa yang set: guardian, member, staff |
| limit_name | string | Nama limit (Uang Jajan Harian, Limit Weekend) |
| daily_limit | decimal | Batas harian (NULL = tanpa batas) |
| per_transaction_limit | decimal | Batas per transaksi |
| allowed_start_time | time | Waktu mulai diizinkan (NULL = 00:00) |
| allowed_end_time | time | Waktu akhir diizinkan (NULL = 23:59) |
| effective_from | date | Tanggal mulai berlaku |
| effective_until | date | Tanggal berakhir (NULL = tanpa batas) |
| apply_on | enum | Berlaku pada: all_days, weekdays, weekends, specific_dates |
| specific_dates | json | Tanggal spesifik [2025-12-25, 2025-12-31] |
| priority | int | Prioritas (angka lebih tinggi = prioritas lebih tinggi) |
| allowed_categories | json | Kategori yang diizinkan [makanan, minuman] |
| blocked_categories | json | Kategori yang diblokir [permen, soda] |
| is_active | boolean | Status aktif limit |

---

### Transaction Management

#### menu_item
Produk yang dijual oleh tenant.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| tenant_id | uuid | FK ke tenant |
| name | string | Nama produk (Nasi Goreng, Coffee) |
| sku | string | Stock Keeping Unit |
| description | text | Deskripsi produk |
| price | decimal | Harga jual |
| category | string | Kategori: food, beverage, snack, stationery, merchandise |
| image_url | string | URL gambar produk |
| is_available | boolean | Status ketersediaan |
| stock_quantity | int | Jumlah stok (0 = unlimited jika track_stock=false) |
| track_stock | boolean | Lacak inventori atau tidak |
| low_stock_threshold | int | Threshold alert stok rendah |

#### purchase
Transaksi pembelian.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| member_id | uuid | FK ke member (denormalized) |
| wallet_id | uuid | FK ke wallet |
| organization_id | uuid | FK ke organization (denormalized) |
| tenant_id | uuid | FK ke tenant |
| device_id | uuid | FK ke device (NULL jika manual) |
| cashier_id | uuid | FK ke staff |
| purchase_code | string | Kode unik (PU20251117001) |
| total_amount | decimal | Total sebelum diskon |
| discount_amount | decimal | Total diskon |
| final_amount | decimal | Jumlah yang dibayar |
| status | enum | Status: completed, failed |
| payment_method | enum | Metode: wallet, cash, mixed |
| purchase_time | datetime | Waktu transaksi |
| verification_data | json | Data verifikasi (fingerprint quality, device info) |
| notes | text | Catatan kasir |

#### purchase_item
Item dalam transaksi pembelian.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| purchase_id | uuid | FK ke purchase |
| menu_item_id | uuid | FK ke menu_item |
| item_name | string | Snapshot nama produk saat beli |
| item_sku | string | Snapshot SKU |
| item_price | decimal | Snapshot harga saat beli |
| quantity | int | Jumlah item |
| subtotal | decimal | item_price * quantity |
| discount_amount | decimal | Diskon per item |

---

### Device & Biometric

#### device
Perangkat yang terhubung ke sistem.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| organization_id | uuid | FK ke organization |
| tenant_id | uuid | FK ke tenant (NULL = device level org) |
| device_code | string | Kode device (DEV001) |
| device_serial | string | Serial number |
| device_type | enum | Tipe: fingerprint_scanner, pos_terminal, tablet, kiosk |
| ip_address | string | IP address |
| mac_address | string | MAC address |
| status | enum | Status: active, inactive, maintenance, offline |
| last_heartbeat_at | datetime | Waktu heartbeat terakhir |
| device_info | json | Info: OS, app version, hardware specs |

#### fingerprint
Template biometrik sidik jari member.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| member_id | uuid | FK ke member |
| finger_position | enum | Posisi jari: thumb_left, thumb_right, index_left, index_right, middle_left, middle_right |
| encrypted_template | blob | Template biometrik terenkripsi |
| template_hash | string | Hash untuk lookup cepat |
| quality_score | float | Skor kualitas (0-100) |
| enrolled_at | datetime | Waktu pendaftaran |
| last_verified_at | datetime | Waktu terakhir digunakan |
| is_active | boolean | Status aktif |
| failed_attempts | int | Jumlah percobaan gagal |
| locked_until | datetime | Terkunci sampai (auto unlock) |

---

### Notification & Communication

#### notification
Notifikasi ke pengguna.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| recipient_type | enum | Tipe penerima: guardian, member, staff |
| recipient_id | uuid | ID penerima (polymorphic) |
| organization_id | uuid | FK ke organization (denormalized) |
| title | string | Judul (Low Balance, Purchase Success) |
| message | text | Isi pesan |
| type | enum | Tipe: topup, purchase, low_balance, alert, limit_reached, refund, payroll |
| channel | enum | Channel: in_app, email, sms, push |
| status | enum | Status: pending, sent, failed, read |
| is_read | boolean | Sudah dibaca |
| sent_at | datetime | Waktu dikirim |
| read_at | datetime | Waktu dibaca |
| retry_count | int | Jumlah retry untuk pengiriman gagal |
| error_message | text | Pesan error |
| data | json | Data tambahan |

---

### Audit & Compliance

#### audit_log
Log aktivitas untuk audit trail.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| organization_id | uuid | FK ke organization (NULL = system-wide) |
| auditable_type | string | Tipe entitas (wallet, member, purchase) |
| auditable_id | uuid | ID entitas |
| event | string | Event: created, updated, deleted, balance_changed, login |
| actor_type | string | Tipe aktor: user, staff, guardian, member, system |
| actor_id | uuid | ID aktor (NULL untuk system) |
| old_values | json | Nilai sebelum perubahan |
| new_values | json | Nilai setelah perubahan |
| ip_address | string | IP address aktor |
| user_agent | string | User agent browser/app |
| description | text | Deskripsi aktivitas |

---

### RBAC (Role-Based Access Control)

#### role
Definisi role dalam sistem.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| organization_id | uuid | FK ke organization (NULL = global role) |
| name | string | Nama role (super_admin, org_admin, cashier) |
| slug | string | Slug (unique per org) |
| guard_name | string | Guard: web, api |
| description | text | Deskripsi role |
| is_system | boolean | Role sistem (tidak bisa dihapus) |

#### permission
Definisi permission dalam sistem.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| id | uuid | Primary key |
| organization_id | uuid | FK ke organization (NULL = global permission) |
| name | string | Nama permission (manage_wallets, create_topup) |
| slug | string | Slug (unique per org) |
| guard_name | string | Guard: web, api |
| group | string | Grup: wallet, transaction, user, report |
| description | text | Deskripsi permission |
| is_system | boolean | Permission sistem (tidak bisa dihapus) |

#### role_has_permission
Relasi role ke permission.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| role_id | uuid | FK ke role |
| permission_id | uuid | FK ke permission |

#### model_has_role
Relasi user ke role.

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| role_id | uuid | FK ke role |
| model_id | uuid | FK ke user |
| model_type | string | Tipe model (App\Models\User) |

#### model_has_permission
Relasi user ke permission (direct assignment).

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| permission_id | uuid | FK ke permission |
| model_id | uuid | FK ke user |
| model_type | string | Tipe model (App\Models\User) |

---

## Enum Values

### subscription_tier
- `trial` - Masa percobaan
- `basic` - Paket dasar
- `premium` - Paket premium
- `enterprise` - Paket enterprise

### sector_type
- `school` - Sekolah
- `industry` - Industri/Pabrik
- `corporate` - Korporasi/Kantor
- `retail` - Retail/Toko
- `government` - Pemerintah

### member_type
- `student` - Siswa (school)
- `employee` - Karyawan (corporate/government)
- `worker` - Pekerja (industry)
- `customer` - Pelanggan (retail)

### gender
- `male` - Laki-laki
- `female` - Perempuan
- `other` - Lainnya

### relationship (member_guardian)
- `father` - Ayah
- `mother` - Ibu
- `grandfather` - Kakek
- `grandmother` - Nenek
- `sibling` - Saudara
- `uncle` - Paman
- `aunt` - Bibi
- `guardian` - Wali
- `emergency_contact` - Kontak darurat
- `other` - Lainnya

### wallet_type
- `main` - Wallet utama
- `savings` - Tabungan
- `meal_allowance` - Tunjangan makan
- `transport` - Transportasi

### entry_type (wallet_history)
- `credit` - Uang masuk
- `debit` - Uang keluar

### transaction_type (wallet_history)
- `topup` - Pengisian saldo
- `purchase` - Pembelian
- `refund` - Pengembalian dana
- `adjustment` - Penyesuaian manual
- `transfer_in` - Transfer masuk
- `transfer_out` - Transfer keluar

### payment_method (wallet_topup)
- `bank_transfer` - Transfer bank
- `ewallet` - E-wallet (GoPay, OVO, Dana)
- `qris` - QRIS
- `cash` - Tunai
- `payroll_deduction` - Potongan gaji
- `bulk_transfer` - Transfer bulk

### topup_status
- `pending` - Menunggu pembayaran
- `completed` - Selesai
- `failed` - Gagal
- `expired` - Kadaluarsa

### set_by_type (spending_limit)
- `guardian` - Diset oleh guardian
- `member` - Diset oleh member sendiri
- `staff` - Diset oleh staff/admin

### apply_on (spending_limit)
- `all_days` - Semua hari
- `weekdays` - Hari kerja (Senin-Jumat)
- `weekends` - Akhir pekan (Sabtu-Minggu)
- `specific_dates` - Tanggal tertentu

### device_type
- `fingerprint_scanner` - Scanner sidik jari
- `pos_terminal` - Terminal POS
- `tablet` - Tablet
- `kiosk` - Kiosk self-service

### device_status
- `active` - Aktif
- `inactive` - Tidak aktif
- `maintenance` - Dalam pemeliharaan
- `offline` - Offline

### finger_position
- `thumb_left` - Jempol kiri
- `thumb_right` - Jempol kanan
- `index_left` - Telunjuk kiri
- `index_right` - Telunjuk kanan
- `middle_left` - Jari tengah kiri
- `middle_right` - Jari tengah kanan

### notification_type
- `topup` - Notifikasi top-up
- `purchase` - Notifikasi pembelian
- `low_balance` - Saldo rendah
- `alert` - Peringatan umum
- `limit_reached` - Batas tercapai
- `refund` - Pengembalian dana
- `payroll` - Terkait gaji

### notification_channel
- `in_app` - Dalam aplikasi
- `email` - Email
- `sms` - SMS
- `push` - Push notification

### notification_status
- `pending` - Menunggu dikirim
- `sent` - Sudah dikirim
- `failed` - Gagal dikirim
- `read` - Sudah dibaca

### purchase_status
- `completed` - Selesai
- `failed` - Gagal

### payment_method (purchase)
- `wallet` - Pembayaran via wallet
- `cash` - Tunai
- `mixed` - Kombinasi wallet + cash

---

## Istilah Teknis

### Soft Delete
Penghapusan data secara logis dengan mengisi field `deleted_at`. Data tidak benar-benar dihapus dari database.

### Denormalization
Duplikasi data untuk optimasi query. Contoh: `organization_id` di tabel `purchase` untuk mempercepat filtering tanpa JOIN.

### Polymorphic Relationship
Relasi dimana satu field bisa mereferensi ke beberapa tabel berbeda. Contoh: `notification.recipient_id` bisa merujuk ke guardian, member, atau staff.

### Idempotency
Properti dimana operasi yang sama dapat dijalankan berulang kali dengan hasil yang sama. Contoh: `purchase_code` dan `topup_code` unique untuk mencegah duplikasi.

### Template Hash
Hash dari template biometrik untuk lookup cepat tanpa decrypt seluruh template.

### Heartbeat
Sinyal periodik dari device untuk menunjukkan status online.

---

## Kode Format

### purchase_code
Format: `PU[YYYYMMDD][SEQUENCE]`
Contoh: `PU20251117001`

### topup_code
Format: `TU[YYYYMMDD][SEQUENCE]`
Contoh: `TU20251117001`

### device_code
Format: `DEV[SEQUENCE]`
Contoh: `DEV001`

---

## Audit Trail Fields

Hampir semua tabel memiliki field audit:
- `created_at` - Waktu pembuatan
- `updated_at` - Waktu update terakhir
- `deleted_at` - Waktu soft delete
- `created_by` - User yang membuat
- `updated_by` - User yang update terakhir
- `deleted_by` - User yang menghapus
