# Kategori Tagihan

Modul ini digunakan untuk mengelompokkan jenis tagihan (misal: SPP, Gedung, Praktikum). Modul ini bersifat **Lokal** (tidak ada sinkronisasi otomatis ke SIAKAD seperti modul Beasiswa).

## Get

Mengambil daftar kategori tagihan dengan dukungan DataTables.

- **URL**: `GET /admin/keuangan/master/kategori-tagihan`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Wajib | Keterangan                                                                                                                                                             |
| :-------------- | :----- | :---- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `page`          | Int    | Tidak | **⚠️ PERHATIAN:** Parameter ini digunakan untuk filter `sts_hapus`. <br>`1` = Data Aktif (Default). <br>`2` = Data di Tong Sampah. <br>Bukan untuk pagination halaman. |
| `search[value]` | String | Tidak | Keyword pencarian global.                                                                                                                                              |
| `length`        | Int    | Tidak | Limit data (Pagination limit).                                                                                                                                         |
| `start`         | Int    | Tidak | Offset data.                                                                                                                                                           |

---

## Store

Menyimpan data kategori tagihan baru atau memperbarui data yang ada.

- **URL**: `POST /admin/keuangan/master/kategori-tagihan/store`
- **Content-Type**: `multipart/form-data` atau `application/json`

### Body Parameters

| Param                      | Tipe   | Wajib    | Keterangan                                                                               |
| :------------------------- | :----- | :------- | :--------------------------------------------------------------------------------------- |
| `id_kategori_tagihan`      | Int    | Opsional | Isi untuk **Edit**. Kosongkan untuk **Create**.                                          |
| `nama_kategori_tagihan`    | String | Ya       | Nama Kategori (Unik). Min: 5 chars.                                                      |
| `nama_kategori_tagihan_en` | String | Ya       | Nama Kategori Inggris (Unik). Min: 5 chars.                                              |
| `urutan`                   | Int    | Ya       | Urutan tampilan di frontend (Unik). Tidak boleh ada dua kategori dengan nomor urut sama. |
| `sts_aktif`                | Int    | Ya       | `1` = Aktif, `2` = Nonaktif.                                                             |

> **🛡️ Security Note (Whitelist):**
> Endpoint ini menggunakan metode `$request->only()`. Field tambahan di luar daftar di atas akan diabaikan oleh sistem (Aman dari Mass Assignment).

---

## Delete

Mengelola penghapusan data.

### Soft Delete (Tong Sampah)

Memindahkan data ke status terhapus (bisa di-restore).

- **URL**: `POST /admin/keuangan/master/kategori-tagihan/destroy`
- **Param**: `id_kategori_tagihan` (Int)

### Restore

Mengembalikan data dari tong sampah.

- **URL**: `POST /admin/keuangan/master/kategori-tagihan/restore`
- **Param**: `id_kategori_tagihan` (Int)

### Force Delete

Menghapus data selamanya dari database. Hanya bisa dilakukan jika data sudah di-soft delete sebelumnya.

- **URL**: `POST /admin/keuangan/master/kategori-tagihan/force-delete`
- **Param**: `id_kategori_tagihan` (Int)

> **Integrity Check:**
> Sistem akan menolak penghapusan (Soft/Force) jika kategori ini sudah digunakan dalam transaksi tagihan siswa (Foreign Key Check manual di Controller).
