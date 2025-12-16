# Program Jurusan

Modul ini mengelola data Jurusan atau Program Studi yang ada di setiap Unit sekolah. Data ini terhubung erat dengan SIAKAD.

## Get List Jurusan

Mengambil daftar jurusan per unit.

- **URL**: `GET /admin/pendaftaran/master/program-jurusan`
- **Auth**: Session Cookie

### Query Parameters

| Param                   | Tipe | Keterangan                                                  |
| :---------------------- | :--- | :---------------------------------------------------------- |
| `filter_unit`           | UUID | Filter Unit Sekolah.                                        |
| `filter_sts_registrasi` | Int  | `1` (Buka Pendaftaran), `2` (Tutup).                        |
| `filter_pesantren`      | Int  | `1` (Pesantren), `2` (Non-Pesantren) - _Via Unit Relation_. |
| `filter_aktif`          | Int  | `1` (Aktif), `2` (Nonaktif).                                |

---

## Store Jurusan (Update Status)

Endpoint ini **TIDAK** digunakan untuk membuat jurusan baru (karena master data jurusan harus berasal dari SIAKAD). Endpoint ini hanya digunakan untuk mengubah status pendaftaran (`sts_registrasi`).

- **URL**: `POST /admin/pendaftaran/master/program-jurusan/store`

### Body Parameters

| Param                | Tipe | Wajib | Keterangan                           |
| :------------------- | :--- | :---- | :----------------------------------- |
| `id_program_jurusan` | UUID | Ya    | ID Jurusan yang akan diupdate.       |
| `sts_registrasi`     | Int  | Ya    | `1` = Buka Pendaftaran, `2` = Tutup. |

> **⚠️ Sync Trigger:**
> Perubahan status di sini akan memicu _Webhook_ ke SIAKAD (`/sync-data/program-jurusan/update`) agar data di kedua sistem tetap konsisten.

---

## Sync Data (Integrasi SIAKAD)

Sinkronisasi data jurusan dengan server SIAKAD.

- **Pull (Get Sync)**: `GET /admin/pendaftaran/master/program-jurusan/get-sync`
  - Menarik data jurusan dari SIAKAD.
  - Melakukan validasi relasi `id_unit_fr`. Jika Unit tidak ditemukan di lokal, data jurusan **DIABAIKAN** (Integrity Check).
- **Push (Update From Sync)**: `POST /admin/pendaftaran/master/program-jurusan/update-from-sync`
  - Webhook receiver untuk menerima perubahan nama/kode jurusan dari SIAKAD.
