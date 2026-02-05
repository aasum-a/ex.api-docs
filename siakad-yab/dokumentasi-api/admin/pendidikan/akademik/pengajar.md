# Pengajar

Modul ini menangani pengelolaan data Tenaga Pendidik (Guru) dan Kependidikan. Fitur Utama:

- CRUD Data Biodata Lengkap (Dapodik-like).
- Import Data Massal: Fitur upload Excel untuk input data guru dalam jumlah banyak.
- Auto-User Generation: Setiap guru otomatis dibuatkan akun pengguna (Role Level 3) untuk login ke sistem.
- Integrasi Data Wilayah (Provinsi s/d Kelurahan) dan Agama.

> ⚠️ PERFORMANCE WARNING Fitur Import Pengajar melakukan validasi database (cek duplikasi) dan pencarian referensi wilayah untuk setiap baris data Excel. Proses import data dalam jumlah besar (>100 baris) disarankan dilakukan secara bertahap untuk menghindari Request Timeout.

## 1. List Data Pengajar

- **URL**: `GET /admin/pendidikan/akademik/pengajar`
- **Auth**: Session
- **Controller**: `PengajarController@index`

> Description: Menampilkan daftar guru/pengajar. Mendukung server-side searching dan filtering unit.

### Parameters

| Param           | Tipe   | Wajib | Keterangan          |
| :-------------- | :----- | :---- | :------------------ |
| `draw`          | Int    | Tidak | Param Datatables    |
| `search[value]` | String | Tidak | Pencarian global    |
| `page`          | Int    | Tidak | Filter status hapus |
| `filter_unit`   | Int    | Tidak | Filter Unit         |

---

## 2. Store Pengajar

- **URL**: `POST /admin/pendidikan/akademik/pengajar/store`
- **Auth**: Session
- **Controller**: `PengajarController@store`

> Description: Menyimpan data guru.
> Create: Membuat record `pengajar` dan `user` baru.
> Update: Memperbarui data `pengajar` dan menyinkronkan data `user` terkait.

### Parameters

| Param           | Tipe   | Keterangan                  |
| :-------------- | :----- | :-------------------------- |
| `nama_pengajar` | String | Nama lengkap                |
| `email`         | String | Email                       |
| `no_hp`         | String | Nomor HP                    |
| `jenis_kelamin` | enum   | L/P                         |
| `tempat_lahir`  | String | Kota lahir                  |
| `tanggal_lahir` | Date   | YYYY-MM-DD                  |
| `sts_aktif`     | Int    | `1 = Aktif`, `2 = Nonaktif` |

### Parameters (Dapodik)

| Param                 | Tipe   | Keterangan                                 |
| :-------------------- | :----- | :----------------------------------------- |
| `nuptk`, `nip`, `nik` | String | Nomor Identitas                            |
| `sts_kepegawaian`     | String | PNS/Honorer/Tetap                          |
| `jenis_ptk`           | String | Guru Mata pelajaran/wali kelas             |
| `sts_perkawinan`      | Int    | 1 = Belum Kawin, 2 = Kawin, 3 = Janda/Duda |
| `...`                 | ...    | (Data alamat & keluarga lengkap)           |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Import Pengajar

- **URL**: `POST /admin/pendidikan/akademik/pengajar/import`
- **Auth**: Session
- **Controller**: `PengajarController@import`

> Description: Memproses file Excel berisi data guru.
> Flow:
>
> 1. Upload Excel
> 2. Parsing data baris per baris
> 3. Mapping data Excel ke format JSON preview
> 4. Mengembalikan data JSON ke frontend untuk diverifikasi user sebelum disimpan (`store_import`)

### Parameters

| Param  | Tipe | Wajib | Keterangan          |
| :----- | :--- | :---- | :------------------ |
| `file` | File | Ya    | Format .xls / .xlsx |

### Response Example

```json
{
  "status": "success",
  "data": [
    { "no": 1, "nama": "Budi", "nip": "123...", ... },
    ...
  ]
}
```

---

## 4. Eksekusi Import

- URL: POST /admin/pendidikan/akademik/pengajar/store-import
- Auth: Session
- Controller: PengajarController@store_import

> Description: Menyimpan data hasil import Excel ke database.
> Logic:
>
> 1. Loop data JSON
> 2. Cek duplikasi berdasarkan NIK/NIP/NUPTK/NPWP
> 3. Jika duplikat: update data existing
> 4. Jika baru:
>    - Generate username (`kode_pengajar`)
>    - Buat user baru (role guru)
>    - Cari ID Wilayah & Agama berdasarkan string nama
>    - Simpan data pengajar

### Parameters

| Param | Tipe        | Wajib | Keterangan                                 |
| :---- | :---------- | :---- | :----------------------------------------- |
| data  | JSON String | Ya    | Array object data guru hasil parsing Excel |

---

## 5. Delete & Restore

- URL: DELETE .../destroy & POST .../restore
- Auth: Session

> Description:
>
> - Destroy: Soft delete data pengajar dan menonaktifkan akun user (`active = 2`)
> - Restore: Mengaktifkan kembali data pengajar dan akun user
> - Validasi: Tidak bisa hapus jika guru masih aktif mengajar (punya jadwal di `kelas_mata_pelajaran`, jadi Wali Kelas, atau punya riwayat Nilai/Pembinaan)
