# Monitoring Progres

Modul ini digunakan oleh Panitia PPDB untuk memantau sejauh mana kelengkapan data calon siswa. Sangat berguna untuk _follow-up_ siswa yang macet di tengah jalan (misal: sudah daftar akun tapi belum bayar).

> **⚠️ PERFORMANCE NOTICE**
> Endpoint ini memiliki masalah **N+1 Query** yang signifikan karena melakukan query ulang ke database untuk setiap baris data guna menghitung persentase.
> **Rekomendasi:** Gunakan pagination kecil (10-25 data) untuk menghindari _loading time_ yang lama.

## Get Progres List

Mengambil data progres pendaftaran siswa dengan detail persentase kelengkapan per tahap.

- **URL**: `GET /admin/pendaftaran/calon-siswa/progres`
- **Headers**: `X-Requested-With: XMLHttpRequest` (Wajib untuk dapat JSON)
- **Auth**: Session Cookie

### Query Parameters

| Param                    | Tipe   | Keterangan                |
| :----------------------- | :----- | :------------------------ |
| `filter_unit`            | UUID   | Filter Unit Sekolah.      |
| `filter_gelombang`       | UUID   | Filter Gelombang.         |
| `filter_program_jurusan` | UUID   | Filter Jurusan.           |
| `search[value]`          | String | Pencarian nama/no daftar. |

### Response Data Structure

Data yang dikembalikan adalah **Persentase Kelengkapan (0-100%)** per tahapan.

```json
{
  "draw": 1,
  "recordsTotal": 100,
  "data": [
    {
      "no_daftar": "REG-2024-001",
      "nama_siswa": "Budi Santoso",
      "formulir": 100, // Persentase kelengkapan form
      "dokumen": 50, // Persentase upload dokumen
      "biodata": 0, // Persentase biodata diri
      "pembayaran": 100, // Status bayar (biasanya 0 atau 100)
      "persentase_all": 75 // Rata-rata keseluruhan
    }
  ]
}
```
