# Kenaikan Kelas & Kelulusan

Modul ini menangani proses transisi siswa antar tingkat (naik kelas), tinggal kelas, atau kelulusan (Alumni).

> ⚠️ **CRITICAL LOGIC NOTE**
>
> 1. Dependency Heavy: Modul ini sangat bergantung pada controller lain (`TranskripNilai`, `User`, `DaftarKelas`).
>    Kegagalan pada salah satu controller tersebut akan menggagalkan proses kenaikan kelas.
> 2. Strict Rollback: Fitur `batal_kenaikan` melakukan penghapusan data fisik (hard delete) pada Transkrip dan Daftar Kelas. Sistem menolak pembatalan jika data Nilai atau Keuangan sudah terisi di kelas tujuan.
> 3. Hardcoded Levels: Validasi kelulusan saat ini dikunci hanya untuk tingkat 9 dan 12.

## 1. List Riwayat Kenaikan

- **URL**: `GET /admin/pendidikan/akademik/kenaikan-kelas`
- **Auth**: Session
- **Controller**: `KenaikanKelasController@index`

> Description: Menampilkan riwayat proses kenaikan kelas yang pernah dilakukan. Jika dipanggil via AJAX, endpoint ini akan me-reuse logic dari `DaftarKelasPesertaDidikController@index` untuk menampilkan detail siswa dalam satu batch kenaikan.

---

## 2. Form Kenaikan

- **URL**: `GET /admin/pendidikan/akademik/kenaikan-kelas/get_data_kelas_baru`
- **Auth**: Session
- **Controller**: `KenaikanKelasController@get_data_kelas_baru`

> Description: Menyiapkan data siswa untuk diproses naik kelas.
> Logic:
>
> 1. Mengambil daftar siswa dari kelas asal.
> 2. Mencari Tahun Ajaran berikutnya.
> 3. Menandai siswa yang sudah mencapai tingkat akhir (untuk opsi Lulus).

### Parameters

| Param                  | Tipe | Wajib | Keterangan    |
| :--------------------- | :--- | :---- | :------------ |
| id_kelas_peserta_didik | UUID | Ya    | ID Kelas asal |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "id_peserta_didik": "uuid...",
      "nama_peserta_didik": "Budi",
      "tingkat": 10,
      "kenaikan": true // True jika bisa naik ke 11
    }
  ]
}
```

## 3. Eksekusi Kenaikan (Store)

- **URL**: `POST /admin/pendidikan/akademik/kenaikan-kelas/store`
- **Auth**: Session
- **Controller**: `KenaikanKelasController@store`

> Description: Memproses kenaikan kelas, atau kelulusan secara massal.
> Core Workflow:
>
> 1. Validasi Input.
> 2. Membuat record `kenaikan_kelas`
> 3. Loop setiap siswa:
>
> - **Jika Naik/Tinggal Kelas**:
> - Validasi apakah sudah ada di kelas tujuan.
> - Insert ke `daftar_kelas_peserta_didik` baru.
> - Generate Transkrip Nilai kosong untuk mapel di kelas baru.
> - Update staus aktif siswa di master `peserta_didik` (Update ID Kelas Aktif)
>
> - **Jika Lulus**:
> - Validasi apakah siswa berada di tingkat akhir.
> - Update status siswa menjadi "Lulus" (Status = 4) di master `peserta_didik`.
> - Hapus referensi kelas aktif.
>
> - **Jika Keluar/Pindah**: (Hanya update status di data kelas sebelumnya).
>
> 4. Update status data kelas sebelumnya (History).

### Parameters

| Param                    | Tipe        | Wajib | Keterangan                     |
| :----------------------- | :---------- | :---- | :----------------------------- |
| `id_kelas_peserta_didik` | UUID        | Ya    | ID Kelas Asal                  |
| `data`                   | JSON String | Ya    | Array detail status tiap siswa |

### Format JSON `data`:

```json
[
  {
    "id_peserta_didik_fr": "uuid...",
    "status_next": 2, // 2=Naik, 3=Tinggal, 6=Lulus
    "kelas_next": "uuid-kelas-tujuan", // Required jika Naik/Tinggal
    "keterangan": "Naik bersyarat"
  }
]
```

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data kenaikan kelas berhasil disimpan"
}
```

---

## 4. Pembatalan Kenaikan

- **URL**: `POST /admin/pendidikan/akademik/kenaikan-kelas/batal-kenaikan`
- **Auth**: Session
- **Controller**: `KenaikanKelasController@batal_kenaikan`

> Description: Membatalkan status kenaikan kelas untuk satu siswa.
> Validasi: Pembatalan ditolak jika di kelas tujuan sudah memiliki:
>
> 1. Data Pembayaran
> 2. Data Transkrip Nilai (yang sudah diisi angka)
> 3. Data Absensi (Mapel/Wali Kelas).
>
> **Jika Valid**:
>
> 1. Hapus data `daftar_kelas_peserta_didik` di kelas tujuan.
> 2. Hapus data `transkrip_nilai` di kelas tujuan.
> 3. Kembalikan status siswa ke kelas asal.
> 4. Update status `kenaikan_kelas_detail` menjadi `7` (dibatalkan).

### Parameters

| Param                    | Tipe | Wajib | Keterangan                   |
| :----------------------- | :--- | :---- | :--------------------------- |
| id_kenaikan_kelas_detail | UUID | Ya    | ID Detail Transaksi Kenaikan |
