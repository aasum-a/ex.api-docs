# Tahapan Pendaftara melalui Admin

Modul ini digunakan oleh Admin untuk menginput atau mengedit data pendaftaran siswa step-by-step. Terdiri dari 4 Tahapan (Wizard).

> **⚠️ NOTE:** Controller ini menggunakan pola **Static Method Dispatcher**.
> Logic setiap step dipisah ke dalam fungsi static (`data_step_1`, `store_step_1`, dst).

## Show Form Step

Menampilkan formulir pendaftaran berdasarkan nomor tahapan.

- **URL**: `GET /admin/pendaftaran/calon-siswa/data-calon-siswa/tahap-pendaftaran/step/{stepNumber}/{id_calon_siswa}`
- **Auth**: Session Cookie

### Path Parameters

| Param            | Tipe | Keterangan                                                                |
| :--------------- | :--- | :------------------------------------------------------------------------ |
| `stepNumber`     | Int  | `1` (Formulir), `2` (Persetujuan), `3` (Pembayaran), `4` (Biodata/Final). |
| `id_calon_siswa` | UUID | ID Siswa yang sedang diedit.                                              |

### Logic Otorisasi

Sistem akan mengecek `ProgresPendaftaran`. Admin **TIDAK BISA** melompat ke Step 3 jika Step 1 & 2 belum berstatus 'Selesai' (Flag `1` atau `2`). Akan di-redirect paksa ke step terakhir yang valid.

---

## Store Data

Menyimpan data pendaftaran. Endpoint ini bersifat dinamis, logic penyimpanan berbeda tergantung `stepNumber`.

- **URL**: `POST /admin/pendaftaran/calon-siswa/data-calon-siswa/tahap-pendaftaran/step/{stepNumber}/{id_calon_siswa}`

### Step 1: Formulir Data Diri

Validasi sangat ketat untuk NIK, NISN, dan Email (Unique Check).

- **Body**: `nama_siswa`, `nik`, `no_kk`, `email`, `id_unit`, `id_program_jurusan`, dll.
- **Side Effect**: Otomatis membuat/mengupdate data `OrangTua` dan `User` login untuk orang tua.

### Step 2: Persetujuan

Hanya checklist persetujuan.

- **Body**: `setuju_perjanjian` (Boolean).
- **Side Effect**: Mengirim Notifikasi WA & Email ke Admin/User bahwa perjanjian disetujui.

### Step 3: Pembayaran & Bukti

- **Body**:
  - `type`: `pengajuan_beasiswa` (Jika upload berkas beasiswa) atau `next_step` (Jika skip/lanjut).
  - `id_dokumen_fr_X`: File bukti (PDF/Img) untuk beasiswa tertentu.
- **Logic Unik**: File langsung diupload ke **Google Drive** secara _Synchronous_ (User harus menunggu proses upload selesai).

### Step 4: Finalisasi & Dokumen Pendukung

Upload berkas wajib (KK, Akta, Ijazah).

- **Body**: `file_foto`, `file_ijazah`, `file_kartu_keluarga`, dll.
- **Validasi**: File Foto wajib ada. Dokumen lain opsional tergantung konfigurasi.
