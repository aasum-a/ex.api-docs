# Beasiswa Calon Siswa

Modul ini digunakan oleh Admin Keuangan untuk melihat daftar siswa yang mengajukan beasiswa, memeriksa detail dokumen, dan memutuskan apakah pengajuan tersebut diterima atau ditolak.

---

## Lihat Pengajuan

Mengambil daftar pengajuan beasiswa dalam format JSON yang kompatibel dengan **DataTables** (Server-side processing).

### Informasi Teknis

- **URL**: `/admin/keuangan/beasiswa-calon-siswa`
- **Method**: `GET`
- **Auth**: `Session Cookie`
- **Controller**: `Admin\Keuangan\CalonSiswa\BeasiswaCalonSiswaController@index`

### Query Parameters

Selain parameter standar DataTables, endpoint ini menerima filter khusus:

| Parameter              | Tipe | Opsional? | Keterangan                                                 |
| :--------------------- | :--- | :-------- | :--------------------------------------------------------- |
| `filter_sts_pengajuan` | Int  | Ya        | Filter status.<br>`1`=Pending, `2`=Approved, `3`=Rejected. |
| `id_tahun_ajaran_fr`   | Int  | Ya        | Filter berdasarkan ID Tahun Ajaran tertentu.               |

### Query Parameters (DataTables Standard)

| Parameter          | Tipe   | Keterangan                      |
| :----------------- | :----- | :------------------------------ |
| `draw`             | Int    | Counter request (wajib).        |
| `start`            | Int    | Index awal data (Pagination).   |
| `length`           | Int    | Jumlah data per halaman.        |
| `search[value]`    | String | Keyword pencarian (Nama Siswa). |
| `order[0][column]` | Int    | Kolom yang di-sort.             |

### Response

**Sukses (`200 OK`)**

```json
{
  "status": "success",
  "code": 200,
  "draw": 1,
  "recordsTotal": 50,
  "recordsFiltered": 50,
  "data": [
    {
      "id_beasiswa_calon_siswa": "UUID-1",
      "nama_siswa": "Andy Siswa",
      "nama_kelompok_beasiswa": "Beasiswa Prestasi",
      "sts_pengajuan": 1,
      "tgl_pengajuan": "2025-01-01 10:00:00",
      "count_pending": 1,
      "count_approved": 0,
      "count_rejected": 0
    }
  ]
}
```

---

## Lihat Pengajuan Detail

Endpoint ini digunakan untuk mengambil detail lengkap satu pengajuan beasiswa, termasuk dokumen pendukung, berdasarkan ID Siswa.

> ⚠️ Catatan Penting: Endpoint in menggunakan **Foreign Key** (id_calon_siswa_fr), BUKAN Primary Key tabel beasiswa

### Informasi Teknis

- **URL**: `/admin/keuangan/beasiswa-calon-siswa/lihat-pengajuan`
- **Method**: `GET`
- **Auth**: `Session Cookie`
- **Controller**: `Admin\Keuangan\CalonSiswa\BeasiswaCalonSiswaController@lihatPengajuan`

### Query Parameters

| Parameter         | Tipe | Wajib? | Keterangan                                        |
| :---------------- | :--- | :----- | :------------------------------------------------ |
| id_calon_siswa_fr | UUID | Ya     | ID siswa (Foreign Key) yang ingin dilihat datanya |

### Response

**Success (`200 OK`)**

```json
{
    "status": "success",
    "code": 200,
    "message" "Data berhasil ditemukan",
    "data": [
        {
            "id_beasiswa_calon_siswa": "UUID-Primary",
            "beasiswa": { "id": 1, "nama": "Beasiswa Prestasi" },
            "nama_user": "Nama Siswa",
            "keterangan": "Kurang mampu, melampirkan SKTM",
            "file": "path/to/sktm.pdf",
            "sts_pengajuan": 1,
            "nama_dokumen": "Kartu Keluarga"
        }
    ]
}
```

---

## Update Status

Endpoint ini digunakan untuk menyetujui atau menolak pengajuan beasiswa.

### Informasi Teknis

- **URL**: `/admin/keuangan/beasiswa-calon-siswa/update-status`
- **Method**: `POST`
- **Auth**: `Session Cookie`
- **Controller**: `Admin\Keuangan\CalonSiswa\BeasiswaCalonSiswaController@updateStatus`

### Query Parameters

| Parameter               | Tipe | Wajib? | Keterangan                                                       |
| :---------------------- | :--- | :----- | :--------------------------------------------------------------- |
| id_beasiswa_calon_siswa | UUID | Ya     | ID Transaksi Pengajuan                                           |
| sts_pengajuan           | int  | Ya     | Status Keputusan (`1` = Pending, `2` = Approved, `3` = Rejected) |

### Response

**Success (`200 OK`)**

```json
{
  "status": "success",
  "code": 200,
  "message": "Status beasiswa berhasil diperbarui"
}
```

**Error Validasi (`422 Unprocessible`) Jika ID tidak ditemukan atau status yang dikirim bukan 1, 2, atau 3.**

```json
{
  "status": "error",
  "code": 422,
  "message": "Data gagal divalidasi",
  "data": {
    "sts_pengajuan": ["The selected sts pengajuan is invalid."]
  }
}
```
