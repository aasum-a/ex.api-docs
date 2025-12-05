# Transaksi

Modul ini digunakan oleh Staff Keuangan/Admin untuk memantau data pembayaran yang masuk dari seluruh channel (Transfer, Cash, Payment Gateway).

> **⚠️ ACCESS CONTROL WARNING**
> Endpoint `detail-bayar` saat ini tidak memvalidasi otorisasi Unit.
> Admin dari Unit A berpotensi dapat melihat detail pembayaran siswa Unit B jika mengetahui `id_pembayaran`-nya.

## Get Pembayaran

Menampilkan tabel riwayat pembayaran dengan filter yang sangat spesifik (DataTables).

- **URL**: `GET /admin/keuangan/transaksi/pembayaran`
- **Auth**: Session Cookie

### Query Parameters

| Param                       | Tipe   | Keterangan                                     |
| :-------------------------- | :----- | :--------------------------------------------- |
| `search[value]`             | String | Keyword pencarian global.                      |
| `filter_unit`               | UUID   | Filter Unit Sekolah.                           |
| `filter_program_jurusan`    | UUID   | Filter Jurusan.                                |
| `filter_gelombang`          | UUID   | Filter Gelombang Pendaftaran.                  |
| `filter_status_calon_siswa` | Int    | Filter status siswa (Diterima, Cadangan, dll). |
| `filter_unit_ps`            | UUID   | Filter Unit (Pilihan Kedua/Pindah Jenjang).    |

> **Session Logic:**
> Data otomatis difilter berdasarkan **Tahun Ajaran** yang sedang aktif di session admin (`_get_tahun_ajaran_session()`).

---

## Get Detail Pembayaran

Mengambil rincian item apa saja yang dibayar dalam satu Transaksi ID, termasuk informasi potongan beasiswa yang diterapkan.

- **URL**: `GET /admin/keuangan/transaksi/pembayaran/detail-bayar`
- **Auth**: Session Cookie

### Query Parameters

| Param           | Tipe | Wajib  | Keterangan               |
| :-------------- | :--- | :----- | :----------------------- |
| `id_pembayaran` | UUID | **Ya** | ID Transaksi Pembayaran. |

### Response Structure (Complex Nested)

Response berisi object `pembayaran` (Header) dan array `detail` (Items).

```json
{
  "status": "success",
  "code": 200,
  "data": {
    "pembayaran": {
      "id_pembayaran": "uuid...",
      "total_bayar": 5000000,
      "tgl_bayar": "2025-12-05",
      "status_pembayaran": "LUNAS" // (Helper: _status_pembayaran)
    },
    "detail": [
      {
        "nama_tagihan": "Uang Gedung",
        "nominal_tagihan": 10000000,
        "nominal_beasiswa": 5000000,
        "nominal_bayar": 5000000,
        "rencana_beasiswa": [
          {
            "nama_beasiswa": "Beasiswa Prestasi",
            "nominal": 5000000
          }
        ]
      }
    ]
  }
}
```
