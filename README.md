
# Sistem Manajemen Terintegrasi Pelaporan Kepatuhan ISO 27001 & 27701

## Deskripsi Umum
Aplikasi ini berfungsi sebagai **platform terintegrasi untuk pengelolaan, pelaporan, dan pemantauan kepatuhan terhadap standar ISO/IEC 27001 (Keamanan Informasi)** dan **ISO/IEC 27701 (Privasi Informasi)**.  
Tujuannya adalah untuk mempermudah satuan kerja dalam:
- Melakukan **self-assessment** terhadap kontrol ISO,
- Mengelola bukti kepatuhan secara digital,
- Melakukan **verifikasi dan audit internal**,
- Menyusun laporan kepatuhan otomatis untuk kebutuhan audit eksternal.

---

## Arsitektur Sistem

###  Komponen Utama
1. **Frontend (Web / Mobile)**  
   - Framework: Flutter 
   - Fitur utama: form checklist, upload bukti, dashboard progress, verifikasi auditor.

2. **Backend & Database**
   - Menggunakan **Supabase PostgreSQL **  
   - API bawaan Supabase digunakan untuk CRUD checklist, session, dan evidence.  
   - Dilengkapi dengan **Row Level Security (RLS)** untuk autentikasi per pengguna.

3. **Storage**
   - Supabase Storage untuk file evidence (dokumen, screenshot, laporan audit, kebijakan, dsb).

4. **Authentication**
   - Menggunakan dengan integrasi email/password.

---

##  Struktur Modul Utama

###  **Checklist Management**
Menangani daftar pertanyaan audit / self-assessment.

| Entitas | Deskripsi |
|----------|------------|
| `checklist_items` | Menyimpan master pertanyaan berdasarkan ISO 27001 & 27701 |
| `checklist_response` | Menyimpan jawaban user, link evidence, verifikasi |
| `checklist_sessions` | Menyimpan periode atau konteks penilaian (mis. Q1 2025 - Unit Surabaya) |

#### Alur Umum
1. Admin menambahkan atau mengimpor checklist ISO (dari CSV).  
2. User memilih periode penilaian → sistem memuat daftar pertanyaan.  
3. User mengisi evidence dan status kepatuhan.  
4. Auditor memverifikasi hasil penilaian.  
5. Dashboard menampilkan tingkat kepatuhan keseluruhan.

---

### **Evidence Management**
- User dapat mengunggah atau menautkan file (Drive, internal link).  
- Data disimpan dalam `evidence_link` pada tabel `checklist_response`.  
- Bukti dapat diakses auditor untuk verifikasi.

---

###  **Audit & Verifikasi**
- Auditor dapat memberi tanda ✔️ pada kolom verifikasi.  
- Auditor dapat menambahkan catatan temuan (`notes`) dan status:
  - **Compliant / Approved**
  - **Nonconformity**
  - **Pending**

---

###  **Pelaporan & Dashboard**
- Menampilkan grafik tingkat kepatuhan:
  - Persentase kontrol yang “Terverifikasi”
  - Breakdown per kategori (Tata Kelola, Teknis, SDM, Privasi)
  - Status tindak lanjut per kontrol
- Ekspor laporan ke CSV / PDF untuk audit eksternal.

---

##  Desain Skema Database (Supabase)

### **Tabel: checklist_items**
```sql
create table public.checklist_items (
  id bigint generated always as identity primary key,
  iso_control text,
  category text,
  question text not null,
  expected_evidence text,
  action_guidance text,
  framework_id int not null default 1,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create table public.checklist_sessions (
  id bigint generated always as identity primary key,
  title text not null,
  organization text,
  start_date date default now(),
  end_date date,
  created_by uuid references auth.users(id),
  created_at timestamptz default now()
);

create table public.checklist_response (
  id bigint generated always as identity primary key,
  checklist_item_id bigint references public.checklist_items(id) on delete cascade,
  session_id bigint references public.checklist_sessions(id) on delete cascade,
  user_id uuid references auth.users(id),
  evidence_link text,
  verification boolean default false,
  notes text,
  status text default 'Pending',
  created_at timestamptz default now(),
  updated_at timestamptz default now()
)
```


                                       
