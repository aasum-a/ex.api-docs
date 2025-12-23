# Pengajuan Mutasi & Pindah Jenjang

Modul ini menangani proses siswa/calon siswa yang ingin berpindah Unit Sekolah (misal: SMP -> SMA), Jurusan (IPA -> IPS), atau Program (Reguler -> Pesantren) **setelah** melakukan pembayaran awal.

> **⚠️ LOGIC REQUIREMENT**
> User **WAJIB** sudah melunasi pembayaran pendaftaran (`status_pembayaran = 2`) di jenjang saat ini sebelum bisa mengajukan mutasi.

---

## 1. Halaman Pengajuan Mutasi

Menampilkan form pengajuan mutasi dengan data unit/jurusan tujuan yang tersedia.

- **URL**: `GET /dashboard/pengajuan-pindah-jenjang`
- **Auth**: Session Cookie
- **Controller**: `index`

### Logic Flow

1.  **Session Check:** Mengambil ID Calon Siswa dari sesi.
2.  **Pending Request Check:** Mengecek tabel `mutasi_program_calon_siswa`. Jika ada pengajuan berstatus `1` (Pending) yang belum ada `tgl_mutasi`-nya, user ditolak (Redirect).
3.  **Payment Check:** Memastikan user sudah bayar lunas (`status_pembayaran = 2`) di Tahun Ajaran aktif.
4.  **Data Loading:**
    - Mengambil daftar Unit & Jurusan yang `buka_pendaftaran = 1` dan **bukan** unit saat ini.
    - Mengambil data Unit Pesantren (Boarding).

---

## 2. Simulasi Tagihan Mutasi (Calculator)

Endpoint AJAX untuk menghitung selisih atau perbandingan biaya antara Unit Lama vs Unit Tujuan.

- **URL**: `GET /dashboard/pengajuan-pindah-jenjang/generate-pengajuan`
- **Auth**: Session Cookie
- **Controller**: `getPerbandinganTagihan`

### Parameters

| Param                          | Tipe   | Wajib  | Keterangan                                  |
| :----------------------------- | :----- | :----- | :------------------------------------------ |
| `id_unit_dituju`               | UUID   | Ya     | ID Unit Sekolah tujuan.                     |
| `id_program_jurusan_dituju`    | UUID   | Ya     | ID Jurusan tujuan.                          |
| `jenis_program_sekolah_dituju` | Int    | Ya     | `1`: Reguler, `2`: Boarding.                |
| `jenis_pendaftaran_dituju`     | Int    | Ya     | `1`: Baru, `2`: Pindahan.                   |
| `id_unit_ps_dituju`            | UUID   | _Cond_ | Wajib jika tujuan Boarding.                 |
| `id_program_jurusan_ps_dituju` | UUID   | _Cond_ | Wajib jika tujuan Boarding.                 |
| `payment_option`               | String | Ya     | `normal`, `sekaligus`, atau `cicil_custom`. |
| `periode`                      | Array  | _Cond_ | Array angka (1-12) jika `cicil_custom`.     |
| `tingkat_pendaftaran_dituju`   | Int    | Ya     | Tingkat kelas tujuan.                       |

### Response Example

```json
{
    "status": "success",
    "code": 200,
    "data": {
        "tagihan_lama": { ... }, // Detail tagihan unit sekarang
        "tagihan_baru": { ... }  // Estimasi tagihan unit tujuan
    }
}
```

---

## 3. Simpan Pengajuan Mutasi (Store)

Menyimpan data pengajuan mutasi ke _database_ untuk diproses oleh Admin

- **URL**: `POST /dashboard/pengajuan-pindah-jenjang/store`
- **Auth**: `Session Cookie`
- **Method**: `POST`

> ⚠️ **DATA INTEGRITY**: Endpoint ini juga mengupdate data rekening bank siswa (m_data_lengkap) sesuai inputan form mutasi.

### Request Body

| Param                          | Tipe   | Wajib  | Keterangan                                  |
| :----------------------------- | :----- | :----- | :------------------------------------------ |
| `id_unit_dituju`               | UUID   | Ya     | ID Unit Sekolah tujuan.                     |
| `id_program_jurusan_dituju`    | UUID   | Ya     | ID Jurusan tujuan.                          |
| `jenis_program_sekolah_dituju` | Int    | Ya     | `1`: Reguler, `2`: Boarding.                |
| `jenis_pendaftaran_dituju`     | Int    | Ya     | `1`: Baru, `2`: Pindahan.                   |
| `id_unit_ps_dituju`            | UUID   | _Cond_ | Wajib jika tujuan Boarding.                 |
| `id_program_jurusan_ps_dituju` | UUID   | _Cond_ | Wajib jika tujuan Boarding.                 |
| `payment_option`               | String | Ya     | `normal`, `sekaligus`, atau `cicil_custom`. |
| `periode`                      | Array  | _Cond_ | Array angka (1-12) jika `cicil_custom`.     |
| `tingkat_pendaftaran_dituju`   | Int    | Ya     | Tingkat kelas tujuan.                       |
| ajukan                         | Int    | Ya     | Harus bernilai 1                            |
| alasan_mutasi                  | String | Ya     | Alasan perpindahan                          |
| bank_dituju                    | String | Ya     | Nama Bank untuk refund/selisih              |
| no_rekening_bank_dituju        | String | Ya     | Nomor Rekening                              |
| rekening_an_bank_dituju        | String | Ya     | Atas nama pemilik rekening                  |

### Database Updates

1. **Insert** ke tabel `mutasi_program_calon_siswa` dengan `status_pengajuan = 1`
2. **Update** tabel `data_lengkap_calon_siswa`
