# Jenis Penilaian

Modul ini menangani pengelolaan data referensi untuk Jenis-jenis Penilaian Akademik. Contoh data ini:

- Ulangan Harian (UH)
- Tugas Mandiri
- Praktikum
- Penilaian Tengah Semester (PTS)
- Penilaian Akhir Semester (PAS)

Data ini nantinya digunakan dalam menyusun skema penilaian di dalam Kurikulum (`penilaian_mata_pelajaran_kurikulum`)

**Fitur Utama**:

- CRUD Data Jenis Penilaian
- Auto-generate Kode Jenis Penilaian
- Validasi nama unik per unit sekolah

> ⚠️ LOGIC NOTe
> Copy-Paste Artifact: Sama seperti controller sebelumnya, di method `store` masih terdapat pengecekan `$data->username` yang kemungkinan adalah dead code atau sisa copy-paste dari User Controller.
>
> - Dependensi: Data ini menjadi parent/referensi bagi tabel `penilaian_mata_pelajaran_kurikulum`. Penghapusan ditolak jika data sudah digunakan dalam skema kurikulum.

---

## 1. List Jenis Penilaian

- **URL**: `GET /admin/penilaian/master/jenis-penilaian`
- **Auth**: Session
- **Controller**: `JenisPenilaianController@index`

> Description: Menampilkan daftar jenis penilaian. Mendukung server-side searching dan filtering berdasarkan Unit Sekolah.

### Parameters

| Param           | Tipe   | Wajib | Keterangan                                           |
| :-------------- | :----- | :---- | :--------------------------------------------------- |
| `draw`          | Int    | Tidak | Param Datatables                                     |
| `search[value]` | String | Tidak | Keyword pencarian global                             |
| `page`          | Int    | Tidak | Filter status hapus (`1` = Aktif, `2` = Tidak aktif) |
| `filter_unit`   | Int    | Tidak | Filter ID Unit                                       |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_jenis_penilaian": "JP-001",
      "nama_jenis_penilaian": "Ulangan Harian",
      "nama_jenis_penilaian_en": "Daily Test",
      "nama_unit": "SMA",
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Jenis Penilaian

- **URL**: `POST /admin/pendidikan/master/jenis-penilaian/store`
- **Auth**: Session
- **Controller**: `JenisPenilaianController@store`

> Description: Menyimpan data jenis penilaian baru atau memperbarui data yang ada.
> Validasi: Nama jenis harus unik dalam satu unit sekolah.

### Parameters

| Param                                 | Tipe   | Wajib    | Keterangan                   |
| :------------------------------------ | :----- | :------- | :--------------------------- |
| `id_jenis_penilaian`                  | UUID   | Optional | Isi jika mode Edit           |
| `nama_jenis_penilaian`                | String | Ya       | Nama Jenis Penilaian (id)    |
| `nama_jenis_penilaian_bahasa_inggris` | String | Ya       | Nama Jenis Penilaian (en)    |
| `status_aktif`                        | Int    | Ya       | `1` = Aktif, `2` = Non-aktif |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

### 3. Get Detail Jenis Penilaian

- **URL**: `GET /admin/pendidikan/master/jenis-penilaian/get`
- **Auth**: Session
- **Controller**: `JenisPenilaianController@get`

> Description: Mengambil detail satu data atau mencari list jenis penilaian.

### Parameters

| Param              | Tipe   | Wajib    | Keterangan             |
| :----------------- | :----- | :------- | :--------------------- |
| id_jenis_penilaian | UUID   | Optional | Get detail single data |
| search             | String | Optional | Keyword pencarian      |

---

## 4. Delete Jenis Penilaian

- **URL**: `DELETE /admin/pendidikan/master/jenis-penilaian/destroy`
- **Auth**: Session
- **Controller**: `JenisPenilaianController@destroy`

> Description: Menghapus data jenis penilaian (Soft Delete).
> Validasi: Validasi: Data tidak dapat dihapus jika sudah digunakan di tabel `penilaian_mata_pelajaran_kurikulum`.

### Parameters

| Param              | Tipe | Wajib | Keterangan                |
| :----------------- | :--- | :---- | :------------------------ |
| id_jenis_penilaian | UUID | Ya    | ID data yang akan dihapus |

---

## 5. Restore Jenis Penilaian

- **URL**: `POST /admin/pendidikan/master/jenis-penilaian/restore`
- **Auth**: Session
- **Controller**: `JenisPenilaianController@restore`

> Description: Mengembalikan data jenis penilaian yang sudah dihapus.
> Validasi: Mencegah restore jika data dengan nama/kode yang sama sudah ada di data aktif.

### Parameters

| Param              | Tipe | Wajib | Keterangan              |
| :----------------- | :--- | :---- | :---------------------- |
| id_jenis_penilaian | UUID | Ya    | ID data yang dipulihkan |
