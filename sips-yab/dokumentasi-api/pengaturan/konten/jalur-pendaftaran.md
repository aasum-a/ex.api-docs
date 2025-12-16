# Jalur Pendaftaran

Modul ini mengelola konten Jalur Pendaftaran, termasuk judul, deskripsi, dan gambar pendukung yang tampil di Laman Depan.

## Store Data (Create / Update)

Menyimpan konten Jalur Pendaftaran baru atau mengupdate yang sudah ada.

- **URL**: `POST /admin/pengaturan/konten/jalur-pendaftaran/store`
- **Tipe**: `multipart/form-data` (karena ada file upload)

### Body Parameters

| Param                  | Tipe   | Wajib    | Keterangan                                                              |
| :--------------------- | :----- | :------- | :---------------------------------------------------------------------- |
| `id_jalur_pendaftaran` | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.                            |
| `judul`                | String | Ya       | Judul jalur (Min 5, Max 192 chars).                                     |
| `deskripsi`            | String | Tidak    | Penjelasan detail jalur.                                                |
| `gambar`               | File   | Ya\*     | File gambar (JPEG, PNG, GIF, SVG. Max 2MB). _Hanya wajib untuk Create._ |
| `urutan`               | Int    | Ya       | Urutan tampil. **Harus unik.**                                          |
| `sts_aktif`            | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                                            |

> **⚠️ Security Note (Gambar):**
> Gambar yang diunggah **divalidasi** untuk mencegah _Unrestricted File Upload_. Pastikan file yang diunggah hanya memiliki ekstensi yang diizinkan.

---

## Force Delete (Hapus Permanen)

Menghapus data selamanya, **termasuk file gambarnya** dari _Storage_.

- **URL**: `DELETE /admin/pengaturan/konten/jalur-pendaftaran/force-delete`
- **Param**: `id_jalur_pendaftaran` (UUID).
- **Warning:** Aksi ini tidak dapat dibatalkan dan file akan terhapus dari disk.
