# Master Feeder

Modul ini bersifat Read-Only (untuk pengguna umum) yang menyediakan data referensi standar nasional, seperti Wilayah, Agama, Jenis Tinggal, dan Jenjang Pendidikan.
Fitur Utama:

- Wilayah Bertingkat: Mengambil data Provinsi, Kabupaten, Kecamatan, hingga Kelurahan secara hierarkis (_Parent-Child_).
- Autocomplete: Menyediakan endpoint pencarian (_search_) untuk integrasi dengan library dropdown seperti Select2.

> ⚠️ LOGIC NOTE
> Struktur Wilayah: Data wilayah menggunakan relasi Self-Join pada tabel data_wilayah.
>
> - Level 1 = Provinsi
> - Level 2 = Kabupaten (Parent = ID Provinsi)
> - Level 3 = Kecamatan (Parent = ID Kabupaten)
>   Public Access: Controller ini umumnya diakses secara publik atau dengan autentikasi minimal karena datanya bersifat umum.

## 1. Get Data Wilayah (Provinsi/Kab/Kec)

- **URL**: `GET /admin/pendidikan/master/data/wilayah`
- **Auth**: Session
- **Controller**: `DataController@get_data_wilayah`

> Description: Mengambil daftar wilayah.
>
> - Jika parameter `id` kosong, mengembalikan daftar Provinsi (Level 1).
> - Jika parameter `id` diisi (misal ID Jawa Barat), mengembalikan daftar anak wilayahnya (misal: Bandung, Bogor, dst).

### Parameters

| Param | Tipe | Wajib | Keterangan                                                 |
| :---- | :--- | :---- | :--------------------------------------------------------- |
| `id`  | UUID | Tidak | ID Wilayah Induk (Parent). Kosongkan untuk ambil Provinsi. |

### Response Example

```json
{
  "status": "ok",
  "data": [
    {
      "id_data_wilayah": "uuid...",
      "nm_wil": "Jawa Barat",
      "id_level_wil": 1
    }
  ]
}
```

2. Get Data Pendidikan

- **URL**: `GET /admin/pendidikan/master/data/pendidikan`
- **Auth**: Session
- **Controller**: `DataController@get_data_pendidikan`

> Description: Mengambil daftar jenjang pendidikan (SD, SMP, SMA, S1, dll) untuk data orang tua/wali.

### Parameters

| Param                | Tipe   | Wajib    | Keterangan                |
| :------------------- | :----- | :------- | :------------------------ |
| `id_data_pendidikan` | UUID   | Optional | Ambil satu data spesifik. |
| `search`             | String | Optional | Keyword pencarian.        |

3. Get Data Agama

- **URL**: `GET /admin/pendidikan/master/data/agama`
- **Auth**: Session
- **Controller**: `DataController@get_data_agama`

> Description: Mengambil daftar agama resmi.

### Parameters

| Param           | Tipe   | Wajib | Keterangan                |
| :-------------- | :----- | :---- | :------------------------ |
| `id_data_agama` | UUID   | Opt   | Ambil satu data spesifik. |
| `search`        | String | Opt   | Keyword pencarian.        |

4. Get Jenis Tinggal
   URL: GET /admin/pendidikan/master/data/jenis-tinggal
   Auth: Session
   Controller: DataController@get_data_jenis_tinggal

> Description; Mengambil daftar jenis tempat tinggal (Bersama Orang Tua, Wali, Kos, Asrama, dll).

### Parameters

| Param                   | Tipe   | Wajib | Keterangan                |
| :---------------------- | :----- | :---- | :------------------------ |
| `id_data_jenis_tinggal` | UUID   | Opt   | Ambil satu data spesifik. |
| `search`                | String | Opt   | Keyword pencarian.        |
