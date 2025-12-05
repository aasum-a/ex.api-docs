# Kategori Beasiswa

Modul ini mengelola pengelompokan jenis beasiswa (misal: Prestasi, KIP-K, Yayasan).

> **⚠️ INTEGRATION WARNING**
> Modul ini memiliki **Synchronous Dependency** ke sistem SIAKAD.
> Setiap aksi Simpan/Hapus akan memicu HTTP Request ke API SIAKAD.
>
> 1. Jika API SIAKAD _Down/Timeout_, data **GAGAL** disimpan di SIPS (Rollback).
> 2. Proses simpan bisa terasa lambat tergantung koneksi ke server SIAKAD.

## Get

Mengambil daftar kategori beasiswa (Standard DataTables).

- **URL**: `GET /admin/keuangan/master/kategori-beasiswa`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Wajib | Keterangan                                  |
| :-------------- | :----- | :---- | :------------------------------------------ |
| `search[value]` | String | Tidak | Keyword pencarian global.                   |
| `sts_hapus`     | Int    | Tidak | Filter status: `1` (Aktif), `2` (Terhapus). |

---

## Store

Menyimpan data kategori dan melakukan **Real-time Sync** ke SIAKAD.

- **URL**: `POST /admin/keuangan/master/kategori-beasiswa/store`
- **Content-Type**: `multipart/form-data`

### Body Parameters

| Param                       | Tipe   | Wajib    | Keterangan                                      |
| :-------------------------- | :----- | :------- | :---------------------------------------------- |
| `id_kategori_beasiswa`      | Int    | Opsional | Isi untuk **Edit**. Kosongkan untuk **Create**. |
| `nama_kategori_beasiswa`    | String | Ya       | Nama Kategori (Unik). Min: 3 chars.             |
| `nama_kategori_beasiswa_en` | String | Ya       | Nama Kategori (Inggris).                        |
| `sts_aktif`                 | Int    | Ya       | `1` = Aktif, `2` = Nonaktif.                    |

### Error Responses (Sync Specific)

Karena bergantung pada pihak ketiga, error code bisa bervariasi:

| Code  | Message                               | Penyebab                                       |
| :---- | :------------------------------------ | :--------------------------------------------- |
| `401` | Pengaturan Web Services belum di atur | Config URL/Token SIAKAD di `.env` salah.       |
| `500` | Internal Server Error                 | Bisa error DB SIPS atau Error dari API SIAKAD. |

---

## Delete & Restore

Soft delete/restore data lokal dan mengirim trigger update status ke SIAKAD.

- **URL Delete**: `POST /admin/keuangan/master/kategori-beasiswa/destroy`
- **URL Restore**: `POST /admin/keuangan/master/kategori-beasiswa/restore`

### Body Parameters

| Param                  | Tipe | Wajib | Keterangan                  |
| :--------------------- | :--- | :---- | :-------------------------- |
| `id_kategori_beasiswa` | Int  | Ya    | ID data yang akan diproses. |

---

## System Integration Endpoints (Webhook)

Endpoint ini **TIDAK** untuk digunakan oleh UI Admin/User. Endpoint ini khusus didesain untuk menerima data lemparan (Callback/Webhook) dari server SIAKAD (Server-to-Server Communication).

> **🛡️ Security Audit Required**
> Pastikan route menuju endpoint ini dilindungi oleh Middleware IP Whitelist atau Token Auth. Kode controller tidak memiliki validasi otentikasi eksplisit.

### A. Update From Sync (Receiver)

Menerima push data perubahan dari SIAKAD.

- **URL**: `POST /admin/keuangan/master/kategori-beasiswa/update-from-sync`
- **Method**: `POST`

**Body Parameters (Payload dari SIAKAD):**

- `id_kategori_beasiswa` (Required)
- `nama_kategori_beasiswa`
- `sts_aktif`
- `sts_hapus`
- `updated_by`

### B. Soft Delete From Sync (Receiver)

Menerima trigger penghapusan dari SIAKAD.

- **URL**: `POST /admin/keuangan/master/kategori-beasiswa/soft-delete-from-sync/{id}`
