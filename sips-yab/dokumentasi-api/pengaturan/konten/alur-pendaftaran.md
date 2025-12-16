# Alur Pendaftaran

Modul ini berfungsi sebagai CMS untuk mengatur konten "Langkah-Langkah Pendaftaran" yang tampil di halaman depan (Landing Page).

## Get List Alur

Mengambil daftar alur pendaftaran.

- **URL**: `GET /admin/pengaturan/konten/alur-pendaftaran`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe   | Keterangan                                                           |
| :-------------- | :----- | :------------------------------------------------------------------- |
| `page`          | Int    | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |
| `search[value]` | String | Pencarian pada judul/deskripsi.                                      |

---

## Store Data (Create / Update)

Menyimpan konten alur baru atau mengupdate yang sudah ada.

- **URL**: `POST /admin/pengaturan/konten/alur-pendaftaran/store`

### Body Parameters

| Param                   | Tipe   | Wajib    | Keterangan                                   |
| :---------------------- | :----- | :------- | :------------------------------------------- |
| `id_alur_pendaftaran`   | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**. |
| `judul`                 | String | Ya       | Judul langkah (Min 5 chars).                 |
| `deskripsi`             | String | Tidak    | Penjelasan detail langkah.                   |
| `urutan`                | Int    | Ya       | Urutan tampilan (1, 2, 3...).                |
| `tampilkan_laman_depan` | Int    | Ya       | `1` (Tampil), `2` (Sembunyi).                |
| `sts_aktif`             | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                 |

> **⚠️ Logic Validasi Urutan:**
> Sistem **TIDAK** melakukan _auto-shift_ (geser otomatis).
> Jika Anda memasukkan nomor `urutan` yang sudah dipakai oleh data lain dengan status tampil yang sama, sistem akan menolak (Error 422) dan menampilkan daftar urutan yang sudah terpakai.

---

## Delete Actions

Mengelola penghapusan konten.

### Soft Delete (Tong Sampah)

Memindahkan data ke status terhapus sementara.

- **URL**: `DELETE /admin/pengaturan/konten/alur-pendaftaran/destroy`
- **Param**: `id_alur_pendaftaran` (UUID).

### Restore (Pulihkan)

Mengembalikan data dari tong sampah.

- **URL**: `POST /admin/pengaturan/konten/alur-pendaftaran/restore`
- **Param**: `id_alur_pendaftaran` (UUID).

### Force Delete (Hapus Permanen)

Menghapus data selamanya dari database.

- **URL**: `DELETE /admin/pengaturan/konten/alur-pendaftaran/force-delete`
- **Param**: `id_alur_pendaftaran` (UUID).
- **Warning:** Aksi ini tidak dapat dibatalkan.
