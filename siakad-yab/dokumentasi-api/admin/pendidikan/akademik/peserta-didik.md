# Peserta Didik

Modul ini adalah pusat data siswa. Mengelola biodata diri, data orang tua (Ayah, Ibu, Wali), data akademik (Jurusan, Kelas), hingga integrasi pembuatan akun User untuk orang tua.

> ⚠️ PERFORMANCE WARNING
>
> - Import: Fitur import excel dan imput lanjutan memproses data baris-per-baris dengan query database intensif. Hindari upload >200 baris sekaligus.
> - Side Effect: Setiap penyimpanan data siswa (create/update) akan memicu kalkulasi ulang tabel pembantu keuangan (`helper_tagihan`). Proses ini sinkronus dan mungkin memakan waktu.

---

## 1. List Data Peserta Didik

- **URL**: `GET /admin/pendidikan/akademik/peserta-didik`
- **Auth**: Session
- **Controller**: `PesertaDidikController@index`

> Description: Menampilkan daftar seluruh peserta didik.
>
> Fitur Filter:
>
> - Status Aktif: Aktif, Lulus, Keluar, atau Semua.
> - Akun Orang Tua: Filter siswa yang orang tuanya sudah/belum punya akun login.
> - Tingkat & Kelas: Filter berdasarkan rombel.

### Parameters

| Param                        | Tipe | Keterangan                       |
| :--------------------------- | :--- | :------------------------------- |
| `draw`, `start`, `length`    | Int  | Standard Datatables              |
| `filter_status_aktif`        | Int  | `1 = Aktif`, `2 = Alumni/Keluar` |
|                              |      | `3 = Lulus`, `4 = Mutasi`        |
| `filter_akun_orang_tua`      | Int  | `1 = Sudah Ada`, `2 = Belum Ada` |
| `filter_kelas_peserta_didik` | UUID | Filter ID Kelas                  |

---

## 2. Store Peserta Didik

- **URL**: `POST /admin/pendidikan/akademik/peserta-didik/store`
- **Auth**: Session
- **Controller**: `PesertaDidikController@store`

> Description
> Menyimpan data lengkap siswa dan orang tua.
> Flow Logic:
>
> 1. Validasi: Cek NIK (Max 5 duplikat), NIS/NISN Unik.
> 2. Simpan Siswa: Insert/Update tabel peserta_didik.
> 3. Simpan Data Lengkap: Insert/Update tabel data_lengkap (Alamat, Fisik, dll).
> 4. Simpan Orang Tua:
>    - Cek apakah NIK Ibu/Ayah/Wali sudah ada di tabel orang_tua?
>    - Jika belum, Insert baru. Jika sudah, Update data existing.
>    - Auto-Create User: Jika data orang tua baru, otomatis buatkan akun User (Role Wali Murid) via UserController.
> 5. Trigger Keuangan: Generate ulang tabel tagihan siswa tersebut.

### Parameters

| Param                   | Tipe   | Wajib    | Keterangan       |
| :---------------------- | :----- | :------- | :--------------- |
| `id_peserta_didik`      | UUID   | Optional | Isi saat edit    |
| `nama_peserta_didik`    | String | Ya       | Nama Lengkap     |
| `nis`, `nisn`, `nik`    | String | Tidak    | Identitas unik   |
| `id_tahun_ajaran_fr`    | UUID   | Ya       | Tahun masuk      |
| `id_program_jurusan_fr` | UUID   | Ya       | Jurusan masuk    |
| `nik_ibu`, `nama_ibu`   | String | Ya       | Data Ibu Kandung |
| `nik_ayah`, `nama_ayah` | String | Tidak    | Data Ayah        |
| `no_kk`                 | String | Ya       | Nomor KK         |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Import Data

- **URL**: `POST /admin/pendidikan/akademik/peserta-didik/import`
- **Auth**: Session
- **Controller**: `PesertaDidikController@import` (Parsing) & `store_import` (Saving)

> Description:
> Mengupload data siswa masal dari file Excel (Format Dapodik).
> Logic Store Import:
>
> 1. Loop data JSON hasil parsing.
> 2. Cek duplikasi berdasarkan NIPD/NISN/NIK.
> 3. Cari ID Wilayah (Kecamatan/Kab/Prov) berdasarkan kesamaan nama string (LIKE %nama%).
> 4. Insert data siswa baru.
> 5. (Note: Import ini tidak otomatis membuat user orang tua).

---

## 4. Import Lanjutan (Update Data)

- **URL**: `POST /admin/pendidikan/akademik/peserta-didik/import-lanjutan`
- **Controller**: `PesertaDidikController@importLanjutan` & `store_import_lanjutan`

> Description: Fitur khusus untuk Memperbarui data siswa yang sudah ada secara massal (misal: melengkapi NIK, NISN, atau Nama Orang Tua yang kosong).
>
> Logic: Mencocokkan data berdasarkan ID Peserta Didik, Hanya mengupdate field yang disediakan di Excel.

---

### 5. Get Detail & Helper

- **URL**: `GET .../get`
- **Controller**: `PesertaDidikController@get`

> Description: Mengambil detail lengkap data siswa, termasuk data orang tua, kelas aktif, dan wilayah.

### Parameters

| Param              | Tipe | Wajib | Keterangan                             |
| :----------------- | :--- | :---- | :------------------------------------- |
| `id_peserta_didik` | UUID | Ya    | ID Siswa                               |
| `id_unit`          | Int  | Ya    | ID Unit (untuk filter wilayah/jurusan) |

---

## 6. Delete & Restore

- **URL**: `DELETE .../destroy` & `POST .../restore`
- **Auth**: Session

> Description:
>
> - Destroy: soft-delete data siswa, menonaktifkan user siswa.
> - Restore: mengembalikan data siswa.
> - Validasi: Tidak bisa dihapus jika siswa memiliki data terkait di:
>   - Absensi, Daftar Kelas, Transkrip Nilai, Riwayat Pelanggaran/Prestasi/Kesehatan. 
