# Testimoni Alumni

Modul ini digunakan untuk mengelola data testimoni alumni (atau pengguna) yang tampil di situs web.

## Get List Testimoni (DataTables)

Mengambil daftar data testimoni untuk tampilan tabel admin (DataTables).

- **URL**: `GET /admin/pengaturan/konten/testimoni`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan                                                           |
| :-------------- | :----- | :------------------------------------------------------------------- |
| `draw`          | Int    | Counter draw DataTables.                                             |
| `start`         | Int    | Offset data (pagination).                                            |
| `length`        | Int    | Limit data per halaman.                                              |
| `search[value]` | String | Keyword pencarian.                                                   |
| `page`          | Int    | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |

---

## Get Single/All Data

Mengambil detail data testimoni berdasarkan ID, atau mengambil semua data jika tidak ada ID.

- **URL**: `GET /admin/pengaturan/konten/testimoni/get`

### Query Parameters

| Param          | Tipe   | Wajib    | Keterangan                                                    |
| :------------- | :----- | :------- | :------------------------------------------------------------ |
| `id_testimoni` | UUID   | Opsional | ID testimoni. Jika ada, ambil detail.                         |
| `search`       | String | Opsional | Keyword pencarian (hanya berlaku jika `id_testimoni` kosong). |

---

## Store Data (Create / Update)

Menyimpan data testimoni baru atau memperbarui yang sudah ada.

- **URL**: `POST /admin/pengaturan/konten/testimoni/store`
- **Tipe**: `multipart/form-data`

### Body Parameters

| Param          | Tipe   | Wajib    | Keterangan                                                                     |
| :------------- | :----- | :------- | :----------------------------------------------------------------------------- |
| `id_testimoni` | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.                                   |
| `judul`        | String | Ya       | Judul atau Nama Alumni/Testimonial (Min 5, Max 192 chars).                     |
| `sub_judul`    | String | Opsional | Jabatan atau Posisi Alumni (Max 192 chars).                                    |
| `gambar`       | File   | Ya\*     | Foto/Avatar Alumni (JPEG, PNG, GIF, SVG. Max 2MB). _Hanya wajib untuk Create._ |
| `deskripsi`    | Text   | Opsional | Isi/Teks Testimoni.                                                            |
| `urutan`       | Int    | Ya       | Urutan tampilan. **Harus unik.**                                               |
| `sts_aktif`    | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                                                   |

> **Validation Logic:**
>
> - Validasi keunikan `urutan` dilakukan terpisah dari _Validator_ bawaan Laravel.
> - Saat update, jika gambar baru diunggah, gambar lama akan dihapus dari storage.

---

## Delete Actions (Soft, Restore, Force)

### - Soft Delete

Memindahkan data testimoni ke tong sampah (`sts_hapus = 2`).

- **URL**: `DELETE /admin/pengaturan/konten/testimoni/destroy`
- **Param**: `id_testimoni` (UUID).

### - Restore

Mengembalikan data testimoni dari tong sampah (`sts_hapus = 1`).

- **URL**: `POST /admin/pengaturan/konten/testimoni/restore`
- **Param**: `id_testimoni` (UUID).

### - Force Delete (Hapus Permanen)

Menghapus data testimoni selamanya dari database dan menghapus file gambar.

- **URL**: `DELETE /admin/pengaturan/konten/testimoni/force-delete`
- **Param**: `id_testimoni` (UUID).
- **Warning:** Aksi ini tidak dapat dibatalkan. File fisik akan terhapus.
