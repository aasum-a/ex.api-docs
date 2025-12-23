# Pengaturan Akun & Notifikasi

Modul ini menangani manajemen profil pengguna (Orang Tua), keamanan akun, serta pusat pemberitahuan (in-app notifications).

> **⚠️ SECURITY & LOGIC NOTES**
>
> 1.  **OTP Requirement:** Setiap perubahan data sensitif (Email, No HP, Password) mewajibkan verifikasi OTP WhatsApp sebelum perubahan diterapkan.
> 2.  **Temporary Storage:** Data perubahan disimpan sementara dalam kolom `data_scope` (terenkripsi) sampai OTP diverifikasi.
> 3.  **Data Sync:** Perubahan data akun Orang Tua akan otomatis menyinkronkan data kontak (Email/HP) ke semua data Anak (`calon_siswa`) yang terhubung.

---

## 1. Pengaturan Akun

Halaman dan logika untuk mengubah profil, password, dan avatar pengguna.

### A. Halaman Pengaturan

Menampilkan data diri pengguna saat ini.

- **URL**: `GET /dashboard/pengaturan-akun`
- **Auth**: Session Cookie
- **Controller**: `PengaturanController@pengaturanAkunIndex`

#### Data Response (View)

- `id_user`, `name`, `username`, `email`
- `phone_number`, `avatar` (URL Google Drive)
- `no_kk`, `alamat` (Data Orang Tua)

---

### B. Update Profil (Request OTP)

User mengirimkan data baru. Sistem tidak langsung mengupdate, melainkan menyimpan "draft" perubahan dan mengirim OTP.

- **URL**: `POST /dashboard/pengaturan-akun/pengaturan-akun-update`
- **Auth**: Session Cookie
- **Controller**: `PengaturanController@pengaturanAkunUpdate`

#### Parameters

| Param                   | Tipe   | Wajib  | Keterangan                      |
| :---------------------- | :----- | :----- | :------------------------------ |
| `name`                  | String | Ya     | Nama Lengkap.                   |
| `no_kk`                 | String | Ya     | 16 Digit Nomor KK.              |
| `email`                 | String | Tidak  | Email baru (Unique check).      |
| `phone_number`          | String | Tidak  | No WA baru (Unique check).      |
| `password`              | String | Tidak  | Password baru (Min 8 char).     |
| `password_confirmation` | String | _Cond_ | Wajib jika password diisi.      |
| `avatar`                | File   | Tidak  | Foto profil (JPG/PNG, Max 2MB). |

#### Logic Flow

1.  **Validasi:** Cek format dan duplikasi data.
2.  **Upload Avatar:** Jika ada file, langsung upload ke Google Drive dan update kolom `avatar` (Pengecualian: Avatar diupdate tanpa OTP).
3.  **Staging Data:**
    - Data lama (`old_data`) disimpan dalam `data_scope` terenkripsi.
    - Data baru di-set ke model User (tapi belum di-commit permanen sampai OTP, _koreksi: di code user langsung di-save tapi status verifikasi direset via OTP logic_).
4.  **OTP Generation:** Generate kode, simpan hash, kirim WA.
5.  **Redirect:** Ke halaman input OTP.

---

### C. Verifikasi Perubahan (Submit OTP)

Memvalidasi OTP untuk mengonfirmasi perubahan data.

- **URL**: `POST /dashboard/pengaturan-akun/pengaturan-akun-otp`
- **Auth**: Session Cookie
- **Controller**: `PengaturanController@pengaturanAkunOtpSend`

#### Parameters

| Param         | Tipe   | Wajib | Keterangan                                      |
| :------------ | :----- | :---- | :---------------------------------------------- |
| `token`       | String | Ya    | Token referensi OTP (Hash).                     |
| `otp[]`       | Array  | Ya    | 4 Digit Kode OTP.                               |
| `method_send` | String | Ya    | `otp` (Verify) atau `otp-resend` (Kirim Ulang). |

#### Logic Flow (Success)

1.  Validasi Hash OTP.
2.  **Komparasi Data:** Membandingkan `old_data` (dari `data_scope`) dengan data saat ini untuk mencatat log perubahan.
3.  **Notifikasi:** Mengirim pesan WhatsApp berisi ringkasan data apa saja yang berubah (Audit Trail).
4.  **Sync Data Anak:** Mengupdate `email` dan `phone_number` di tabel `calon_siswa` dan `data_lengkap` milik semua anak yang terkait.
5.  **Cleanup:** Menghapus token OTP dan `data_scope`.

---

## 2. Notifikasi (In-App)

Pusat notifikasi untuk pengguna (misal: Tagihan baru, Status pembayaran, Info sekolah).

### A. List Notifikasi

Menampilkan daftar notifikasi user (Pagination 15 item).

- **URL**: `GET /dashboard/notifikasi`
- **Auth**: Session Cookie
- **Controller**: `NotificationController@index`

---

### B. Tandai Dibaca (Mark as Read)

Menandai satu notifikasi spesifik sebagai "Sudah Dibaca" dan redirect ke URL aksi.

- **URL**: `GET /dashboard/notifikasi/mark-as-read/{id}`
- **Auth**: Session Cookie
- **Controller**: `NotificationController@markAsRead`

#### Parameters

| Param | Tipe | Wajib | Keterangan                             |
| :---- | :--- | :---- | :------------------------------------- |
| `id`  | UUID | Ya    | ID Notifikasi (tabel `notifications`). |

---

### C. Tandai Semua Dibaca (Mark All)

Menandai **semua** notifikasi yang belum dibaca menjadi "Sudah Dibaca".

- **URL**: `GET /dashboard/notifikasi/mark-all-as-read`
- **Auth**: Session Cookie
- **Controller**: `NotificationController@markAllAsRead`
