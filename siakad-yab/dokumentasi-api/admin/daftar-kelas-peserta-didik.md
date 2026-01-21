# Daftar Kelas Peserta Didik

Modul ini menangani proses _enrollment_ (pendaftaran) siswa ke dalam kelas tertentu. Modul ini juga menangani logika inisialisasi transkrip nilai dan validasi perpindahan kelas.

> ⚠️ **PERFORMANCE/ARCHITECTURE NOTE**
>
> 1. **Tight Coupling**: Controller ini memanggil method dari `KenaikanKelasController` dan `TranskripNilaiController` secara langsung. Sangat disarankan refactor ke _Service Pattern_.
> 2. **Heavy Write**: Endpoint `store` sangat berat karena otomatis men-generate _placeholder_ nilai untuk semua mata pelajaran di kelas tersebut.
> 3. **N+1 Issue**: Endpoint `get_peserta_didik` (Pencarian calon siswa) melakukan query berulang di dalam loop, diperlukan optimasi.

## 1. List Peserta Didik

- **URL**: `GET /admin/pendidikan/akademik/daftar-kelas-peserta-didik`
- **Auth**: Session
- **Controller**: `DaftarKelasPesertaDidikController@index`

> Description: Menampilkan daftar siswa yang terdaftar dalam kelas spesifik. Data mencakup status siswa (Aktif, Pindah, Keluar) dan informasi kenaikan kelas (jika ada).

### Parameters

| Param                    | Tipe   | Wajib | Keterangan                             |
| :----------------------- | :----- | :---- | :------------------------------------- |
| `id_kelas_peserta_didik` | UUID   | Ya    | ID kelas yang ingin dilihat anggotanya |
| `page`                   | Int    | Tidak | Filter status: 1 = aktif/lulus,        |
|                          |        |       | 2 = keluar/pindah.                     |
| `search[value]`          | String | Tidak | Pencarian Datatables                   |

### Response Example

- **JSON Datatables Format**:

```json
{
  "status": "success",
  "data": [
    {
      "nis": "12345",
      "nama_peserta_didik": "Budi",
      "keterangan": "Pindah Ke Kelas XI IPA 1", // Jika status pindah
      "sts_peserta": 1, // 1: Aktif
      ...
    }
  ]
}
```

## 2 Store Peserta Didik

- **URL**: `POST /admin/pendidikan/akademik/daftar-kelas-peserta-didik/store`
- **Auth**: Session
- **Controller**: `DaftarKelasPesertaDidikController@store`

> Description: Mendaftarkan satu atau banyak siswa ke dalam kelas. **Flow Logic**:
>
> 1. Validasi data input.
> 2. Cek apakah siswa sudah terdaftar di kelas lain yang aktif pada tahun ajaran yang sama (mencegah _double class_).
> 3. Insert ke tabel `daftar_kelas_peserta_didik`
> 4. Update status kelas aktif di tabel master `peserta_didik`
> 5. **Heavy Task**: Loop semua Mapel di kelas tersebut, lalu insert data kosong ke tabel `transkrip_nilai_ via `TranskripNilaiController`.

### Parameters

| Param                    | Tipe        | Wajib | Keterangan                                |
| :----------------------- | :---------- | :---- | :---------------------------------------- |
| `id_kelas_peserta_didik` | UUID        | Ya    | ID Kelas tujuan.                          |
| `data`                   | JSON String | Ya    | Array object siswa yang akan ditambahkan. |

### Format JSON `data`:

```json
[
  {
    "id_peserta_didik": "uuid...",
    "nama_peserta_didik": "Budi", // Info pelengkap untuk error msg
    "nva": "...",
    "kelas_sebelum": "X IPA 1"
  }
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

## Get Peserta Didik

- **URL**: `GET /admin/pendidikan/akademik/daftar-kelas_peserta_didik/get_peserta_didik`
- **Auth**: Session
- **Controller**: `DaftarKelasPesertaDidikController@get_peserta_didik`

> Description: Mengambil daftar kandidat siswa yang bisa dimasukkan ke dalam kelas tersebut. Kandidat berasal dari tiga sumber:
>
> 1. Tinggal Kelas: Siswa tingkat yang sama dari tahun lalu yang tidak naik kelas.
> 2. Naik Kelas: Siswa tingkat sebelumnya (tingkat 1) yang dinyatakan naik kelas.
> 3. Siswa Baru: Siswa baru yang belum pernah masuk kelas manapun.

### Parameters

| Param                  | Tipe | Wajib | Keterangan                                     |
| :--------------------- | :--- | :---- | :--------------------------------------------- |
| id_kelas_peserta_didik | UUID | Ya    | ID kelas tujuan (untuk cek kurikulum/tingkat). |
| id_kurikulum           | UUID | Ya    | ID Kurikulum kelas tersebut.                   |

### Response Example

```json
{
  "status": "ok",
  "code": 200,
  "data": [
    {
      "id_peserta_didik": "uuid...",
      "nama_peserta_didik": "Andi",
      "kelas_sebelum": "X IPA 1 (Naik Kelas)" // atau "Siswa Baru"
    }
  ]
}
```

## 4. Hapus Peserta Didik

- **URL**: `DELETE /admin/pendidikan/akademik/daftar-kelas-peserta-didik/destroy`
- **Auth**: Session
- **Controller**: `DaftarKelasPesertaDidikController@Destroy`

> Description: Mengeluarkan siswa dari kelas (_soft-delete_: `sts_hapus  = 2`). **Warning**: validasi relasi data (`_checkIsUsed`) saat ini dinonaktifkan (selalu return false), sehingga penghapusan bisa dilakukan paksa meskipun data sudah berelasi dengan nilai/absensi.

### Parameters

| Param                         | Tipe | Wajib | Keterangan                              |
| :---------------------------- | :--- | :---- | :-------------------------------------- |
| id_daftar_kelas_peserta_didik | UUID | Ya    | ID Record enrollment yang akan dihapus. |

## 5. Detail Infor Kelas

- **URL**: `GET /admin/pendidikan/akademik/daftar-kelas-peserta-didik/get`
- **Auth**: Session
- **Controller**: `DaftarKelasPesertaDidik@get`

> Description: Mengambil detail informasi kelas peserta didik (header info) untuk ditampilkan di bagian atas halaman detail kelas.

### Parameters

| Param                  | Tipe | Wajib | Keterangan |
| :--------------------- | :--- | :---- | :--------- |
| id_kelas_peserta_didik | UUID | Ya    | ID Kelas.  |
