# Jenis Konseling

Modul ini menangani pengelolaan data referensi untuk jenis-jenis layanan konseling (Bimbingan Konseling) yang tersedia di sekolah.

**Fitur Utama**

- CRUD Data Jenis Konseling
- Auto-generate kode jenis konseling
- validasi nama unit per unit

> ⚠️ **LOGIC NOTE**
>
> - Validation: Nama jenis konseling harus unik dalam satu unit sekolah
> - Code Generation: Kode jenis konseling di-generate otomatis oleh sistem
> - Soft Delete: Data menggunakan mekanisme soft delete

## 1. List Jenis Konseling

- **URL**: `GET /admin/pendidikan/master/jenis-konseling`
- **Auth**: Session
- **Controller**: `JenisKonselingController@index`

> Description: Menampilkan daftar jenis konseling, mendukung server-side searching dan filtering berdasarkan Unit Sekolah.

### Parameters

| Param         | Tipe   | Wajib | Keterangan                                           |
| :------------ | :----- | :---- | :--------------------------------------------------- |
| draw          | Int    | Tidak | Param Datatables                                     |
| search[value] | String | Tidak | Keyword pencarian global                             |
| page          | Int    | Tidak | Filter status hapus (`1 = aktif`, `2 = tidak aktif`) |
| filter_unit   | Int    | Tidak | Filter ID Unit                                       |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_jenis_konseling": "JK-001",
      "nama_jenis_konseling": "Bimbingan Karir",
      "nama_jenis_konseling_en": "Career Guidance",
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Jenis Konseling

- **URL**: `POST /admin/pendidikan/master/jenis-konseling/store`
- **Auth**: Session
- **Controller**: `JenisKonselingController@store`

> Description: Menyimpan data jenis konseling baru atau memperbarui data yang ada.

### Parameters

| Param                     | Tipe   | Wajib    | Keterangan                   |
| :------------------------ | :----- | :------- | :--------------------------- |
| `id_jenis_konseling`      | UUID   | Optional | Isi jika mode Edit           |
| `nama_jenis_konseling`    | String | Ya       | Nama Jenis Konseling (id)    |
| `nama_jenis_konseling_en` | String | Ya       | Nama Jenis Konseling (en)    |
| `sts_aktif`               | Int    | Ya       | `1` = aktif, `2` = non-aktif |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Get Detail Jenis Konseling

- **URL**: `GET /admin/pendidikan/master/jenis-konseling/get`
- **Auth**: Session
- **Controller**: `JenisKonselingController@get`

> Description: Mengambil detail satu data jenis konseling atau mencari list jenis konseling.

### Parameters

| Param                | Tipe   | Wajib    | Keterangan             |
| :------------------- | :----- | :------- | :--------------------- |
| `id_jenis_konseling` | UUID   | Optional | Get detail single data |
| `search`             | String | Optional | Keyword                |

---

## 4. Delete Jenis Konseling

- **URL**: `DELETE /admin/pendidikan/master/jenis-konseling/destroy`
- **Auth**: Session
- **Controller**: `JenisKonselingController@destroy`

> Description: Menghapus data jenis konseling.
> validasi: Data tidak dapat dihapus jika sudah digunakan di tabel lain (misal: Mata Pelajaran - Note: validasi \_checkIsUsed mengecek tabel mata_pelajaran, perlu dipastikan apakah relasinya benar).

### Parameters

| Param              | Tipe | Wajib | Keterangan                |
| :----------------- | :--- | :---- | :------------------------ |
| id_jenis_konseling | UUID | Ya    | ID Data yang akan dihapus |

---

## 5. Restore Jenis Konseling

- URL: `POST /admin/pendidikan/master/jenis-konseling/restore`
- Auth: Session
- Controller: `JenisKonselingController@store`

> Description: Mengembalikan data jenis konseling yang sudah dihapus.
> Validasi: Mencegah restore jika data dengan nama/kode yang sama sudah ada di data aktif.

### Parameters

| Param              | Tipe | Wajib | Keterangan              |
| :----------------- | :--- | :---- | :---------------------- |
| id_jenis_konseling | UUID | Ya    | ID Data yang dipulihkan |
