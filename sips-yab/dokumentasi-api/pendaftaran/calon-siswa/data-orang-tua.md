# Data Orang Tua

Modul ini digunakan untuk memantau data akun Orang Tua/Wali yang terdaftar di sistem. Data ini terhubung dengan akun `User` (login) dan data `CalonSiswa`.

> **⚠️ PRIVACY WARNING**
> Endpoint ini menampilkan data sensitif (NIK, No HP, Penghasilan). Pastikan akses ke modul ini dibatasi hanya untuk Admin yang berwenang.

## Get List Orang Tua

Mengambil daftar orang tua dengan pagination server-side.

- **URL**: `GET /admin/pendaftaran/calon-siswa/data-orang-tua`
- **Auth**: Session Cookie

### Query Parameters

| Param              | Tipe   | Keterangan                                                           |
| :----------------- | :----- | :------------------------------------------------------------------- |
| `draw`             | Int    | Counter draw DataTables.                                             |
| `start`            | Int    | Offset data.                                                         |
| `length`           | Int    | Limit data per halaman.                                              |
| `search[value]`    | String | Keyword pencarian global.                                            |
| `filter_sts_aktif` | Int    | Filter status akun: `1` (Aktif), `0` (Nonaktif).                     |
| `page`             | Int    | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |

### Data Relations

Query utama melakukan JOIN ke tabel:

1.  `User`: Untuk mengambil username & no hp akun login.
2.  `Role`: Untuk memastikan role user.
3.  `Wilayah` (Prov, Kab, Kec): Untuk alamat.
4.  `Pendidikan`: Untuk latar belakang pendidikan.

### Response Data

JSON berisi array data orang tua, termasuk kolom virtual `avatar` yang digenerate dari storage path.
