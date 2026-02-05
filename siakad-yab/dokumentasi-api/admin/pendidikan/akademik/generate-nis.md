# Generate NIS

Modul ini berfungsi untuk men-generate Nomor Induk Siswa (NIS) secara otomatis berdasarkan format Tahun Ajaran dan Kode Kelas.

> **TECHNICAL DEBT & RISK NOTE**
>
> 1. **Race Condition**: Penentuan nomor urut dilakukan di sisi aplikasi saat _read-time_, bukan _write-time_. hal itu rentan duplikasi jika diakses secara bersamaan.
> 2. **Fragile Logic**: Parsing nomor urut menggunakan `substr(..., -3)` yang sangat bergantung pada format _hardcoded_ 3 digit terakhir. Perubahan format NIS akan mematahkan fitur ini.
> 3. **Anti-Pattern**: Controller memanggil method controller lain secara langsung (`new Controller()`)

## 1. List Kelas untuk Generate NIS

- **URL**: `GET /admin/pendidikan/akademik/generate-nis`
- **Auth**: Session
- **Controller**: `GenerateNISController@index`

> Description: Menampilkan daftar kelas yang memiliki siswa tanpa NIS. Endpoint ini me-reuse logic dari `KelasPesertaDidikController` dan `DaftarKelasPesertaDidikController` (via direct instantiation).

### Parameters

| Param        | Tipe    | Wajib | Keterangan                                                             |
| :----------- | :------ | :---- | :--------------------------------------------------------------------- |
| validasi_nis | Boolean | Tidak | (Internal) digunakan untuk filter kelas yang siswanya belum punya NIS. |

### Response Example

- **Render View**: `pages...generate-nis.index`
- **Data**: List Kelas.

## 2. Preview Generate NIS

- **URL**: `GET /admin/pendidikan/akademik/generate-nis/create`
- **Auth**: Session
- **Controller**: `GenerateNISController@create`

> Description: Menghitung dan menampilkan preview NIS untuk siswa dalam kelas terpilih yang belum memiliki NIS. **Format NIS**: `[2 Digit Tahun Awal][2 Digit Tahun Akhir][Kode NIS Kelas][3 Digit No Urut]` Contoh: `2425100001` (TA 2024/2025, Kode Kelas 10, Urut 001).

### Parameters

| Param                  | Tipe | Wajib | Keterangan                  |
| :--------------------- | :--- | :---- | :-------------------------- |
| id_kelas_peserta_didik | UUID | Ya    | ID Kelas yang akan diproses |

### Response Example

- **View Data Variable**:

```json
{
  "title": "Generate NIS Kelas X-A",
  "data_siswa": [
    {
      "id": "uuid-siswa-1",
      "nis": "242510001", // Generated Preview
      "nama_peserta_didik": "Ahmad",
      "tempat_lahir": "Bandung",
      "tanggal_lahir": "2008-01-01"
    },
    ...
  ]
}
```

## 3. Store Generated NIS

- **URL**: `POST /admin/pendidikan/akademik/generate-nis/store`
- **Auth**: Session
- **Controller**: `GenerateNISController@store`

> Description: Menyimpan NIS yang telah disetujui/diedit oleh user ke dalam database.
> **Flow**:
>
> 1. Validasi request.
> 2. Loop array data siswa.
> 3. Verifikasi apakah siswa benar-benar anggota kelas tersebut (mencegah IDOR/Manipulation).
> 4. Update kolom `nis` di tabel `peserta_didik`.

### Parameters

| Param                  | Tipe        | Wajib | Keterangan                      |
| :--------------------- | :---------- | :---- | :------------------------------ |
| id_kelas_peserta_didik | UUID        | Ya    | ID Kelas konteks saat ini.      |
| data                   | JSON String | Ya    | Array object siswa dan NIS baru |

### Format JSON `data`:

```json
[
  {
    "id_peserta_didik": "uuid...",
    "nis": "242510001",
    "nama_peserta_didik": "Ahmad", // Utk pesan error
    "tempat_lahir": "...", // Utk pesan error
    "tanggal_lahir": "..." // Utk pesan error
  }
]
```

### Response Example:

- **Success**:

```json
{
  "status": "success",
  "code": 200,
  "message": "Data berhasil disimpan"
}
```

- **Error (Siswa Tidak Ditemukan di Kelas)**:

```json
{
  "status": "error",
  "code": 422,
  "message": "Data siswa dengan ID Peserta Didik [Nama Anak],2008-01-01 tidak ditemukan di kelas ini."
}
```
