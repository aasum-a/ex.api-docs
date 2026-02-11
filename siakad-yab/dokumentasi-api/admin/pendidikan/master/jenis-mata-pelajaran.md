# Jenis Mata Pelajaran

Modul ini menangani pengelolaan data referensi untuk Kelompok/Jenis Mata Pelajaran. Contaoh data ini biasanya:

- Muatan Nasional (A)
- Muatan Kewilayahan (B)
- Muatan Peminatan Kejuruan (C)
- Dasar Bidang Keahlian (C1)
- dll.

Data ini digunakan untuk mengelompokkan Mata Pelajaran agar terstruktur rapor sesuai dengan standar kurikulum nasional.

**Fitur Utama**:

- CRUD Data Jenis Mata Pelajaran
- Auto-generate kode Jenis Mata Pelajaran
- Validasi nama unit per unit sekolah

> ⚠️ LOGIC & CODE NOTE
>
> - Copy-paste Artifact: di method `store`, terdapat pengecekan `$data->username`. Model `JenisMataPelajaran` kemungkinan besar tidak memiliki kolom `username`. Kode ini sepertinya sisa copy-paste dari User/Pengajar Controller dan berpotensi menjadi _dead code_ atau error jika properti tersebut diakses secara strict.
> - Dependensi: Data ini menjadi parent/referensi bagi tabel `mata_pelajaran`. Penghapusan ditolak jika data sudah digunakan.

## 1. List Jenis Mata Pelajaran

- **URL**: `GET /admin/pendidikan/master/jenis-mata-pelajaran`
- **Auth**: Session
- **Controller**: `JenisMataPelajaranController@index`

> Description: Menampilkan daftar jenis mata pelajaran. Mendukung server-side searching dan filtering berdasarkan Unit Sekolah.

### Parameters

| Param           | Tipe   | Wajib | Keterangan                                     |
| :-------------- | :----- | :---- | :--------------------------------------------- |
| `draw`          | Int    | Tidak | Param Datatables                               |
| `search[value]` | String | Tidak | Keyword pencarian global                       |
| `page`          | Int    | Tidak | Filter status (`1` = Aktif, `2` = Tidak Aktif) |
| `filter_unit`   | Int    | Tidak | Filter ID Unit                                 |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_jenis_mata_pelajaran": "JMP-001",
      "nama_jenis_mata_pelajaran": "Muatan Nasional",
      "nama_jenis_mata_pelajaran_en": "National Content",
      "nama_unit": "SMK Bakti",
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Jenis Mata Pelajaran

- **URL**: `POST /admin/pendidikan/master/jenis-mata-pelajaran/store`
- **Auth**: Session
- **Controller**: `JenisMataPelajaranController@store`

> Description: Menyimpan data jenis mata pelajaran baru atau memperbarui data yang ada.
> Validasi: Nama jenis mata pelajaran harus unik dalam satu unit sekolah.

### Parameters

| Param                        | Tipe   | Wajib    | Keterangan                     |
| :--------------------------- | :----- | :------- | :----------------------------- |
| id_jenis_mata_pelajaran      | UUID   | Optional | Isi jika mode Edit             |
| nama_jenis_mata_pelajaran    | String | Ya       | Nama Jenis Mata Pelajaran (id) |
| nama_jenis_mata_pelajaran_en | String | Ya       | Nama Jenis Mata Pelajaran (en) |
| status_aktif                 | Int    | Ya       | `1` = Aktif, `2` = Non-aktif   |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Get Detail Jenis Mata Pelajaran

- **URL**: `GET /admin/pendidikan/master/jenis-mata-pelajaran/get`
- **Auth**: Session
- **Controller**: `JenisMataPelajaranController@get`

> Description: Mengambil detail satu data atau mencari list jenis mata pelajaran

### Parameters

| Param                   | Tipe   | Wajib    | Keterangan             |
| :---------------------- | :----- | :------- | :--------------------- |
| id_jenis_mata_pelajaran | UUID   | Optional | Get detail single data |
| search                  | String | Optional | Keyword pencarian      |

---

## 4. Delete Jenis Mata Pelajaran

- **URL**: `DELETE /admin/pendidikan/master/jenis-mata-pelajaran/destroy`
- **Auth**: Session
- **Controller**: `JenisMataPelajaranController@destroy`

> Description: Menghapus data jenis mata pelajaran.
> Validasi: Data tidak dapat dihapus jika sudah digunakan di tabel `mata_pelajaran`.

### Parameters

| Param                   | Tipe | Wajib | Keterangan                |
| :---------------------- | :--- | :---- | :------------------------ |
| id_jenis_mata_pelajaran | UUID | Ya    | ID Data yang akan dihapus |

---

## 5. Restore Jenis Mata Pelajaran

- **URL**: `POST /admin/pendidikan/master/jenis-mata-pelajaran/restore`
- **Auth**: Session
- **Controller**: `JenisMataPelajaranController@restore`

> Description: Mengembalikan data jenis mata pelajaran yang sudah dihapus.
> Validasi: Mencegah restore jika data dengan nama/kode yang sama sudah ada di data aktif.

### Parameters

| Param                   | Tipe | Wajib | Keterangan              |
| :---------------------- | :--- | :---- | :---------------------- |
| id_jenis_mata_pelajaran | UUID | Ya    | ID data yang dipulihkan |
