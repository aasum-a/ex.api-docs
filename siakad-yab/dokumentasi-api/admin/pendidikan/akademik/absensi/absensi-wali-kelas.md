# Modul Absensi Wali Kelas

Modul ini menangani absensi harian siswa yang dikelola oleh Wali Kelas. Data ini biasanya menjadi acuan utama kehadiran siswa (sakit/izin/alpha) pada hari tersebut.

> ⚠️ SECURITY & QUALITY NOTE
>
> 1. **N+1 Query Issue**: `create` memuat data absensi per siswa menggunakan query di dalam loop. perlu optimasi eager loading.
> 2. **Tight Coupling**: Controller ini memiliki ketergantungan langsung (instansiasi manual) terhadap `DaftarKelasPesertaDidikController`.
> 3. **Data Integrity**: Validasi penghapusan data (`_checkIsUsed`) dinonaktifkan (return false), berisiko menghapus data yang memiliki relasi penting.

## 1. List Riwayat Absensi Wali Kelas

- **URL**: `GET /admin/pendidikan/akademik/absensi/wali-kelas`
- **Auth**: Session
- **Controller**: `AbsensiWaliKelasController@index`

> Description: Menampilkan daftar riwayat absensi harian kelas.
>
> 1. Jika parameter `id_kelas_peserta_didik` kosong, sistem akan menampilkan view pemilihan kelas (untuk admin/staff) atau me-redirect ke kelas ampuannya (untuk wali kelas).
> 2. Jika parameter ada, sistem menampilkan tabel riwayat absensi (JSON/Datatables).

### Parameters

| Param                    | Tipe | Wajib | Keterangan                   |
| :----------------------- | :--- | :---- | :--------------------------- |
| `id_kelas_peserta_didik` | UUID | Tidak | Filter berdasarkan ID Kelas. |
| `filter_tanggal`         | Date | Tidak | Filter tanggal spesifik      |
| `draw`                   | Int  | Tidak | Param Datatables             |

## 2. Form Absensi Harian (Create)

- **URL**: GET /admin/pendidikan/akademik/absensi/wali-kelas/create
- **Auth**: Session
- **Controller**: `AbsensiWaliKelasController@create`

> Description: Menyiapkan form absensi harian. mengambil daftar siswa dalam kelas wali kelas yang login, beserta status kehadiran mereka jika data tanggal tersebut sudah ada.

### Parameters

| Param                    | Tipe | Wajib | Keterangan                 |
| :----------------------- | :--- | :---- | :------------------------- |
| `id_kelas_peserta_didik` | UUID | Ya    | ID Kelas yang akan diabsen |
| `id_absen_wali_kelas`    | UUID | Tidak | Isi jika mode Edit         |

### Response Example

```json
{
  "id_kelas_peserta_didik": "uuid...",
  "tanggal": "2024-12-24",
  "row_absen_peserta": [
    {
      "id_peserta_didik": "uuid...",
      "nama": "Andy",
      "hadir": 1, // 1:Hadir, 2:Izin, 3:Sakit, 4:Alpha
      "keterangan": "-"
    }
  ]
}
```
