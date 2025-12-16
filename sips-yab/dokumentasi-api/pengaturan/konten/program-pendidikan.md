# Program Pendidikan

Modul ini mengelola konten "Program Pendidikan" yang ditampilkan di laman depan (Frontend). Konten ini biasanya berisi informasi tentang jenjang atau program unggulan sekolah.

## Get List Program Pendidikan

Mengambil daftar program pendidikan yang tersedia.

- **URL**: `GET /admin/pengaturan/konten/program-pendidikan`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan                                                           |
| :-------------- | :----- | :------------------------------------------------------------------- |
| `draw`          | Int    | Counter draw DataTables.                                             |
| `start`         | Int    | Offset data (pagination).                                            |
| `length`        | Int    | Limit data per halaman.                                              |
| `search[value]` | String | Keyword pencarian (Judul, Deskripsi).                                |
| `page`          | Int    | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |

---

## Store Data (Create / Update)

Menyimpan konten program pendidikan baru atau mengupdate yang sudah ada.

- **URL**: `POST /admin/pengaturan/konten/program-pendidikan/store`
- **Tipe**: `multipart/form-data`

### Body Parameters

| Param                   | Tipe   | Wajib    | Keterangan                                                              |
| :---------------------- | :----- | :------- | :---------------------------------------------------------------------- |
| `id_program_pendidikan` | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.                            |
| `judul`                 | String | Ya       | Judul program (Min 5, Max 192 chars).                                   |
| `deskripsi`             | String | Tidak    | Penjelasan detail program.                                              |
| `gambar`                | File   | Ya\*     | File gambar (JPEG, PNG, GIF, SVG. Max 2MB). _Hanya wajib untuk Create._ |
| `urutan`                | Int    | Ya       | Urutan tampilan. **Harus unik.**                                        |
| `sts_aktif`             | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                                            |

> **Validation Logic:**
> Sistem akan menolak jika `urutan` yang dimasukkan sudah digunakan oleh data lain. Anda harus memasukkan urutan yang belum terpakai atau mengedit data lain terlebih dahulu.

---

## Delete Actions

Mengelola penghapusan konten program pendidikan.

### Soft Delete

Memindahkan data ke tong sampah sementara.

- **URL**: `DELETE /admin/pengaturan/konten/program-pendidikan/destroy`
- **Param**: `id_program_pendidikan` (UUID).

### Restore

Mengembalikan data dari tong sampah.

- **URL**: `POST /admin/pengaturan/konten/program-pendidikan/restore`
- **Param**: `id_program_pendidikan` (UUID).

### Force Delete (Hapus Permanen)

Menghapus data selamanya dari database dan menghapus file gambar dari penyimpanan.

- **URL**: `DELETE /admin/pengaturan/konten/program-pendidikan/force-delete`
- **Param**: `id_program_pendidikan` (UUID).
- **Warning:** Aksi ini tidak dapat dibatalkan.
