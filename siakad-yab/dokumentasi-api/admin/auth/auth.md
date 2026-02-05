# Modul Authentication & Dashboard

Modul ini menangani otentikasi pengguna (Admin), manajemen sesi, pemulihan akun, dan tampilan ringkasan data statistik.

> ⚠️ **SECURITY/PERFORMANCE NOTE**
>
> 1. Rate Limiting Missing: Endpoint login rentan terhadap _Brute Force Attack_. Perlu implementasi `RateLimiter`.
> 2. User Enumeration: Endpoint `lupa-password` membocorkan status email terdaftar melalui validasi `exists`.
> 3. Token Expiry: Token reset password tidak memiliki waktu kadaluarsa (forever valid) dan menggunakan kolom `remember_token` yang tidak standar.
> 4. Performance Issue: Dashboard menggunakan method `get()->count()` yang membebani memori server (N+1 memory issue).

## 1. Login Authentication

- **URL**: `POST /admin/login/store`
- **Auth**: Guest
- **Controller**: Admin\Auth\AuthController@loginStore

> Description: Memvalidasi kredensial pengguna berdasarkan `email`, `password`, dan `tahun ajaran`. jika valid, sistem membuat sesi dan `cookie otentikasi`.

### Paramters

| Param      | Tipe   | Wajib | Keterangan                                    |
| :--------- | :----- | :---- | :-------------------------------------------- |
| `email`    | String | Ya    | Email atau Username pengguna                  |
| `password` | String | Ya    | Password akun pengguna                        |
| `data_ta`  | UUID   | Ya    | ID Tahun Ajaran yang dipilih untuk sesi aktif |

### Response Example

- **Success (Redirect)**: HTTP 302 Found -> Redirect to `/admin`
- **Error (Redirect with Flash Data)**:

```json
// Disimpan di Session 'data'
{
  "status": "error",
  "message": {
    "email": "Username atau Password Anda salah!",
    "password": "Username atau Password Anda salah!"
  }
}
```

---

## 2. Request Reset Password

- **URL**: `POST /admin/lupa-password/store`
- **Auth**: Guest
- **Controller**: `Admin\Auth\AuthController@lupaPasswordStore`

> Description: Menerima permintaan reset password. Sistem akan men-generate token (menggunakan kolom `remember_token`) dan mengirimkan link reset ke email pengguna.

### Parameters

| Param   | Tipe   | Wajib | Keterangan                                   |
| :------ | :----- | :---- | :------------------------------------------- |
| `email` | String | Ya    | Email terdaftar untuk dikirimkan link reset. |

### Response Example

- **Success/Error Generic (Redirect)**:

```json
{
  "status": "error_sys",
  "message": "Silahkan cek folder inbox/spam email Anda. kami telah mengirim link reset password."
}
```

## 3. Execute Reset Password

- **URL**: `POST /admin/reset-password/store`
- **Auth**: Guest
- **Controller**: `Admin\Auth\AuthController#resetPasswordStore`

> Description: Memproses perubahan password baru berdasarkan token verifikasi.

### Parameters

| Param                 | Tipe   | Wajib | Keterangan                                                  |
| :-------------------- | :----- | :---- | :---------------------------------------------------------- |
| token                 | String | Ya    | Token verifikasi dari email (check param `remember_token`). |
| `password`            | String | Ya    | Password baru (Min 8 char).                                 |
| `konfirmasi_password` | String | Ya    | Harus sama dengan password                                  |

### Response Example

- **Success (Redirect)**

```json
{
  "status": "error_sys", // Note: Penamaan status 'error_sys' untuk sukses ini membingungkan (Bad Practice)
  "message": "Password berhasil direset."
}
```

---

## 4. Dashboard Statistics

- **URL**: `GET /admin`
- **Auth**: Session/Cookie
- **Controller**: `Admin\DashboardController@index`

> Description: Menampilkan halaman utama admin dengan statistik jumlah Peserta Didik, Guru, Kelas, dan Mata Pelajaran berdasarkan Unit akses pengguna. **Side Effect**: Trigger otomatis pengecekan _payment expiry_ dan pembersihan log aktivitas.

### Parameters

| Param   | Tipe | Wajib | Keterangan                                                         |
| :------ | :--- | :---- | :----------------------------------------------------------------- |
| unit_id | Int  | Tidak | (Optional) Filter data berdasarkan unit via helper \_get_id_unit() |

### Response Example

- **Render View**: `pages.admin.dashboard`
- **Variables**: `$user`, `$peserta_didik` (Int), `$guru` (Int) `$mata_pelajaran` (Int), `$kelas` (Int).
