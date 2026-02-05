# Kurikulum

Modul ini menangani pengelolaan data Kurikulum Akademik. Modul ini menjadi fondasi bagi pembentukan Kelas, Mata Pelajaran, dan Penilaian. Fitur Utama:

- CRUD Data Kurikulum.
- Duplikasi Kurikulum: Fitur untuk menyalin struktur kurikulum (Mata Pelajaran & Skema Penilaian) dari Tahun Ajaran sebelumnya ke Tahun Ajaran baru.
- Validasi unik untuk mencegah duplikasi kurikulum pada kombinasi Unit, Tahun Ajaran, Jurusan, dan Tingkat yang sama.

> ⚠️ LOGIC NOTE
> Replikasi Data: Fitur `store_duplikasi` tidak hanya menyalin header kurikulum, tetapi juga men-deep copy data relasinya (`MataPelajaranKurikulum` dan `PenilaianMataPelajaranKurikulum`).
> Validasi Unik Kompleks: Menggunakan `Rule::unique` dengan klausa `where` berlapis untuk memastikan integritas data per Tahun Ajaran/Unit.

## 1. List Data Kurikulum

- **URL**: `GET /admin/pendidikan/akademik/kurikulum`
- **Auth**: Session
- **Controller**: `KurikulumController@index`

> Description: Menampilkan daftar kurikulum akademik. Mendukung server-side searching dan filtering berdasarkan Unit Sekolah, Tahun Ajaran, Tingkat, dan Jurusan.

### Parameters

| Param                  | Tipe   | Wajib | Keterangan                              |
| :--------------------- | :----- | :---- | --------------------------------------- |
| draw                   | Int    | Tidak | Param Datatables.                       |
| search[value]          | String | Tidak | Keyword pencarian global.               |
| filter_unit            | Int    | Tidak | Filter ID Unit (untuk Super Admin).     |
| filter_program_jurusan | UUID   | Tidak | Filter ID Jurusan.                      |
| filter_tingkat         | Int    | Tidak | Filter Tingkat Kelas (e.g. 10, 11, 12). |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_kurikulum": "KUR-2024-XA",
      "nama_kurikulum": "Kurikulum Merdeka X-A",
      "tahun_ajaran": "2024/2025",
      "tingkat": 10,
      "nama_program_jurusan": "IPA",
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Kurikulum (Create/Update)

- **URL**: `POST /admin/pendidikan/akademik/kurikulum/store`
- **Auth**: Session
- **Controller**: `KurikulumController@store`

> Description: Menyimpan data kurikulum baru atau memperbarui data yang ada. Validasi: Sistem mencegah pembuatan kurikulum ganda untuk kombinasi {Tahun Ajaran, Unit, Jurusan, Tingkat} yang sama.

### Parameters

| Param                   | Tipe   | Wajib    | Keterangan                            |
| :---------------------- | :----- | :------- | ------------------------------------- |
| `id_kurikulum`          | UUID   | Optional | Isi jika mode edit.                   |
| `id_tahun_ajaran_fr`    | UUID   | Ya       | ID Tahun Ajaran berlakunya kurikulum. |
| `id_program_jurusan_fr` | UUID   | Ya       | ID Program Jurusan.                   |
| `nama_kurikulum`        | String | Ya       | Nama Kurikulum.                       |
| `nama_kurikulum_en`     | String | Ya       | Nama Kurikulum (english)              |
| `tingkat`               | Int    | Ya       | Tingkat Kelas.                        |
| `sts_aktif`             | Int    | Ya       | `1` = Aktif, `2` = Nonaktif.          |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Duplikasi Kurikulum (Clone)

- **URL**: `POST /admin/pendidikan/akademik/kurikulum/store-duplikasi`
- **Auth**: Session
- **Controller**: `KurikulumController@store_duplikasi`

> Description: Menduplikasi struktur kurikulum dari satu Tahun Ajaran ke Tahun Ajaran aktif saat ini.
> Proses:
>
> 1.  Mengambil daftar kurikulum dari Tahun Ajaran asal.
> 2.  Membuat kurikulum baru di Tahun Ajaran aktif dengan atribut yang sama.
> 3.  Menyalin semua Mata Pelajaran (MataPelajaranKurikulum) terkait.
> 4.  Menyalin semua Skema Penilaian (PenilaianMataPelajaranKurikulum) terkait.

### Parameters

| Param                  | Tipe  | Wajib | Keterangan                                         |
| :--------------------- | :---- | :---- | -------------------------------------------------- |
| `id_tahun_ajaran_asal` | UUID  | Ya    | ID Tahun Ajaran sumber data.                       |
| `selected_rows`        | Array | Ya    | Array ID Kurikulum yang dipilih untuk diduplikasi. |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data Kurikulum berhasil diduplikasi"
}
```

---

## 4. Get Data Duplikasi (Helper)

- **URL**: `GET /admin/pendidikan/akademik/kurikulum/get-data-duplikasi`
- **Auth**: Session
- **Controller**: `KurikulumController@get_data_duplikasi`

> Description: Mengambil daftar kurikulum yang tersedia di Tahun Ajaran asal untuk ditampilkan dalam modal pemilihan duplikasi.

| Param                | Tipe | Wajib | Keterangan             |
| :------------------- | :--- | :---- | ---------------------- |
| id_tahun_ajaran_asal | UUID | Ya    | ID Tahun Ajaran sumber |

---

## 5. Get Detail Kurikulum

- **URL**: `GET /admin/pendidikan/akademik/kurikulum/get`
- **Auth**: Session
- **Controller**: `KurikulumController@get`

> Description: Mengambil detail satu data kurikulum atau mencari list kurikulum (Autocomplete).

### Parameters

| Param        | Tipe   | Wajib    | Keterangan              |
| :----------- | :----- | :------- | ----------------------- |
| id_kurikulum | UUID   | Optional | Get Detail Single Data. |
| search       | String | Optional | Keyword pencarian.      |

---

## 6. Delete Kurikulum (Soft Delete)

- **URL**: `DELETE /admin/pendidikan/akademik/kurikulum/destroy`
- **Auth**: Session
- **Controller**: `KurikulumController@destroy`

> Description: Menghapus kurikulum (Soft Delete).
> Validasi: Data tidak dapat dihapus jika sudah digunakan di tabel `mata_pelajaran_kurikulum` atau sudah dipetakan ke `kelas_peserta_didik`.

| Param        | Tipe | Wajib | Keterangan                      |
| :----------- | :--- | :---- | ------------------------------- |
| id_kurikulum | UUID | Ya    | ID Kurikulum yang akan dihapus. |

---

## 7. Restore Kurikulum

- **URL**: `POST /admin/pendidikan/akademik/kurikulum/restore`
- **Auth**: Session
- **Controller**: `KurikulumController@restore`

> Description: Mengembalikan kurikulum yang sudah dihapus. Validasi: Mencegah restore jika kurikulum serupa (duplikat kode) sudah ada di data aktif.

### Parameters

| Param        | Tipe | Wajib | Keterangan                    |
| :----------- | :--- | :---- | ----------------------------- |
| id_kurikulum | UUID | Ya    | ID Kurikulum yang dipulihkan. |
