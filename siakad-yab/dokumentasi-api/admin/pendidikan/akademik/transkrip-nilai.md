# Transkrip Nilai

Modul ini menangani penginputan nilai mata pelajaran untuk setiap siswadi dalam suatu kelas. Nilai yang diinput akan membentuk Transkrip Nilai per Semester.

> Fitur Utama:
>
> - Menampilkan daftar nilai siswa dalam satu kelas per mata pelajaran.
> - CRUD Nilai (Angka, Predikat, Deskripsi)
> - Mendukung konfigurasi format nilai per Unit (Angka, Huruf, atau Deskripsi)
> - Auto-Init Transkrip: Jika siswa belum memiliki record nilai, sistem otomatis membuatkan record kosong saat data dipanggil.

> LOGIC NOTE:
>
> - Auto-Generation: logic `detail()` akan otomatis membuat record `transkrip_nilai` dan `transkrip_nilai_detail` jika belum ada. Proses ini menggunakan instansiasi langsung ke `TranskripNilaiController` (Dependency Injection Manual).
> - Penilaian Dinamis: Kolom nilai yang muncul bergantung pada konfigurasi `PenilaianMataPelajaranKurikulum` (misal: Tugas, UTS, UAS). Input nilai yang tidak sesuai dengan struktur kurikulum akan dihapus otomatis.

## 1. List Maata Pelajaran Kelas

- **URL**: `GET /admin/pendidikan/akademik/nilai/mata-pelajaran`
- **Auth**: Session
- **Controller**: `NilaiMataPelajaranController@index`

> Description: Menampilkan daftar mata pelajaran yang diajarkan di kelas. Jika user adalah Guru, hanya menampilkan mata pelajaran yang diajarnya. Jika Admin/Staff, menampilkan semua mata pelajaran di unit tersebut.

### Parameters

| Param                   | Tipe   | Wajib | Keterangan                         |
| :---------------------- | :----- | :---- | :--------------------------------- |
| `draw`                  | Int    | Tidak | Param Datatables                   |
| `search[value]`         | String | Tidak | Keyword pencarian global           |
| `page`                  | Int    | Tidak | Filter status hapus                |
| `filter_mata_pelajaran` | UUID   | Tidak | Filter spesifik per mata pelajaran |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "nama_mata_pelajaran": "Matematika",
      "nama_pengajar": "Budi Santoso",
      "kode_kelas": "X-IPA-1",
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Form Input Nilai

- **URL**: `GET /admin/pendidikan/akademik/nilai/mata-pelajaran/detail`
- **Auth**: Session
- **Controller**: `NilaiMataPelajaranController@detail`

> Description: Menampilkan form input nilai untunk seluruh siswa dalam satu kelas pada mata pelajaran tertentu.
> Flow:
>
> 1. Cek struktur penilaian dari kurikulum (UTS/UAS/Tugas)
> 2. Ambil daftar siswa di kelas.
> 3. Loop siswa: cek apakah sudah punya transkrip?
>    - jika belum: panggil `TranskripNilaiController::tambah_transkrip_ke_peserta` untuk generate record kosong.
>    - jika sudah: ambil nilai yang tersimpan.
> 4. Return view tabel input nilai.

### Parameters

| Param                     | Tipe | Wajib | Keterangan                        |
| :------------------------ | :--- | :---- | :-------------------------------- |
| `id_kelas_mata_pelajaran` | UUID | Ya    | ID Mapel Kelas yang akan dinilai. |

---

## 3. Store Nilai

- **URL**: `POST /admin/pendidikan/akademik/nilai/mata-pelajaran/store`
- **Auth**: Session
- **Controller**: `NilaiMataPelajaranController@store`

> Description: Menyimpan nilai siswa secara massal atau parsial. Mendukung update nilai angka, nilai rapot, dan nilai huruf.

### Parameters

| Param                     | Tipe        | Wajib | Keterangan                            |
| :------------------------ | :---------- | :---- | :------------------------------------ |
| `id_kelas_mata_pelajaran` | UUID        | Ya    | ID Mata pelajaran kelas               |
| `data`                    | JSON String | Ya    | Array object nilai yang akan disimpan |

### Format JSON `data`:

```json
[
  {
    "id_transkrip_nilai_detail": "uuid...",
    "nilai": 85, // Nilai Asli
    "nilai_report": 90, // Nilai Rapor
    "nilai_huruf": "A" // Predikat
  }
]
```

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data transkrip nilai berhasil disimpan"
}
```

---

## 4. Riwayat Nilai

- **URL**: `GET /admin/pendidikan/akademik/nilai/mata-pelajaran/riwayat_nilai`
- **Auth**: Session
- **Controller**: `NilaiMataPelajaranController@riwayat_nilai`

> Description: Menampilkan riwayat nilai lengkap seorang siswa untuk mata pelajaran tertentu. Digunakan untuk melihat detail komponen nilai (Tugas 1, Tugas 2, UTS, UAS) yang membentuk nilai akhir.

### Parameters

| Param              | Tipe | Wajib | Keterangan                     |
| :----------------- | :--- | :---- | :----------------------------- |
| `id_peserta_didik` | UUID | Ya    | ID Siswa                       |
| `jenis_unit`       | Int  | Ya    | `1 = Sekolah`, `2 = Pesantren` |

---

## 5. Get List Mata Pelajaran Kurikulum

- **URL**: `GET /admin/pendidikan/akademik/nilai/mata-pelajaran/get-mata-pelajaran-by-kurikulum`
- **Auth**: Session
- **Controller**: `NilaiMataPelajaranController@get_mata_pelajaran_by_kurikulum`

> Description: Mengambil daftar mata pelajaran yang tersedia di kurikulum aktif tahun ajaran ini. Digunakan untuk filter dropdown di halaman index.
> Akses guru: Jika user adalah guru, hanya mengembalikan mata pelajaran yang diajar oleh guru tersebut.

### Parameters

Tidak ada parameter wajib. Filter otomatis berdasarkan Session User (Guru/Admin) dan Tahun Ajaran Aktif.

---
