# Mata Pelajaran Kurikulum

Modul ini menangani penyusunan mata pelajaran ke dalam suatu kurikulum spesifik. Data ini merupakan _Blueprint_ yang nantinya akan di-copy ke kelas nyata (`kelas_mata_pelajaran`) saat kelas dibuat.

Fitur Utama:

- Menambahkan mata pelajaran ke kurikulum.
- Mengatur KKM spesifik per mata pelajaran di kurikulum tersebut.
- Hard Delete: Berbeda dengan modul lain, modul ini menerapkan penghapusan permanen data jika belum digunakan di kelas.

> ⚠️ **LOGIC & CODE QUALITY NOTE**
>
> 1. **Inconsistent Deletion**: Method `destroy` menggunakan hard-delete (`delete()`), sedangkan modul lain rata-rata menggunakan soft-delete (`sts_hapus = 2`). Logic `_checIsUsed` mencegah penghapusan jika mata pelajaran ini sudah dijadwalkan di kelas nyata.
> 2. **Copy-Paste Artifact**: di Method `store`, terdapat pengecekan `$data->username`. Model `MataPelajaranKurikulum` kemungkinan besar tidak memiliki kolom `username`. Kode ini terlihat seperti sisa copy-paste dari `UserController` dan berpotensi error atau dead-code.
> 3. **Data Snapshot**: Saat data disimpan, controller menyalin (snapshot) nama mata pelajaran dan kelompok dari Master Mata Pelajaran. Jika Master Mata Pelajaran berubah nama, data di kurikulum ini **tidak otomatis berubah** (Denormalisasi).

## 1. List Mata Pelajaran dalam Kurikulum

- **URL**: `GET /admin/pendidikan/akademik/mata-pelajaran-kurikulum`
- **Auth**: Session
- **Controller**: `MataPelajaranKurikulumController@index`

> Description: Menampilkan dafter mata pelajaran yang terdaftar dalam satu ID Kurikulum tertentu.

### Parameters

| Param           | Tipe   | Wajib | Keterangan                                  |
| :-------------- | :----- | :---- | :------------------------------------------ |
| `id_kurikulum`  | UUID   | Ya    | ID Kurikulum yang ingin dilihat strukturnya |
| `draw`          | Int    | Tidak | Param Datatables                            |
| `search[value]` | String | Tidak | Keyword pencarian                           |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "nama_mata_pelajaran": "Fisika Dasar",
      "kelompok": "B (Peminatan)",
      "kkm": 75,
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Mata Pelajaran ke Kurikulum

- **URL**: `POST /admin/pendidikan/akademik/mata-pelajaran-kurikulum/store`
- **Auth**: Session
- **Controller**: `MataPelajaranKurikulumController@store`

> Description: Menambahkan mata pelajaran ke dalam kurikulum atau mengupdate data (misal: KKM) dari mata pelajaran yang sudah ada di kurikulum tersebut.
> Validasi: Sistem mencegah input ganda (Mata Pelajaran yang sama tidak boleh ada dua kali di Kurikulum yang sama).

### Parameters

| Param                       | Tipe    | Wajib    | Keterangan                        |
| :-------------------------- | :------ | :------- | :-------------------------------- |
| id_mata_pelajaran_kurikulum | UUID    | Optional | Isi jika mode edit                |
| id_kurikulum_fr             | UUID    | Ya       | ID Kurikulum tujuan               |
| id_mata_pelajaran_fr        | UUID    | Ya       | ID Master Mata Pelajaran          |
| kkm                         | Decimal | Ya       | Nilai KKM (Standar nilai minimum) |
| sts_aktif                   | Int     | Ya       | `1` = aktif, `2` = nonaktif       |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Get Detail Mata Pelajaran Kurikulum

- **URL**: `GET /admin/pendidikan/akademik/mata-pelajaran-kurikulum/get`
- **Auth**: Session
- **Controller**: `MataPelajaranKurikulumController@get`

> Description: Mengambil detail satu data mata pelajaran dalam kurikulum.

### Parameters

| Param                       | Tipe   | Wajib    | Keterangan             |
| :-------------------------- | :----- | :------- | :--------------------- |
| id_mata_pelajaran_kurikulum | UUID   | Optional | Get Detail Single Data |
| search                      | String | Optional | Keyword pencarian      |

---

## 4. Delete Mata Pelajaran dari Kurikulum

- **URL**: `DELETE /admin/pendidikan/akademik/mata-pelajaran-kurikulum/destroy`
- **Auth**: Session
- **Controller**: `MataPelajaranKurikulumController@destroy`

> Description: Menghapus mata pelajaran dari struktur kurikulum (permanent/hard-delete).
> Validasi: Data tidak dapat dihapus jika struktur ini sudah digunakan ("diturunkan") ke kelas nyata (`kelas_mata_pelajaran`).

### Parameters

| Param                       | Tipe | Wajib | Keterangan                 |
| :-------------------------- | :--- | :---- | :------------------------- |
| id_mata_pelajaran_kurikulum | UUID | Ya    | ID Data yang akan dihapus. |
