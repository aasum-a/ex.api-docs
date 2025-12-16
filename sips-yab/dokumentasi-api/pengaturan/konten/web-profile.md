# Pengaturan Konten: Web Profile

Modul ini digunakan untuk mengatur aset grafis utama situs web, seperti Logo, Gambar Login, dan Gambar Pendaftaran. Modul ini hanya mengelola satu baris data (Singleton) yang akan di-update terus menerus.

## Get Web Profile

Mengambil data profil web yang sedang aktif untuk ditampilkan di form pengaturan.

- **URL**: `GET /admin/pengaturan/konten/web-profile`
- **Auth**: Session Cookie

---

## Store Data (Upload & Reset)

Endpoint ini menangani dua fungsi sekaligus berdasarkan parameter yang dikirim: **Mengunggah Gambar Baru** atau **Mereset Gambar ke Default**.

- **URL**: `POST /admin/pengaturan/konten/web-profile/store`
- **Tipe**: `multipart/form-data`

### Skenario A: Upload Gambar Baru

Digunakan untuk mengganti gambar profil web dengan file baru.

| Param         | Tipe | Wajib    | Keterangan                                                    |
| :------------ | :--- | :------- | :------------------------------------------------------------ |
| `logo`        | File | Opsional | Logo Web (WebP, Max 50MB, **422x90px**).                      |
| `login`       | File | Opsional | Gambar Halaman Login (WebP, Max 50MB, **2048x1636px**).       |
| `pendaftaran` | File | Opsional | Gambar Halaman Pendaftaran (WebP, Max 50MB, **2048x1636px**). |

> **Validation Logic:**
>
> - File gambar divalidasi dimensinya secara ketat. Jika ukuran tidak sesuai pixel yang ditentukan, upload akan ditolak.
> - Menggunakan format file `.webp` untuk optimasi web.
> - File lama otomatis dihapus dari _storage_ saat file baru berhasil diupload.

### Skenario B: Reset Gambar (Set Default)

Digunakan untuk menghapus gambar kustom dan kembali ke pengaturan awal (kosong/null).

| Param              | Tipe   | Wajib | Keterangan                                                          |
| :----------------- | :----- | :---- | :------------------------------------------------------------------ |
| `mode_set_default` | String | Ya    | Nama kolom yang ingin direset: `logo`, `login`, atau `pendaftaran`. |

> **Flow Reset:**
>
> 1. Sistem menerima parameter `mode_set_default`.
> 2. Sistem menghapus file fisik gambar yang ada di server.
> 3. Sistem mengosongkan kolom database terkait (`NULL`).

---

## Security Considerations

- **File Validation:** Endpoint ini membatasi tipe file hanya `webp` dan membatasi dimensi gambar untuk menjaga konsistensi tampilan layout.
- **Rollback Mechanism:** Menggunakan database transaction (`DB::beginTransaction`) dan manual rollback file upload untuk memastikan tidak ada file sampah yang tertinggal jika proses simpan ke database gagal.
