# Beasiswa Registrasi

Endpoint untuk mengelola master data beasiswa yang digunakan pada saat pendaftaran mahasiswa baru.

> **⚠️ SECURITY WARNING**
> Controller ini menggunakan **Session Based Authentication**. Tidak ada pengecekan Role/Permission eksplisit di level Controller. Pastikan Middleware Route sudah membatasi akses hanya untuk Admin/Keuangan.

## Get

Mengambil data beasiswa untuk ditampilkan dalam format JSON yang kompatibel dengan jQuery DataTables.

### Informasi Teknis

- **URL**: `/admin/keuangan/master/beasiswa-registrasi`
- **Method**: `GET`
- **Auth**: `Session Cookie`
- **Controller**: `app\Http\Controllers\Admin\Keuangan\Master\BeasiswaRegistrasiController@index`

### Query Parameters

| Param           | Tipe   | Wajib | Keterangan                                                                                                                                                                |
| :-------------- | :----- | :---- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `draw`          | Int    | Ya    | Counter draw DataTables.                                                                                                                                                  |
| `start`         | Int    | Ya    | Offset data (pagination).                                                                                                                                                 |
| `length`        | Int    | Ya    | Limit data per halaman.                                                                                                                                                   |
| `search[value]` | String | Tidak | Keyword pencarian global.                                                                                                                                                 |
| `filter_aktif`  | Int    | Tidak | Filter status aktif (1=Aktif, 2=Tidak).                                                                                                                                   |
| `page`          | Int    | Tidak | Filter status hapus (Default ambil `sts_hapus` sesuai request, tapi logic codingan agak aneh di sini: `where('sts_hapus', '=', $request->page)`). **Perlu review ulang.** |

### Response Example

```json
{
  "status": "success",
  "code": 200,
  "draw": "1",
  "recordsTotal": 50,
  "recordsFiltered": 50,
  "data": [
    {
      "id_beasiswa_registrasi": 1,
      "nama_beasiswa_registrasi": "Beasiswa Prestasi",
      "keterangan": "Potongan 50%",
      "sts_aktif": 1,
      "created_at": "admin (04-12-2025 10:00:00)"
    }
  ]
}
```

---

## Store

Endpoint ini digunakan untuk menyimpan data baru atau memperbarui data yang sudah ada.

### Informasi Teknis

- **URL**: `/admin/keuangan/master/beasiswa-registrasi/store`
- **Method**: `POST`
- **Auth**: `Session Cookie`
- **Controller**: `app\Http\Controllers\Admin\Keuangan\Master\BeasiswaRegistrasiController@store`

### Query Parameters

| Parameter                   | Tipe   | Wajib    | Keterangan                                                      |
| :-------------------------- | :----- | :------- | :-------------------------------------------------------------- |
| id_beasiswa_registrasi      | Int    | Opsional | Kunci Logika: Jika diisi = Mode Edit. Jika kosong = Mode Create |
| nama_beasiswa_registrasi    | String | Ya       | Min: 2, Max: 100.                                               |
| nama_beasiswa_registrasi_en | String | Ya       | Nama dalam Bahasa Inggris.                                      |
| keterangan                  | String | Tidak    | Max: 255.                                                       |
| prioritas_perhitungan       | Int    | Ya       | Urutan prioritas kalkulasi beasiswa.                            |
| sts_aktif                   | Int    | Ya       | `1` (Aktif) atau `2` (Tidak Aktif).                             |

## Delete & Restore

Endpoint ini digunakan untuk mengelola status penghapusan data (_Soft Delete & Restore_)

- **URL**:
  - **Destroy**: `/admin/keuangan/master/beasiswa-registrasi/destroy`
  - **Restore**: `/admin/keuangan/master/beasiswa-registrasi/restore`
  - **Force Delete** `/admin/keuangan/master/beasiswa-registrasi/force-delete`
- **Method**: `POST`
- **Auth**: `Session Cookie`
- **Controller**:
  - **Destroy**:`app\Http\Controllers\Admin\Keuangan\Master\BeasiswaRegistrasiController@destroy`
  - **Restore**: `app\Http\Controllers\Admin\Keuangan\Master\BeasiswaRegistrasiController@restore`
  - **Force Delete**: `app\Http\Controllers\Admin\Keuangan\Master\BeasiswaRegistrasiController@force-delete`

### Body Parameters

| Parameter              | Tipe | Wajib | Keterangan                 |
| :--------------------- | :--- | :---- | :------------------------- |
| id_beasiswa_registrasi | Int  | Ya    | ID data yang akan diproses |
