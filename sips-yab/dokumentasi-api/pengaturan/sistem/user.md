# User

Modul ini mengelola data pengguna (User) yang memiliki akses ke dalam sistem, termasuk detail profil, peran, dan kredensialnya.

## Get List User

Mengambil daftar semua pengguna dalam sistem.

- **URL**: `GET /admin/pengaturan/sistem/user`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan               |
| :-------------- | :----- | :----------------------- |
| `draw`          | Int    | Counter draw DataTables. |
| `start`         | Int    | Offset data.             |
| `length`        | Int    | Limit data per halaman.  |
| `search[value]` | String | Keyword pencarian.       |

> **Data Relation:**
> Endpoint ini melakukan Eager Loading (`with('m_role')`) untuk menampilkan nama Role yang dimiliki oleh setiap User.

---

## Store User (Create / Update)

Menyimpan data User baru atau memperbarui data User yang sudah ada.

- **URL**: `POST /admin/pengaturan/sistem/user`
- **Content Type**: `multipart/form-data` (karena ada upload Avatar)

### Body Parameters (Form Data)

| Param                   | Tipe       | Wajib    | Keterangan                                                      |
| :---------------------- | :--------- | :------- | :-------------------------------------------------------------- |
| `id_user`               | UUID       | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.                    |
| `name`                  | String     | Ya       | Nama lengkap User.                                              |
| `username`              | String     | Ya       | Username (Unique, Alphanumeric + dot, Min 5, Max 100).          |
| `email`                 | String     | Ya       | Email (Unique, Valid Email format).                             |
| `id_role_fr`            | UUID       | Ya       | ID Role (Foreign Key) yang dimiliki User.                       |
| `avatar`                | File       | Opsional | Foto profil. Max 20MB. (jpg, jpeg, png).                        |
| `password`              | String     | Opsional | Password baru. Min 8, Max 100. Wajib dikonfirmasi.              |
| `password_confirmation` | String     | Opsional | Konfirmasi Password. Harus sama dengan `password`.              |
| `address`               | String     | Opsional | Alamat User.                                                    |
| `no_handphone`          | String     | Opsional | Nomor HP lokal (Unique).                                        |
| `no_handphone_intl`     | String     | Opsional | Nomor HP format internasional (Unique).                         |
| `no_handphone_cc`       | String     | Ya       | Kode negara (Country Code) HP. Wajib diisi (Contoh: +62).       |
| `notification_channels` | JSON Array | Opsional | Saluran notifikasi yang dipilih (disimpan sebagai JSON string). |

### Security & File Handling Logic

1.  **Super Admin Protection:** User dengan `username = 'super.admin'` **TIDAK BISA** diedit oleh user lain kecuali user tersebut adalah `super.admin`.
2.  **Uniqueness Validation:** Memastikan `username`, `email`, `phone_number`, dan `phone_number_intl` unik (dengan pengecualian untuk update).
3.  **Password Hashing:** Jika field `password` diisi, password akan di-hash menggunakan `Hash::make()` sebelum disimpan.
4.  **Avatar Management:**
    - Jika `avatar` di-upload, file lama (jika ada) akan dihapus dari _storage_ dan yang baru disimpan.
    - Path penyimpanan: `storage/img/avatars/`.
5.  **Phone Country Code (CC) Sanitization:** Nilai `no_handphone_cc` di-sanitasi agar selalu berformat `+XX` sebelum disimpan.

---

## Get User Details

Mengambil detail data User.

- **URL**: `GET /admin/pengaturan/sistem/user/get`
- **Query Param**: `id_user` (UUID) atau `search` (String).

> **Data Relation:**
> Mengambil data User dengan Eager Loading `m_role` untuk mendapatkan detail Role-nya.

---

## Delete User

Menghapus data User.

- **URL**: `DELETE /admin/pengaturan/sistem/user/destroy`
- **Param**: `id_user` (UUID).

### Security & Logic Rules

1.  **Self-Deletion Prevention:** User **TIDAK BISA** menghapus akunnya sendiri (`$data->id_user === auth()->user()->id_user`).
2.  **Super Admin Protection:** User dengan `username = 'super.admin'` **TIDAK BISA** dihapus.
3.  **Avatar Cleanup:** Jika penghapusan berhasil, file Avatar terkait di _storage_ juga akan dihapus.
4.  **Foreign Key Check:** Jika data user memiliki keterkaitan (Foreign Key) di tabel lain, penghapusan akan dibatalkan untuk menjaga integritas data.

---

## 5. Account Settings View

Menampilkan halaman view untuk pengaturan akun yang sedang login.

- **URL**: `GET /admin/pengaturan/sistem/akun`
- **Auth**: Session Cookie
- **Action**: Mengambil data user yang sedang login menggunakan `auth()->user()->id_user`.
