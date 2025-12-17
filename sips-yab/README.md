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

> Jika gambar diagramnya tidak muncul, silakan _refresh_ halamannnya.

## Alur Pembuatan Akun Pendaftaran (Siswa)

Berikut adalah alur pendaftaran sistem sebagai Pendaftar Baru (Siswa):

```mermaid
flowchart TD
    Start([Mulai: Buka Website]) --> Choice1{Pilih Menu}

    %% Cabang Pilihan
    Choice1 -- Alumni --> AlumniFlow[Menu Alumni]
    Choice1 -- Pendaftar Baru --> Choice2{Daftar Sebagai?}

    %% Cabang Tipe Akun
    Choice2 -- Orang Tua --> ParentFlow[Menu Orang Tua]
    Choice2 -- Siswa --> FormInput[Isi Formulir Siswa]

    %% Proses Validasi
    FormInput --> Validate{Sistem Mengecek Data}
    Validate -- Data Kurang/Salah --> ShowError[Muncul Pesan Error]
    ShowError --> FormInput

    %% OTP Flow
    Validate -- Data Lengkap --> GenOTP[Sistem Kirim Kode WA]
    GenOTP --> InputOTP[Masukkan Kode WA]
    InputOTP --> VerifyOTP{Kode Benar?}

    VerifyOTP -- Salah --> InputOTP
    VerifyOTP -- Benar --> Active[Akun Aktif]

    %% Auto Login & Middleware
    Active --> AutoLogin[Masuk ke Aplikasi]
    AutoLogin --> CheckFamily{Data Keluarga?}

    %% Logic: Cek Data Keluarga (Wajib)
    CheckFamily -- Belum Ada --> InputFamily[Wajib Isi: Data KK & Ortu]
    InputFamily --> CheckFamily

    CheckFamily -- Sudah Lengkap --> Dashboard([Halaman Utama])

    %% --- BAGIAN STYLING ---
    classDef kuning fill:#ffd700,stroke:#333,stroke-width:2px,color:black;
    classDef biru fill:#87ceeb,stroke:#333,stroke-width:2px,color:black;
    classDef merah fill:#ff6b6b,stroke:#333,stroke-width:2px,color:black;
    classDef hijau fill:#90ee90,stroke:#333,stroke-width:2px,color:black;
    classDef ungu fill:#e6e6fa,stroke:#333,stroke-width:2px,color:black;

    class Choice1,Choice2,Validate,VerifyOTP,CheckFamily kuning;
    class Start,AlumniFlow,ParentFlow ungu;
    class Active,Dashboard hijau;
    class ShowError merah;
    class FormInput,GenOTP,InputOTP,AutoLogin,InputFamily biru;
```

## Alur Pembuatan Akun Pendaftaran (Orang Tua)

Berikut adalah alur pendaftaran sistem sebagai Pendaftar Baru (Orang Tua):

```mermaid
flowchart TD
    Start([Mulai: Buka Website]) --> Choice1{Pilih Menu}

    %% Cabang Pilihan
    Choice1 -- Alumni --> AlumniFlow[Menu Alumni]
    Choice1 -- Pendaftar Baru --> Choice2{Daftar Sebagai?}

    %% Cabang Tipe Akun
    Choice2 -- Siswa --> StudentFlow[Menu Siswa]
    Choice2 -- Orang Tua --> FormInput[Isi Biodata Orang Tua]

    %% Proses Validasi
    FormInput --> Validate{Sistem Mengecek Data}
    Validate -- Data Kurang/Salah --> ShowError[Muncul Pesan Error]
    ShowError --> FormInput

    %% OTP Flow
    Validate -- Data Lengkap --> GenOTP[Sistem Kirim Kode WA]
    GenOTP --> InputOTP[Masukkan Kode WA]
    InputOTP --> VerifyOTP{Kode Benar?}

    VerifyOTP -- Salah --> InputOTP
    VerifyOTP -- Benar --> Active[Akun Aktif]

    %% Auto Login & Middleware
    Active --> AutoLogin[Masuk ke Aplikasi]
    AutoLogin --> CheckFamily{Cek Data KK}

    %% Logic 1: Cek Kepala Keluarga (Wajib)
    CheckFamily -- Belum Ada --> InputFamily[Wajib Isi: Data KK & Ortu]
    InputFamily --> CheckFamily

    %% Logic 2: Cek Data Anak
    CheckFamily -- Sudah Ada --> CheckChild{Sudah Ada Data Anak?}

    CheckChild -- Belum Ada --> EmptyState[Tampilan Awal]
    EmptyState --> AddChild[Klik Tombol: Tambah Anak]
    AddChild --> CheckChild

    CheckChild -- Sudah Ada --> Dashboard([Pendaftaran Siap Dimulai])

    %% --- BAGIAN STYLING ---
    classDef kuning fill:#ffd700,stroke:#333,stroke-width:2px,color:black;
    classDef biru fill:#87ceeb,stroke:#333,stroke-width:2px,color:black;
    classDef merah fill:#ff6b6b,stroke:#333,stroke-width:2px,color:black;
    classDef hijau fill:#90ee90,stroke:#333,stroke-width:2px,color:black;
    classDef ungu fill:#e6e6fa,stroke:#333,stroke-width:2px,color:black;

    class Choice1,Choice2,Validate,VerifyOTP,CheckFamily,CheckChild kuning;
    class Start,AlumniFlow,StudentFlow ungu;
    class Active,Dashboard hijau;
    class ShowError merah;
    class FormInput,GenOTP,InputOTP,AutoLogin,InputFamily,EmptyState,AddChild biru;
```

?> **Keterangan Warna Diagram:**
<span class="hint yellow">Kuning</span> = Pilihan/Logika Sistem
<span class="hint blue">Biru</span> = Aktivitas Anda
<span class="hint green">Hijau</span> = Berhasil
<span class="hint red">Merah</span> = Error/Masalah

## Tahapan Pendaftaran

<div class="timeline-container">
  <div class="step-item">
    <h3>1️⃣ Data Siswa & Orang Tua</h3>
    <p>Lengkapi formulir pendaftaran dengan data diri calon siswa dan orang tua secara akurat. Pastikan semua kolom terisi dengan benar (sesuai KK/Akte).</p>
  </div>

  <div class="step-item">
    <h3>2️⃣ Persetujuan & Komitmen</h3>
    <p>Baca dan setujui syarat, ketentuan, dan surat pernyataan komitmen (Tata Tertib & Biaya) sebagai bagian legalitas proses pendaftaran.</p>
  </div>

  <div class="step-item">
    <h3>3️⃣ Pembayaran Administrasi</h3>
    <p>Selesaikan pembayaran biaya pendaftaran melalui Virtual Account atau Transfer Bank yang tersedia untuk membuka akses ke tahap finalisasi.</p>
  </div>

  <div class="step-item">
    <h3>4️⃣ Finalisasi Data</h3>
    <p>Periksa kembali seluruh data. <b>Peringatan:</b> Setelah tombol Finalisasi ditekan, data akan dikunci dan tidak dapat diubah lagi.</p>
  </div>
</div>

---

<h2>Butuh Bantuan?</h2>

Jika Anda mengalami kendala saat pendaftaran (misal: Kode WA tidak masuk atau gagal upload berkas), silakan hubungi tim Support kami:

- **WhatsApp Helpdesk:** [0812-XXXX-XXXX](https://wa.me/62812xxxx) (Jam Kerja 08.00 - 15.00)
- **Email:** [sips.almasoem.com](https://sips.almasoem.com/)
- **Panduan Teknis:** [Klik di sini untuk melihat FAQ](faq.md)
