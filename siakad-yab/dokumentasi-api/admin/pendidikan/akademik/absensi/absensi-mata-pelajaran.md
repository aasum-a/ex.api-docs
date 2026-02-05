# Modul Absensi Mata Pelajaran

Modul ini menangani pencatatan kehadiran siswa per mata pelajaran, rekapitulasi, dan pengiriman notifikasi Whatsapp ke orang tua siswa jika tidak hadir.

> ⚠️ **PERFORMANCE NOTE** Endpoint create memiliki masalah `N+1 Query`. Perlu optimasi query untuk memuat data absensi siswa secara _batch_ agar tidak membebani database saat memuat kelas dengan jumlah siswa banyak.

## 1. List Data Absensi (Datatables)

- **URL**: `GET /admin/pendidikan/akademik/absensi/mata-pelajaran`
- **Auth**: Session
- **Controller**: AbsensiMataPelajaranController@index

> Description: Menampilkan daftar riwayat absensi mata pelajaran. Endpoint ini mendukung request AJAX untuk server-side rendering datatables. Akses Data:
>
> - **Guru/Wali Kelas**: Hanya melihat data kelas yang diajar/diampu.
> - **Admin/Staff**: Melihat semua data unit terkait.

### Parameters (Query String)

| Param              | Tipe   | Wajib | Keterangan                                                     |
| :----------------- | :----- | :---- | :------------------------------------------------------------- |
| `draw`             | Int    | Tidak | Counter draw Datatables.                                       |
| `start`            | Int    | Tidak | Index awal data (Pagination).                                  |
| `length`           | Int    | Tidak | Jumlah data per halaman.                                       |
| `search[value]`    | String | Tidak | Keyword pencarian global.                                      |
| `order[0][column]` | Int    | Tidak | Index kolom untuk sorting.                                     |
| `page`             | Int    | Tidak | Filter status hapus (default logic diambil dari request page). |

### Response Example

```json
{
  "status": "success",
  "draw": 1,
  "recordsTotal": 50,
  "recordsFiltered": 50,
  "data": [
    {
      "no": 1,
      "id_kelas_mata_pelajaran": "uuid...",
      "nama_mata_pelajaran": "Matematika",
      "nama_pengajar": "Budi S.Pd",
      "tanggal": "2024-12-20",
      ...
    }
  ]
}
```

---

## 2. Form Absensi (Create/Edit View)

- **URL**: `GET /admin/pendidikan/akademik/absensi/mata-pelajaran/create`
- **Auth**: Session
- **Controller**: `AbsensiMataPelajaranController@create`

> Description: Menyiapkan data untuk form input absensi. Mengembalikan list siswa dalam kelas tersebut beserta status absensi mereka (jika sedang mengedit/membuka kembali tanggal yang sudah diabsen).

### Parameters

| Param                       | Tipe | Wajib | Keterangan                                 |
| :-------------------------- | :--- | :---- | :----------------------------------------- |
| `id_kelas_mata_pelajaran`   | UUID | Ya    | ID Kelas Mapel yang akan diabsen           |
| `id_absensi_mata_pelajaran` | UUID | Tidak | (Optional) Jika ada, berarti mode **Edit** |

### Response Example

- **Render View**: `pages...create` Data Variable:

```json
{
  "id_kelas_mata_pelajaran": "uuid...",
  "nama_mata_pelajaran": "Biologi",
  "row_absen_peserta": [
    {
      "id_peserta_didik": "uuid...",
      "nama_peserta_didik": "Andi",
      "hadir": 1, // 1: Hadir, 2: Izin, 3: Sakit, 4: Alpha
      "keterangan": "..."
    }
  ]
}
```

---

## 3. Store Absensi (Simpan Data)

- **URL**: `POST /admin/pendidikan/akademik/absensi/mata-pelajaran/store`
- **Auth**: Session
- **Controller**: `AbsensiMataPelajaranController@store`

> Description: Menyimpan header absensi dan detail kehadiran setiap siswa. Jika siswa tidak hadir (Alpha/Bolos), sistem akan men-trigger `KirimNotifikasiWhatsappJob`.

### Parameters

| Param                       | Tipe        | Wajib | Keterangan                        |
| :-------------------------- | :---------- | :---- | :-------------------------------- |
| `id_kelas_mata_pelajaran`   | UUID        | Ya    | ID Kelas Mapel                    |
| `id_absensi_mata_pelajaran` | UUID        | Tidak | Isi jika update data              |
| `tanggal`                   | Date        | Ya    | Format YYYY-MM-DD                 |
| `waktu_mulai`               | Time        | Ya    | Format HH:mm                      |
| `data`                      | JSON String | Ya    | Array object detail absensi siswa |

### Format JSON param `data`:

```json
[
  {
    "id_peserta_didik_fr": "uuid-siswa-1",
    "hadir": 1, // 1=Hadir, 2=Izin, 3=Sakit, 4=Alpha
    "keterangan": "-",
    "keterangan_en": "-"
  },
  ...
]
```

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

## 4. Delete Absensi

- **URL**: `DELETE /admin/pendidikan/akademik/absensi/mata-pelajaran/destroy`
- **Auth**: Session
- **Controller**: `AbsensiMataPelajaranController@destroy`

> Description: Menghapus data header absensi beserta seluruh detail absensi siswa terkait.

### Parameters

| Param                     | Tipe | Wajib | Keterangan                |
| :------------------------ | :--- | :---- | :------------------------ |
| id_absensi_mata_pelajaran | UUID | Ya    | ID data yang akan dihapus |

---

## 5. Get Detail Absensi (AJAX)

- **URL**: `GET /admin/pendidikan/akademik/abseensi/mata-pelajaran/get`
- **Auth**: Session
- **Controller**: `AbsensiMataPelajaranController@get`

> Description: Mengambil detail data absensi, bisa berupa detail per siswa (`type=1`) atau header absensi (`type=2`).

### Parameters

| Param                     | Tipe | Wajib | Keterangan                    |
| :------------------------ | :--- | :---- | :---------------------------- |
| id_absensi_mata_pelajaran | UUID | Ya    | ID Absensi                    |
| type                      | Int  | Ya    | `1`: Get Detail Siswa (List), |
|                           |      |       | `2`: Get Header Info          |

### Response Example

- **Type 1 (List Siswa)**:

```json
{
  "status": "success",
  "data": [ { "nama_peserta_didik": "...", "hadir": 1 }, ... ]
}
```
