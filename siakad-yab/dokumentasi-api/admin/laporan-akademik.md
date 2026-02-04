# Laporan Akademik

Modul ini digunakan untuk mencetak laporan rekapitulasi kehadiran siswa dalam rentang tanggal tertentu. Laporan ini biasanya digunakan untuk evaluasi bulanan atau pelaporan ke orang tua/yayasan.

Fitur Utama:

- Rekap Absensi Wali Kelas: Menampilkan jumlah Sakit, Izin, dan Alpha (S/I/A) per siswa dalam satu kelas.
- Rekap Absensi Mata Pelajaran: Menampilkan jumlah kehadiran siswa pada mata pelajaran tertentu.
- Export Excel: Mengunduh hasil rekapitulasi dalam format .xlsx.

> ⚠️ LOGIC NOTE
>
> Perhitungan Manual: Controller ini menghitung jumlah S/I/A secara on-the-fly (saat request) dengan query count() di dalam loop map (map(function...)).
>
> Performance Risk: Pada get_data_laporan, terdapat N+1 Query yang cukup berat karena sistem melakukan 3x Query (Count Sakit, Count Izin, Count Alpha) untuk setiap siswa di dalam list. Jika satu kelas ada 40 siswa, maka ada 120 query tambahan. Disarankan untuk menggunakan Eager Loading atau Raw SQL Group By.

---

## 1. Halaman Laporan

- **URL**: `GET /admin/pendidikan/laporan`
- **Auth**: Session
- **Controller**: `LaporanAkademikController@index`

> Description: Menampilkan halaman filter laporan. User memilih Unit, Jenis Laporan, Tanggal, dan Kelas sebelum data ditampilkan.

---

## 2. Get Data Laporan

- **URL**: `POST /admin/pendidikan/laporan/get-data-laporan`
- **Auth**: Session
- **Controller**: `LaporanAkademikController@get_data_laporan`

> Description: Mengambil data rekap absensi berdasarkan filter yang dipilih.
>
> Logic:
>
> 1. Ambil daftar siswa di kelas tersebut.
> 2. Loop setiap siswa.
> 3. Hitung jumlah kehadiran (Sakit/Izin/Alpha) berdasarkan tabel `absensi_wali_kelas` atau `absensi_mata_pelajaran` sesuai `jenis_laporan`.
> 4. Return JSON data rekap.

### Parameters

| Param                  | Tipe | Wajib    | Keterangan                                             |
| :--------------------- | :--- | :------- | :----------------------------------------------------- |
| `tanggal_awal`         | Date | Ya       | Format YYYY-MM-DD                                      |
| `tanggal_akhir`        | Date | Ya       | Format YYYY-MM-DD                                      |
| `jenis_laporan`        | Int  | Ya       | `1 = Absensi Wali Kelas`, `2 = Absensi Mata Pelajaran` |
| `id_unit`              | Int  | Ya       | ID Unit Sekolah                                        |
| `kelas_peserta_didik`  | UUID | Ya       | ID Kelas                                               |
| `kelas_mata_pelajaran` | UUID | Optional | Wajib diisi jika jenis laporan =2                      |

### Response Example

```json
{
  "status": "success",
  "data": {
    "data_absensi": [
      {
        "nis": "12345",
        "nama_peserta_didik": "Budi",
        "sakit": 1,
        "izin": 0,
        "tidak_hadir": 2
      }
    ]
  }
}
```

---

## 3. Export Excel

- URL: POST /admin/pendidikan/laporan/export-laporan
- Auth: Session
- Controller: LaporanAkademikController@export

> Description: Mengekspor data rekap yang sudah ditampilkan di preview ke dalam file Excel.
> Logic:
>
> - Menerima payload JSON `data` dari frontend (hasil preview)
> - Mengambil metadata kelas (Wali Kelas, Tahun Ajaran) untuk header laporan
> - Menggunakan `Maatwebsite\Excel` untuk generate file

### Parameters

| Param                 | Tipe        | Wajib | Keterangan                                |
| :-------------------- | :---------- | :---- | :---------------------------------------- |
| `data`                | JSON String | Ya    | Array object data rekap (S/I/A per siswa) |
| `tanggal_awal`        | Date        | Ya    | Metadata header                           |
| `tanggal_akhir`       | Date        | Ya    | Metadata header                           |
| `id_unit`             | Int         | Ya    | Metadata header                           |
| `kelas_peserta_didik` | UUID        | Ya    | Metadata header                           |
