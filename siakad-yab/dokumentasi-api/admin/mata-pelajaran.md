# Mata Pelajaran

Modul ini menangani pengelolaan data induk Mata Pelajaran (Mapel). Data ini nantinya akan digunakan dalam penyusunan Kurikulum dan Transkrip Nilai.

**Fitur Utama:**

- CRUD Data Mata Pelajaran.
- Auto-generate Kode Mapel unik.
- Pengelompokan Mapel berdasarkan Jenis, Jurusan, dan Kelompok Wajib/Peminatan.

> ⚠️ LOGIC NOTE
> Code Generation: Kode Mata Pelajaran di-generate secara otomatis oleh sistem (generateKode()).
> Validation: Nama Mapel harus unik untuk kombinasi ID Mata Pelajaran yang sama (pada update) dalam satu unit sekolah. Validasi unik juga diterapkan pada saat restore data sampah.

## 1. List Mata Pelajaran

- **URL**: `GET /admin/pendidikan/akademik/mata-pelajaran`
- **Auth**: Session
- **Controller**: `MataPelajaranController@index`

> Description: Menampilkan daftar mata pelajaran. Mendukung server-side searching dan filtering.

### Parameters

| Param                       | Tipe   | Wajib | Keterangan                                      |
| :-------------------------- | :----- | :---- | ----------------------------------------------- |
| draw                        | Int    | Tidak | Param Datatables.                               |
| search[value]               | String | Tidak | Keyword pencarian global.                       |
| page                        | Int    | Tidak | Filter status hapus (`1 = aktif`, `2 = sampah`) |
| filter_unit                 | Int    | Tidak | Filter ID UNit (Super Admin).                   |
| filter_program_jurusan      | UUID   | Tidak | Filter Jurusan.                                 |
| filter_jenis_mata_pelajaran | UUID   | Tidak | Filter Jenis Mapel (Muatan Nasional/Lokal/dll.) |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_mata_pelajaran": "MAT-X",
      "nama_mata_pelajaran": "Matematika Umum",
      "nama_program_jurusan": "IPA",
      "kelompok": "A (Wajib)",
      "kkm": 75,
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Mata Pelajaran (Create/Update)

- **URL**: `POST /admin/pendidikan/akademik/mata-pelajaran/store`
- **Auth**: Session
- **Controller**: `MataPelajaranController@store`

> Description: Menyimpan data mapel baru atau memperbarui data mapel yang ada.

### Parameters

| Param                      | Tipe    | Wajib    | Keterangan                          |
| :------------------------- | :------ | :------- | ----------------------------------- |
| id_mata_pelajaran          | UUID    | Optional | isi jika mode edit.                 |
| id_jenis_mata_pelajaran_fr | UUID    | Ya       | ID Jenis Mapel.                     |
| id_program_jurusan_fr      | UUID    | Ya       | ID Program Jurusan.                 |
| nama_mata_pelajaran        | String  | Ya       | Nama Mata Pelajaran.                |
| nama_mata_pelajaran_en     | String  | Ya       | Nama Mata Pelajaran (English).      |
| kelompok                   | String  | Ya       | Kelompok Kelompok Mata Pelajaran.   |
| kelompok_en                | String  | Ya       | Kelompok Mata Pelajaran (English).  |
| kkm                        | Decimal | Ya       | Kriteria Ketuntasan Minimal (0-100) |
| sts_aktif                  | Int     | Ya       | `1` = aktif, `2` = nonaktif.        |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Get Detail Mata Pelajaran

- **URL**: `GET /admin/pendidikan/akademik/mata-pelajaran/get`
- **Auth**: Session
- **Controller**: `MataPelajaranController@get`

> Description: Mengambil detail satu data mata pelajaran atau mencari list mata pelajaran.

### Parameters

| Param             | Tipe   | Wajib    | Keterangan              |
| :---------------- | :----- | :------- | :---------------------- |
| id_mata_pelajaran | UUID   | Optional | Get Detail Single Data. |
| search            | String | Optional | Keyword pencarian.      |

---

## 4. Delete Mata Pelajaran

- **URL**: `DELETE /admin/pendidikan/akademik/mata-pelajaran/destroy`
- **Auth**: Session
- **Controller**: `MataPelajaranController@destroy`

> Description: Menghapus mata pelajaran.
> Validasi: Data tidak dapat dihapus jika sudah digunakan di Kurikulum (`mata_pelajaran_kurikulum`) atau sudah ada di Transkrip Nilai (transkrip_nilai).

### Parameters

| Param               | Tipe | Wajib | Keterangan                           |
| :------------------ | :--- | :---- | ------------------------------------ |
| `id_mata_pelajaran` | UUID | Ya    | ID Mata Pelajaran yang akan dihapus. |

---

## 5. Restore Mata Pelajaran

- **URL**: `POST /admin/pendidikan/akademik/mata-pelajaran/restore`
- **Auth**: Session
- **Controller**: `MataPelajaranController@restore`

> Description: Mengembalikan mata pelajaran yang sudah dihapus.
> Validasi: Mencegah restore jika mata pelajaran dengan kode/nama yang sudah ada di data aktif.

### Parameters

| Param             | Tipe | Wajib | Keterangan                         |
| :---------------- | :--- | :---- | :--------------------------------- |
| id_mata_pelajaran | UUID | Ya    | ID Mata Pelajaran yang dipulihkan. |
