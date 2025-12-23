# Authentication

Modul ini adalah gerbang utama aplikasi SIPS. Menangani siklus hidup akun mulai dari pendaftaran, verifikasi (OTP), login (Hybrid JWT + Session), hingga pemulihan akses.

> **⚠️ ARCHITECTURE WARNING**
> Sistem ini menggunakan **Hybrid Authentication**.
>
> 1.  User login menggunakan kredensial biasa.
> 2.  Backend men-generate **JWT Access Token**.
> 3.  Token disimpan ke dalam **PHP Session** server-side (`session('access_token')`).
> 4.  Middleware mengecek keberadaan token di session untuk otorisasi.

---

## 1. Pendaftaran Akun (Registration)

User baru (Siswa/Orang Tua) mendaftar melalui form publik.

### A. Form Pendaftaran

Mengambil data referensi untuk dropdown di halaman pendaftaran.

- **URL**: `GET /pendaftaran`
- **Auth**: Guest
- **Controller**: `Main\PendaftaranController@pendaftaranIndex`

### B. Submit Pendaftaran

Endpoint krusial yang membuat User, Data Orang Tua, dan Data Calon Siswa (Transaksional).

> **⚠️ SECURITY RISK:** Password user disimpan sementara di kolom `data_scope` (terenkripsi) sambil menunggu verifikasi OTP.

- **URL**: `POST /pendaftaran-post`
- **Auth**: Guest
- **Controller**: `Main\PendaftaranController@pendaftaranPost`

#### Logic Flow

1.  **Validasi**: Cek duplikasi Email/HP (Query Manual).
2.  **User Creation**: Create user dengan status `sts_aktif = 1` (tapi butuh OTP).
3.  **Data Scope Stashing**: Password asli di-hash untuk login, tapi juga di-enkripsi (`Crypt`) dan disimpan di `data_scope` untuk keperluan auto-login pasca OTP.
4.  **OTP**: Generate OTP, simpan Hash-nya, kirim WA.

#### Parameters

| Param             | Tipe   | Wajib  | Keterangan                  |
| :---------------- | :----- | :----- | :-------------------------- |
| `role`            | String | Ya     | `siswa` atau `orang_tua`.   |
| `nama_lengkap`    | String | Ya     | Nama user.                  |
| `email`           | String | Tidak  | Email user (Unik).          |
| `phone_number`    | String | Ya     | No WA (Unik).               |
| `password`        | String | Ya     | Min 8 karakter.             |
| `jenis_daftar`    | Int    | _Cond_ | (Siswa) 1:Baru, 2:Pindahan. |
| `jenjang_sekolah` | UUID   | _Cond_ | (Siswa) ID Unit.            |
| `jenis_orang_tua` | String | _Cond_ | (Ortu) Ayah/Ibu/Wali.       |

---

## 2. Verifikasi OTP (2FA)

Sistem menggunakan OTP WhatsApp sebagai verifikasi utama. State OTP disimpan di tabel `users` (kolom `otp_token`, `data_scope`, `otp_token_expired_at`).

### A. Halaman Input OTP

- **URL**: `GET /otp`
- **Param**: `?token={hash_otp}` (Token referensi dari URL).

### B. Proses Verifikasi / Resend

Endpoint ini menangani dua aksi berdasarkan parameter `method_send`.

- **URL**: `POST /otp`
- **Controller**: `Auth\MainAuthController@otpSend`

#### Action: `otp-resend` (Kirim Ulang)

Men-generate ulang OTP baru dan mengirimkan via WhatsApp.

- **Logic**: Membaca password/data dari `data_scope`, membuat OTP baru, update DB.

#### Action: `otp` (Verifikasi)

Memvalidasi kode yang diinput user.

- **Logic**: `Hash::check(input_otp, database_hash)`.
- **Success**:
  1.  Clear OTP columns.
  2.  Set `phone_number_verified_at`.
  3.  Generate **JWT Token**.
  4.  Simpan Token di Session.
  5.  Redirect ke Dashboard.

#### Parameters

| Param         | Tipe   | Wajib  | Keterangan                         |
| :------------ | :----- | :----- | :--------------------------------- |
| `token`       | String | Ya     | Hash referensi user.               |
| `method_send` | String | Ya     | `otp` atau `otp-resend`.           |
| `otp[]`       | Array  | _Cond_ | Array 4 digit (jika method `otp`). |

---

## 3. Autentikasi (Login Flow)

### A. Proses Login

Menerima input multi-format (Username/Email/No HP) dan Password.

- **URL**: `POST /login-post`
- **Auth**: Guest
- **Controller**: `Auth\MainAuthController@loginPost`

#### Logic Checks

1.  **Credential Resolver**: Mendeteksi input:
    - Jika angka -> No HP (hapus strip `-`).
    - Jika format email -> Email.
    - Lainnya -> Username.
2.  **Role Check**: Hanya user dengan `level = 3` (Orang Tua/Siswa) yang boleh login via endpoint ini.
3.  **Verification Check**:
    - Jika `phone_number_verified_at` NULL -> Redirect paksa ke OTP.
    - Jika No HP kosong -> Redirect ke Flow "Add Phone" (Signed URL).
4.  **Token Generation**:
    - `JWTAuth::attempt($credentials)`.
    - Create `RefreshToken` di database (validitas sesuai config).
    - Simpan ke Session Laravel.

#### Parameters

| Param         | Tipe   | Wajib | Keterangan                               |
| :------------ | :----- | :---- | :--------------------------------------- |
| `username`    | String | Ya    | Username / Email / No HP.                |
| `password`    | String | Ya    | Password.                                |
| `remember_me` | String | Tidak | Value: `remember` (Extend expiry token). |

### B. Logout

Menghapus sesi dan token dari database.

- **URL**: `POST /dashboard/logout`
- **Auth**: Session Cookie

---

## 4. Pemulihan Akun (Forgot Password)

### A. Request Reset

Mengirim OTP ke WhatsApp berdasarkan nomor HP.

- **URL**: `POST /lupa-password`
- **Controller**: `Auth\MainAuthController@lupaPasswordPost`

> **Note:** Logic di sini membuat OTP Hash baru dan menyimpannya di `otp_token`. `data_scope` digunakan untuk menyimpan payload OTP terenkripsi (redundant).

#### Parameters

| Param          | Tipe   | Wajib | Keterangan          |
| :------------- | :----- | :---- | :------------------ |
| `phone_number` | String | Ya    | Nomor HP terdaftar. |

### B. Verify OTP & Reset

Memverifikasi OTP dan jika valid, mengizinkan user mengganti password.

- **URL**: `POST /reset-password`
- **Controller**: `Auth\MainAuthController@resetPasswordPost`

#### Parameters

| Param                   | Tipe   | Wajib | Keterangan                   |
| :---------------------- | :----- | :---- | :--------------------------- |
| `token`                 | String | Ya    | Token OTP (Hash).            |
| `phone`                 | String | Ya    | Nomor HP (Validator).        |
| `password`              | String | Ya    | Password baru (Min 8 chars). |
| `password_confirmation` | String | Ya    | Konfirmasi password.         |

---

## 5. Edge Case: Add Phone (Post-Login)

Flow khusus untuk user yang (mungkin) dibuatkan akun oleh admin tanpa nomor HP, atau data migrasi.

- **URL**: `GET /main/auth/add-phone/{encryptedUser}` (Signed Route)
- **Controller**: `showAddPhoneForm` & `processAddPhone`

Logic ini menangani input nomor HP baru, kirim OTP, dan verifikasi dalam satu rangkaian controller method.
