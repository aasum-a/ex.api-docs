# Laporan

Modul ini menyediakan output data rekapitulasi untuk keperluan audit dan monitoring harian.

> **⚠️ PERFORMANCE NOTICE**
> Endpoint laporan melakukan pemrosesan data (agregasi) di level aplikasi (PHP), bukan database.
>
> - Hindari menarik data dengan rentang filter yang terlalu luas (misal: Semua Unit) pada jam sibuk.
> - Fitur Export PDF dapat menyebabkan _timeout_ jika jumlah data melebihi 100 baris. Gunakan Excel untuk data besar.

## Get Laporan Harian

Menampilkan rekapitulasi pembayaran yang masuk pada tanggal tertentu, dikelompokkan per Unit dan Jenis Tagihan.

- **URL**: `GET /admin/laporan/laporan-harian/get`
- **Auth**: Session Cookie

### Query Parameters

| Param               | Tipe  | Wajib | Keterangan                               |
| :------------------ | :---- | :---- | :--------------------------------------- |
| `tanggal`           | Date  | Ya    | Format: `Y-m-d`. Tanggal transaksi.      |
| `unit[]`            | Array | Ya    | Array UUID Unit yang ingin ditampilkan.  |
| `jenis_pendaftaran` | Int   | Ya    | `1` (Semua), `2` (Pindahan), `3` (Baru). |

### Response

Response ini didesain khusus untuk merender tabel bertingkat (Nested Table) di Frontend.

```json
{
  "success": true,
  "data": {
    "grand_total": {
      "total_pendaftar": 150,
      "total_semua": 500000000
    },
    "laporan_summary": [
      {
        "nama_unit": "SMA Unggulan",
        "program_data": [
          {
            "nama_program": "IPA",
            "tingkat_data": [
              {
                "tingkat": 10,
                "jumlah_pendaftar": 50,
                "total_tingkat_program": 1000000
              }
            ]
          }
        ]
      }
    ]
  }
}
```

---

## Get & Export Laporan Penerima Beasiswa

- **URL**:
  - Get: `/admin/laporan/laporan-harian/get`
  - Export: `/admin/laporan/laporan-penerima-beasiswa/export`
- **Method**: `GET`
- **Auth**: Session Cookie

### Query Parameters

| Param    | Tipe   | Wajib         | Keterangan                              |
| :------- | :----- | :------------ | :-------------------------------------- |
| `bulan`  | Int    | Ya            | Angka bulan (1-12).                     |
| `unit[]` | Array  | Ya            | Array UUID Unit yang ingin ditampilkan. |
| `format` | String | _Export Only_ | `pdf` atau `xls`.                       |
