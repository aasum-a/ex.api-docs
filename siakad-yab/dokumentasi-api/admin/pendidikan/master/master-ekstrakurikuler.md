# Master Ekstrakurikuler

Modul ini menangani pengelolaan data induk Ekstrakurikuler yang tersedia di sekolah.
Fitur Utama:

- CRUD Data Ekstrakurikuler
- Aut-generate Kode Ekstrakurikuler
- Pencatatan Nama Pembina Ekstrakurikuler

> ⚠️ LOGIC NOTE:
>
> - Code Generation: Kode Ekskul di-generate secara otomatis oleh sistem (Ekskul::generateKode()).
> - Validation: Nama Ekskul harus unik dalam satu unit sekolah yang sama.
> - Soft Delete: Penghapusan data bersifat soft delete. Data tidak bisa dihapus permanen jika sudah ada riwayat siswa yang mengikuti ekskul tersebut.

## 1. List Ekstrakurikuler

- **URL**: `GET /admin/pendidikan/master/ekskul`
- **Auth**: Session
- **Controller**: `EkskulController@index`

> Description: Menampilkan kegiatan ekstrakurikuler. Mendukung server-side searching dan filtering berdasarkan Unit Sekolah.

### Parameter

| Param         | Tipe   | Wajib | Keterangan                                       |
| :------------ | :----- | :---- | :----------------------------------------------- |
| draw          | Int    | Tidak | Param Datatables                                 |
| search[value] | String | Tidak | Keyword pencarian global                         |
| page          | Int    | Tidak | Filter status hapus (1 = Aktif, 2 = Tidak Aktif) |
| filter_unit   | Int    | Tidak | Filter ID Unit (untuk Super Admin)               |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_ekskul": "EKS-001",
      "nama_ekskul": "Pramuka",
      "nama_ekskul_en": "Scout",
      "pembina": "Budi Santoso",
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Ekstrakurikuler

- **URL**: `POST /admin/pendidikan/master/ekskul/store`
- **Auth**: Session
- **Controller**: `EkskulController@store`

> Description: Menyimpan data ekstrakurikuler atau memperbarui data yang ada.

### Parameters

| Param                                 | Tipe   | Wajib | Keterangan                        |
| :------------------------------------ | :----- | :---- | :-------------------------------- |
| `id_ekskul`                           | UUID   | Opt   | Isi jika mode Edit                |
| `nama_ekstrakurikuler`                | String | Ya    | Nama Ekstrakurikuler              |
| `nama_ekstrakurikuler_bahasa_inggris` | String | Ya    | Nama Ekstrakurikuler (Inggris)    |
| `pembina`                             | String | Ya    | Nama Guru Pembina Ekstrakurikuler |
| `status_aktif`                        | Int    | Ya    | `1 = Aktif`, `2 = Nonaktif`       |

## Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Get Detail Ekstrakurikuler

- **URL**: `GET /admin/pendidikan/master/ekskul/get`
- **Auth**: Session
- **Controller**: `EkskulController@get`

> Description: Mengambil detail satu data ekstrakurikuler atau mencari list ekstrakurikuler.

### Parameters

| Param     | Tipe   | Wajib    | Keterangan             |
| :-------- | :----- | :------- | :--------------------- |
| id_ekskul | UUID   | Optional | Get Detail Single Data |
| search    | String | Optional | Keyword pencarian      |

---

## 4. Delete Ekstrakurikuler

- **URL**: `DELETE /admin/pendidikan/master/ekskul/destroy`
- **Auth**: Session
- **Controller**: `EkskulController@destroy`

> Description: Menghapus data ekstrakurikuler secara _soft delete_.
> Validasi: Data tidak dapat dihapus jika sudah digunakan di tabel `riwayat_ekskul` (sudah ada siswa yang terdaftar/mengikuti).

### Parameters

| Param     | Tipe | Wajib | Keterangan                  |
| :-------- | :--- | :---- | :-------------------------- |
| id_ekskul | UUID | Ya    | ID Ekskul yang akan dihapus |

---

## 5. Restore Ekstrakurikuler

- **URL**: `POST /admin/pendidikan/master/ekskul/restore`
- **Auth**: Session
- **Controller**: `EkskulController@restore`

> Description: Mengembalikan data ekskul yang sudah dihapus.
> Validasi: Mencegah restore jika ekstrakurikuler dengan nama/kode yang sama sudah ada di data aktif.

### Parameters

| Param     | Tipe | Wajib | Keterangan                |
| :-------- | :--- | :---- | :------------------------ |
| id_ekskul | UUID | Ya    | ID Ekskul yang dipulihkan |
