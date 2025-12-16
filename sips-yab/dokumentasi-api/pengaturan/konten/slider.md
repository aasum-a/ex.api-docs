# Slider

Modul ini digunakan untuk mengelola gambar slider yang tampil di halaman utama (Beranda) situs web.

## Get List Slider

Mengambil daftar slider yang ada.

- **URL**: `GET /admin/pengaturan/konten/slider`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan                                                           |
| :-------------- | :----- | :------------------------------------------------------------------- |
| `draw`          | Int    | Counter draw DataTables.                                             |
| `start`         | Int    | Offset data (pagination).                                            |
| `length`        | Int    | Limit data per halaman.                                              |
| `search[value]` | String | Keyword pencarian (Judul).                                           |
| `page`          | Int    | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |

---

## Store Data (Create / Update)

Menyimpan data slider baru atau memperbarui yang sudah ada.

- **URL**: `POST /admin/pengaturan/konten/slider/store`
- **Tipe**: `multipart/form-data`

### Body Parameters

| Param       | Tipe   | Wajib    | Keterangan                                                              |
| :---------- | :----- | :------- | :---------------------------------------------------------------------- |
| `id_slider` | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.                            |
| `judul`     | String | Ya       | Judul slider (Min 5, Max 255 chars).                                    |
| `gambar`    | File   | Ya\*     | File gambar (JPEG, PNG, GIF, SVG. Max 2MB). _Hanya wajib untuk Create._ |
| `urutan`    | Int    | Ya       | Urutan tampilan. **Harus unik.**                                        |
| `sts_aktif` | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                                            |

> **Validation Logic:**
>
> - Sistem akan menolak jika `urutan` yang dimasukkan sudah digunakan.
> - File gambar divalidasi berdasarkan MIME type dan ekstensi.
> - Saat update, jika gambar baru diunggah, gambar lama akan dihapus dari storage.

---

## Delete Actions

Mengelola penghapusan data slider.

### Soft Delete

Memindahkan data slider ke tong sampah sementara.

- **URL**: `DELETE /admin/pengaturan/konten/slider/destroy`
- **Param**: `id_slider` (UUID).

### Restore

Mengembalikan data slider dari tong sampah.

- **URL**: `POST /admin/pengaturan/konten/slider/restore`
- **Param**: `id_slider` (UUID).

### Force Delete (Hapus Permanen)

Menghapus data slider selamanya dari database dan menghapus file gambar dari penyimpanan publik.

- **URL**: `DELETE /admin/pengaturan/konten/slider/force-delete`
- **Param**: `id_slider` (UUID).
- **Warning:** Aksi ini tidak dapat dibatalkan. File fisik akan terhapus.
