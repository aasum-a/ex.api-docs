# Web Service (API Clients)

Modul ini digunakan untuk mendaftarkan dan mengelola aplikasi pihak ketiga (Third-Party Apps) yang diizinkan mengakses API SIPS. Sistem akan menghasilkan kredensial otentikasi secara otomatis.

## Get List Web Service

Mengambil daftar klien aplikasi yang terdaftar.

- **URL**: `GET /admin/pengaturan/sistem/web-service`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan                                  |
| :-------------- | :----- | :------------------------------------------ |
| `draw`          | Int    | Counter draw DataTables.                    |
| `start`         | Int    | Offset data.                                |
| `length`        | Int    | Limit data per halaman.                     |
| `search[value]` | String | Keyword pencarian (Nama App, Owner, Email). |

---

## Store Data (Register Client)

Mendaftarkan aplikasi baru atau memperbarui data profil aplikasi yang sudah ada.

- **URL**: `POST /admin/pengaturan/sistem/web-service`

### Body Parameters

| Param            | Tipe   | Wajib    | Keterangan                                   |
| :--------------- | :----- | :------- | :------------------------------------------- |
| `id_web_service` | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**. |
| `app_name`       | String | Ya       | Nama Aplikasi (Alphanumeric + dot). Unik.    |
| `app_owner`      | String | Ya       | Nama Pemilik/Developer.                      |
| `app_email`      | String | Ya       | Email kontak developer.                      |
| `app_address`    | String | Ya       | Alamat fisik/kantor developer.               |
| `app_phone`      | String | Ya       | Nomor telepon kontak (8-15 digit).           |
| `sts_aktif`      | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                 |

### Automatic Credential Generation

Saat membuat data baru (**Create**), sistem akan otomatis menghasilkan dan menyimpan:

1.  **Client ID** (UUID)
2.  **Client Key** (Random String 100 chars)
3.  **Server Key** (Random String 100 chars)

> **⚠️ Security Note:**
> Kredensial ini (`server_key` & `client_key`) digunakan untuk otentikasi API (misal: Header Authorization). Pastikan kredensial ini diberikan secara aman kepada pihak pengembang yang bersangkutan.

---

## Delete Web Service

Menghapus akses aplikasi pihak ketiga.

- **URL**: `DELETE /admin/pengaturan/sistem/web-service/destroy`
- **Param**: `id_web_service` (UUID).
- **Effect**: Aplikasi dengan ID ini tidak akan bisa lagi mengakses API SIPS.

---

## Get Detail Web Service

Mengambil detail data aplikasi termasuk kredensialnya.

- **URL**: `GET /admin/pengaturan/sistem/web-service/get`
- **Query Param**: `id_web_service` (UUID).
