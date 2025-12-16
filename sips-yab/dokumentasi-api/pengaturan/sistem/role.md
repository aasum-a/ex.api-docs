# Role

Modul ini mengelola data peran (Role) dan hak akses (Permissions) yang melekat padanya. Role digunakan untuk mengelompokkan user berdasarkan tingkat akses mereka.

## Get List Role

Mengambil daftar Role beserta permission yang dimilikinya.

- **URL**: `GET /admin/pengaturan/sistem/role`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan                     |
| :-------------- | :----- | :----------------------------- |
| `draw`          | Int    | Counter draw DataTables.       |
| `start`         | Int    | Offset data.                   |
| `length`        | Int    | Limit data per halaman.        |
| `search[value]` | String | Keyword pencarian (Nama Role). |

> **Data Relation:**
> Endpoint ini melakukan Eager Loading (`with('m_permissions')`) untuk mengambil daftar permission setiap role dalam satu query, menghindari N+1 query problem.

---

## Store Role (Create / Update / Assign Permissions)

Menyimpan data Role dan mengatur permission yang terkait.

- **URL**: `POST /admin/pengaturan/sistem/role`

### Body Parameters

| Param          | Tipe   | Wajib    | Keterangan                                                  |
| :------------- | :----- | :------- | :---------------------------------------------------------- |
| `id_role`      | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.                |
| `name`         | String | Ya       | Nama Role (Unique). Min 2, Max 191 chars.                   |
| `level`        | String | Ya       | Level hierarki role (Misal: Admin, Staff).                  |
| `permission[]` | Array  | Ya       | Array ID Permission (UUID) yang akan diberikan ke role ini. |

### Security & Logic Rules

1.  **Super Admin Protection:** Role dengan nama `super.admin` **TIDAK BISA** diedit oleh user selain `super.admin` itu sendiri (Hardcoded protection).
2.  **Permission Validation:** Sistem memvalidasi apakah semua ID dalam array `permission` benar-benar ada di database. Jika ada satu ID palsu, proses ditolak.
3.  **Sync Logic:**
    - **Create:** Menggunakan `attach()` untuk menghubungkan permission baru.
    - **Update:** Menggunakan `sync()` untuk mengganti permission lama dengan yang baru. Jika array kosong, permission akan di-`detach` (dihapus semua).

---

## Delete Role

Menghapus data Role.

- **URL**: `DELETE /admin/pengaturan/sistem/role/destroy`
- **Param**: `id_role` (UUID).
- **Warning:** Menghapus role akan mencabut akses semua user yang memiliki role tersebut.

---

## Get Role Details & Permissions

Mengambil detail satu role beserta permission yang sudah dimilikinya (biasanya untuk mengisi form edit).

- **URL**: `GET /admin/pengaturan/sistem/role/get-role-has-permission`

### Query Parameters

| Param     | Tipe | Wajib | Keterangan           |
| :-------- | :--- | :---- | :------------------- |
| `id_role` | UUID | Ya    | ID Role yang dicari. |

### Response Data

Mengembalikan object Role dengan property `m_permissions` berisi array object permission.
