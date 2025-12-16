# Data Calon Siswa

Modul ini adalah **Dashboard Utama** bagi panitia PPDB untuk memantau data pendaftar yang masuk. Menyajikan tabel besar dengan fitur filter yang kompleks.

> **⚠️ PERFORMANCE WARNING**
> Endpoint ini menggunakan query `JOIN` yang sangat masif (10+ tabel).
> Pencarian global (`search[value]`) melakukan scan ke semua kolom teks.
> **Rekomendasi:** Gunakan filter spesifik (Unit/Gelombang) daripada pencarian teks global untuk performa lebih cepat.

## Get

Mengambil daftar calon siswa dengan pagination server-side.

- **URL**: `GET /admin/pendaftaran/calon-siswa/data-calon-siswa`
- **Auth**: Session Cookie

### Query Parameters

Selain parameter standar DataTables (`draw`, `start`, `length`), endpoint ini menerima banyak filter khusus:

| Param                       | Tipe | Keterangan                                                 |
| :-------------------------- | :--- | :--------------------------------------------------------- |
| `filter_unit`               | UUID | Filter Pilihan Sekolah 1.                                  |
| `filter_program_jurusan`    | UUID | Filter Jurusan Pilihan 1.                                  |
| `filter_unit_ps`            | UUID | Filter Pilihan Sekolah 2 (Pindah Sekolah).                 |
| `filter_gelombang`          | UUID | Filter Gelombang Masuk.                                    |
| `filter_status_calon_siswa` | Int  | `1` (Baru), `2` (Diterima), `3` (Cadangan), `4` (Ditolak). |
| `filter_jenis_kelamin`      | Char | `L` atau `P`.                                              |
| `filter_provinsi`           | ID   | Filter Wilayah Asal.                                       |

### Logic Session

Secara default, data akan difilter berdasarkan **Tahun Ajaran** yang aktif di session admin (`_get_tahun_ajaran_session()`). Data tahun lalu tidak akan muncul kecuali session diubah.

### Data Privacy Note

Response JSON berisi data sensitif (**NIK, No KK, No HP, Email**) secara _plain text_.
Pastikan endpoint ini hanya dapat diakses oleh Admin/Panitia yang berwenang (Authorized Personnel Only).

---

## Navigasi ke Detail

Dari list ini, Admin biasanya akan diarahkan ke modul **Tahap Pendaftaran** (`step/{n}/{id}`) untuk melihat/mengedit detail siswa.

- **Edit Form**: `/admin/pendaftaran/calon-siswa/data-calon-siswa/tahap-pendaftaran/step/1/{id_calon_siswa}`
- **Cek Pembayaran**: `/admin/pendaftaran/calon-siswa/data-calon-siswa/tahap-pendaftaran/step/3/{id_calon_siswa}`
