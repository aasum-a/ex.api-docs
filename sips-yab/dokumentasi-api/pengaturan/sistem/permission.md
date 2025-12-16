# Pengaturan Sistem: Permission (Hak Akses)

Modul ini mengelola data Hak Akses (Permissions) dalam sistem Role-Based Access Control (RBAC).

## Get List Permission

Mengambil daftar permission yang tersedia.

- **URL**: `GET /admin/pengaturan/sistem/permission`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan                           |
| :-------------- | :----- | :----------------------------------- |
| `draw`          | Int    | Counter draw DataTables.             |
| `start`         | Int    | Offset data.                         |
| `length`        | Int    | Limit data per halaman.              |
| `search[value]` | String | Keyword pencarian (Nama Permission). |

---

## Store Permission (Create / Update)

Menyimpan data permission baru atau memperbarui yang sudah ada.

- **URL**: `POST /admin/pengaturan/sistem/permission`

### Body Parameters

| Param           | Tipe   | Wajib    | Keterangan                                            |
| :-------------- | :----- | :------- | :---------------------------------------------------- |
| `id_permission` | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.          |
| `name`          | String | Ya       | Nama Permission (Unique). Min 2 chars, Max 191 chars. |

> **Validation Logic:**
> Sistem memvalidasi keunikan `name` permission, dengan pengecualian ID saat proses update.

---

## Delete Permission

Menghapus data permission.

- **URL**: `DELETE /admin/pengaturan/sistem/permission/destroy`
- **Param**: `id_permission` (UUID).

> **Integrity Check:**
> Sistem akan mengecek tabel `role_has_permission`. Jika permission ini sedang digunakan oleh Role tertentu, penghapusan akan **DITOLAK**.

---

## Get Detail Permission

Mengambil detail data permission.

- **URL**: `GET /admin/pengaturan/sistem/permission/get`
- **Query Param**: `id_permission` (UUID) atau `search` (String).
