# Pilih Sesi Data & Manajemen Anak

Modul ini menangani autentikasi sekunder (Verifikasi KK/OTP) dan manajemen sesi anak (memilih anak mana yang sedang dikelola, menambah anak baru, atau mendaftar ke jenjang berikutnya).

> **⚠️ SECURITY AUDIT FINDINGS**
>
> 1.  **High Risk IDOR (`pilihSesiDataJenjang`):** Logic fallback ke `CalonSiswa::find($id)` memungkinkan user mengakses data siswa lain hanya dengan menebak ID, tanpa validasi kepemilikan.
> 2.  **Improper Verb (`pilihSesiDataJenjangStore`):** Route terdaftar sebagai `GET`, padahal logic-nya mengubah state (menyimpan session). Ini rentan serangan CSRF dan _State-Changing via Link_.
> 3.  **Session Poisoning:** Validasi kepemilikan anak saat switch sesi (Store) masih lemah di beberapa titik.

---

## 1. Flow Verifikasi Orang Tua (Pre-Pilih)

Sebelum masuk dashboard, sistem memvalidasi data orang tua via NIK/KK dan OTP WhatsApp.

### A. Cek Status Verifikasi (Page)

- **URL**: `GET /dashboard/sesi/pre-pilih-sesi-data`
- **Auth**: Session Cookie
- **Fungsi**: Mengecek apakah user sudah punya data Orang Tua & KK yang valid. Jika sudah, redirect ke halaman pilih sesi.

### B. Request OTP (Store Data Awal)

Menyimpan data awal Orang Tua (KK, Nama) dan mengirimkan OTP ke WhatsApp terdaftar.

- **URL**: `POST /dashboard/sesi/pre-pilih-sesi-data-store`
- **Auth**: Session Cookie

#### Parameters

| Param       | Tipe    | Wajib  | Keterangan                     |
| :---------- | :------ | :----- | :----------------------------- |
| `no_kk`     | Numeric | Ya     | 16 Digit Nomor KK.             |
| `nama_ayah` | String  | _Cond_ | Wajib jika `nama_ibu` kosong.  |
| `nama_ibu`  | String  | _Cond_ | Wajib jika `nama_ayah` kosong. |
| `nama_wali` | String  | _Cond_ | Wajib jika ayah/ibu kosong.    |

### C. Verifikasi / Resend OTP

Memvalidasi kode OTP yang diinput user atau meminta kirim ulang.

- **URL**: `POST /dashboard/sesi/pre-pilih-sesi-data-otp`
- **Auth**: Session Cookie

#### Parameters

| Param                  | Tipe   | Wajib  | Keterangan                                      |
| :--------------------- | :----- | :----- | :---------------------------------------------- |
| `token`                | String | Ya     | Token hash dari URL/Session.                    |
| `method_send`          | String | Ya     | `otp` (Verify) atau `otp-resend` (Kirim Ulang). |
| `otp[]`                | Array  | _Cond_ | Array 4 digit (jika method `otp`).              |
| `g-recaptcha-response` | String | Ya     | Google Captcha.                                 |

---

## 2. Flow Pilih Sesi (Switch Context)

Memilih anak yang aktif untuk dikelola datanya.

### A. List Data Anak

Menampilkan daftar anak yang terhubung dengan akun Orang Tua (berdasarkan NIK/KK).

- **URL**: `GET /dashboard/sesi/pilih-sesi-data`
- **Auth**: Session Cookie

### B. Set Session Anak (Switch)

Mengaktifkan sesi anak tertentu berdasarkan parameter.

> **⚠️ SECURITY WARNING (IDOR):**
> Endpoint ini menerima parameter `{calon_siswa}`. Logic backend mencoba mendekripsi. Jika gagal, backend melakukan `CalonSiswa::find($raw_id)`.
> **Celah:** Attacker bisa memasukkan ID UUID raw milik siswa lain. Code tidak memvalidasi apakah siswa tersebut benar-benar anak dari User yang login (Parent ID Mismatch).

- **URL**: `GET /dashboard/sesi/pilih-sesi-data-jenjang/{calon_siswa}`
- **Auth**: Session Cookie

#### Parameters

| Param         | Tipe   | Wajib | Keterangan                                                |
| :------------ | :----- | :---- | :-------------------------------------------------------- |
| `calon_siswa` | String | Ya    | Encrypted NIK **ATAU** Raw UUID Calon Siswa (Vulnerable). |

### C. Simpan Session (Finalize Switch)

Menyimpan data anak yang dipilih ke dalam PHP Session (`data_anak`) dan redirect ke Dashboard utama.

> **⚠️ SECURITY WARNING (Improper Verb):**
> Route ini terdaftar sebagai `GET|HEAD` tapi melakukan aksi simpan session. Seharusnya method `POST` untuk mencegah CSRF.

- **URL**: `GET /dashboard/sesi/pilih-sesi-data-jenjang-store`
- **Auth**: Session Cookie

#### Parameters

| Param            | Tipe | Wajib | Keterangan                           |
| :--------------- | :--- | :---- | :----------------------------------- |
| `id_calon_siswa` | UUID | Ya    | ID Calon Siswa yang akan diaktifkan. |

---

## 3. Flow Tambah Data (New/Existing Student)

### A. Tambah Anak Baru (Adik)

Mendaftarkan siswa baru (belum pernah sekolah di sini).

- **URL**: `POST /dashboard/sesi/tambah-pilih-sesi-data-store`
- **Auth**: Session Cookie

#### Parameters

| Param                   | Tipe   | Wajib  | Keterangan                             |
| :---------------------- | :----- | :----- | :------------------------------------- |
| `nama_lengkap`          | String | Ya     | Nama siswa.                            |
| `jenis_daftar`          | Int    | Ya     | 1 (Baru), 2 (Pindahan), 3 (Internal?). |
| `tanggal_lahir`         | Date   | Ya     | Format YYYY-MM-DD.                     |
| `jenis_kelamin`         | String | Ya     | `L` atau `P`.                          |
| `jenjang_sekolah`       | UUID   | Ya     | ID Unit (SD/SMP/SMA).                  |
| `jenis_kelas`           | UUID   | Ya     | ID Program Jurusan (Reguler).          |
| `jenis_program_sekolah` | Int    | Ya     | 1 (Fullday), 2 (Boarding).             |
| `jenis_kelas_boarding`  | UUID   | _Cond_ | Wajib jika program sekolah = 2.        |

### B. Tambah Jenjang (Anak Lama Lanjut Sekolah)

Mendaftarkan siswa yang sudah ada ke jenjang berikutnya (misal: SD -> SMP). Sistem akan menduplikasi data biodata lama.

- **URL**: `POST /dashboard/sesi/tambah-pilih-sesi-data-jenjang-store`
- **Auth**: Session Cookie

#### Parameters

| Param                 | Tipe   | Wajib  | Keterangan                                                                    |
| :-------------------- | :----- | :----- | :---------------------------------------------------------------------------- |
| `nik`                 | String | _Cond_ | Encrypted NIK (Untuk lookup data lama).                                       |
| `id_calon_siswa`      | UUID   | _Cond_ | Fallback jika NIK tidak ada (Rawan IDOR jika tidak divalidasi ownership-nya). |
| `jenjang_sekolah`     | UUID   | Ya     | ID Unit Baru.                                                                 |
| `jenis_kelas`         | UUID   | Ya     | ID Program Jurusan Baru.                                                      |
| `tingkat_pendaftaran` | Int    | Ya     | Tingkat kelas (misal: 7).                                                     |
