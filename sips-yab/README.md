# Overview SIPS YAB

> Sistem Informasi Penerimaan Siswa Yayasan Al Ma'soem Bandung.

Sistem ini dirancang untuk memfasilitasi proses Pendaftaran Calon Siswa/i Al Ma'soem secara online, mulai dari pendaftaran akun, verifikasi berkas, hingga pengumuman kelulusan.

---

## Fitur Utama

Berikut adalah kapabilitas utama sistem saat ini:

- **Front-Office (Siswa dan Orang Tua/Wali):**
  - [x] Registrasi Akun & Login
  - [x] Input Biodata & Upload Dokumen (KK, Akta, Ijazah)
  - [x] Metode Pembayaran Terintegrasi
  - [x] Cetak Kartu Bukti Pendaftaran (PDF)
- **Back-Office (Admin, Panitia dan Stakeholders):**
  - [x] Dashboard Statistik Real-time
  - [x] Verifikasi Dokumen (Approve/Reject)
  - [x] Export Data ke Excel

## Alur Pendaftaran Siswa

Berikut adalah alur pendaftaran sistem sebagai Pendaftar Baru (Siswa):

```mermaid
flowchart TD
    Start([Mulai: Landing Page]) --> Choice1{Pilih Pendaftar}

    %% Cabang Pilihan
    Choice1 -- Pendaftar Baru --> Choice2{Tipe Akun?}
    Choice1 -- Alumni --> AlumniFlow[Alur alumni]

    %% Cabang Tipe Akun
    Choice2 -- Siswa --> FormInput[Isi Form Registrasi]
    Choice2 -- Orang Tua --> ParentFlow[Alur Orang Tua]

    %% Proses Validasi
    FormInput --> Validate{Validasi Server}
    Validate -- Error/Invalid --> ShowError[Tampil Pesan Error]
    ShowError --> FormInput

    %% OTP Flow
    Validate -- Valid --> GenOTP[Generate & Kirim OTP]
    GenOTP --> InputOTP[User Input OTP]
    InputOTP --> VerifyOTP{Verifikasi OTP?}

    VerifyOTP -- Salah/Expired --> InputOTP
    VerifyOTP -- Benar --> Active[Set Akun Aktif]

    %% Auto Login & Middleware
    Active --> AutoLogin[Auto-Login User]
    AutoLogin --> CheckFamily{Cek Data Keluarga?}

    %% Logic Force Redirect
    CheckFamily -- Kosong (NULL) --> InputFamily[Form Wajib: KK & Ortu]
    InputFamily --> CheckFamily

    CheckFamily -- Ada Data --> Dashboard([Masuk Dashboard])

    %% Styling (Biar cantik warnanya)
    classDef decision fill:#f9f,stroke:#333,stroke-width:2px;
    classDef terminal fill:#ccf,stroke:#333,stroke-width:2px;
    classDef success fill:#cfc,stroke:#333,stroke-width:2px;

    class Choice1,Choice2,Validate,VerifyOTP,CheckFamily decision;
    class Start,AlumniFlow,ParentFlow terminal;
    class Dashboard,Active success;
