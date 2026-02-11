# Kelas

Modul ini menangani pengelolaan data induk Ruang Kelas (Rombel). Data ini merepresentasikan entitas fisik atau logis dari sebuah kelas (misal: "X IPA 1", "X IPS 2", "Asrama Putra A").

Fitur Utama:

- CRUD Data Kelas
- Auto-generate Kode Kelas
- Validasi unik kombinasi Tingkat dan Kelompok Kelas dalam satu Unit.

> ⚠️ **LOGIC NOTE**
>
> - Konsep Kelas: Data ini adalah _Master Data_ (Template Kelas), bukan _Kelas Aktif_ per Tahun Ajaran. Kelas Aktif (yang ada siswanya) dikelola di modul `KelasPesertaDidik`.
> - Code Generation: Kode Kelas di-generate otomatis.
> - Soft Delete: Data menggunakan mekanisme _soft delete_ untuk menjaga integritas histori.

---

## 1. List Ruang Kelas

- **URL**: `GET /admin/pendidikan/master/kelas`
- **Auth**: Session
- **Controller**: `KelasController@index`

> Description: Menampilkan daftar ruang kelas. Mendukung server-side searching dan filtering.

### Parameters

| Param            | Tipe   | Wajib | Keterangan                                       |
| :--------------- | :----- | :---- | :----------------------------------------------- |
| `draw`           | Int    | Tidak | Param Datatables                                 |
| `search[value]`  | String | Tidak | Keyword Pencarian                                |
| `page`           | Int    | Tidak | Filter status hapus (1 = aktif, 2 = tidak aktif) |
| `filter_unit`    | Int    | Tidak | Filter ID Unit                                   |
| `filter_tingkat` | Int    | Tidak | Filter Tingkat Kelas                             |

### Respone Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_kelas": "KLS-001",
      "tingkat": 10,
      "kelompok_kelas": "IPA 1",
      "kode_nis": "101",
      "sts_aktif": 1
    }
  ]
}
```

---
