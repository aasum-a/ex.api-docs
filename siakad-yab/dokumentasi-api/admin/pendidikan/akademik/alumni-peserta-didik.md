# Alumni Peserta Didik

Modul ini menangani manajemen data alumni, khususnya pencatatan pengambilan ijazah (transkrip nilai/dokumen kelulusan).

> ⚠️ **SECURITY/LOGIC NOTE**:
>
> 1. **Race Condition**: Tidak ada mekanisme locking saat pengecekan status ijazah. Potensi duplikasi data jika terjadi _double submit_.
> 2. **Magic Number**: Penggunaan angka hardcoded `2` untuk status pengambilan ijazah dapat mempersulit pemeliharaan kode (_maintenance_).

## 1. Halaman Index Alumni

- **URL**: `GET /admin/pendidikan/akademik/alumni-peserta-didik`
- **Auth**: Session
- **Controller**: `AlumniPesertaDidikController@index_alumni`

> Description: Menampilkan halaman daftar alumni. Jika user adalah Admin, halaman memuat daftar Unit Sekolah yang tersedia untuk filter.

### Parameters

Tidak ada paramter input (query string mungkin digunakan oleh Datatables di view, tapi tidak di-handle box di method ini).

### Response Example

- **Render View**: `pages.admin.pendidikan.akademik.alumni-peserta-didik.index`
- **Variables**: `$title`, `$units` (Collection/Array).

---

## 2 Store Pengambilan Ijazah

- **URL**: `POST /admin/pendidikan/akademik/alumni-peserta-didik/`
- **Auth**: Session
- **Controller**: `AlumniPesertaDidikController@store_pengambilan_ijazah`

> Description: Mencatat transaksi pengambilan ijazah oleh alumni/wali. Endpoint ini akan membuat record baru di tabel `pengambilan_ijazah` dan mengupdate status siswa menjadi "Sudah Diambil" (status = 2).

### Parameters

| Param                 | Tipe   | Wajib | Keterangan                                   |
| :-------------------- | :----- | :---- | :------------------------------------------- |
| `id_peserta_didik`    | UUID   | Ya    | ID Alumni yang mengambil ijazah              |
| `tanggal_pengambilan` | Date   | Ya    | Format YYYY-MM-DD                            |
| `nama_pengambil`      | String | Ya    | Nama orang yang mengambil (3 - 100 karakter) |

### Response Example

- **Success**:

```json
{
  "status": "success",
  "code": 200,
  "message": "Data berhasil disimpan"
}
```

- **Error**:

```json
{
  "status": "error",
  "code": 422,
  "message": "Data gagal disimpan",
  "data": {
    "nama_pengambil": ["The nama pengambil must be at least 3 characters."]
  }
}
```

- **Error**:

```json
{
  "status": "error",
  "code": 422,
  "message": "Siswa tersebut sudah mengambil ijazah"
}
```
