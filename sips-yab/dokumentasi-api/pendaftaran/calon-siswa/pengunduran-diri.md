# Pengunduran Diri

Modul ini menangani proses siswa yang membatalkan pendaftaran atau mengundurkan diri (Resign). Proses ini melibatkan perubahan status siswa dan **Pengembalian Dana (Retur/Refund)** jika siswa sudah melakukan pembayaran.

> **⚠️ FINANCIAL SECURITY RISK**
> Endpoint `store` saat ini mempercayai input `total_retur` dari Frontend (Client-side) tanpa menghitung ulang di Backend.
> **Risiko:** Penyerang dapat memanipulasi request untuk mencatat nilai refund yang lebih besar dari yang seharusnya di database.
> **Rekomendasi:** Backend wajib menjumlahkan ulang `nominal_retur` dari setiap item di `data_pembayaran` untuk memastikan integritas data.

---

## Get List Pengunduran Diri

Menampilkan daftar siswa yang sudah resmi mengundurkan diri.

- **URL**: `GET /admin/pendaftaran/calon-siswa/pengunduran-diri`
- **Auth**: Session Cookie

### Query Parameters

| Param                    | Tipe   | Keterangan                                              |
| :----------------------- | :----- | :------------------------------------------------------ |
| `search[value]`          | String | Pencarian global (Nama, No Daftar).                     |
| `filter_unit`            | UUID   | Filter Unit Sekolah.                                    |
| `filter_program_jurusan` | UUID   | Filter Jurusan.                                         |
| `filter_refund`          | Int    | Status Refund: `1` (Belum/Tidak Ada), `3` (Ada Refund). |
| `page`                   | Int    | Filter Hapus (Standard SIPS logic).                     |

---

## Lookup Calon Siswa

Digunakan saat Admin memilih siswa yang **akan** diproses mundur. Endpoint ini mengembalikan data diri siswa beserta **Riwayat Pembayaran** yang bisa di-refund.

- **URL**: `GET /admin/pendaftaran/calon-siswa/pengunduran-diri/get`
- **Auth**: Session Cookie

### Query Parameters

| Param                    | Tipe | Wajib | Keterangan                                                                                                             |
| :----------------------- | :--- | :---- | :--------------------------------------------------------------------------------------------------------------------- |
| `id_calon_siswa`         | UUID | Ya    | ID Siswa yang akan diproses.                                                                                           |
| `status_calon_siswa`     | Int  | Tidak | Filter status saat ini.                                                                                                |
| `jenis_pengunduran_diri` | Int  | Tidak | `1` = Boarding (Asrama), `2` = Sekolah (Full). Filter ini menentukan item tagihan mana yang muncul di list pembayaran. |

### Response Data

Mengembalikan object `CalonSiswa` dengan property tambahan `pembayaran` (array).

```json
{
  "status": "success",
  "data": {
    "id_calon_siswa": "uuid...",
    "nama_siswa": "Andy",
    "pembayaran": [
      {
        "id_pembayaran": "uuid...",
        "total_bayar": 5000000,
        "returList": [
          {
            "tagihan": "Uang Gedung",
            "nominal_bayar": 5000000
          }
        ]
      }
    ]
  }
}
```

## Store Pengunduran Diri

Menyimpan data pengunduran diri dan membuat transaksi retur keuangan secara otomatis.

- **URL**: `/admin/pendaftaran/calon-siswa/pengunduran-diri/store`
- **Method**: `POST`

### Body Parameters

| Parameters              | Tipe   | Wajib | Keterangan                                               |
| :---------------------- | :----- | :---- | :------------------------------------------------------- |
| id_calon_siswa          | UUID   | Ya    | Target siswa.                                            |
| jenis_pengunduran_diri  | Int    | Ya    | `1` (Boarding Only), `2` (Sekolah/Full)                  |
| tgl_pengunduran_diri    | Date   | Ya    | Format `Y-m-d`.                                          |
| alasan_pengunduran_diri | String | Tidak | Catatan alasan.                                          |
| total_retur             | Double | Ya    | `⚠️ RAWAN MANIPULASI`: Total nominal yang akan direfund. |
| data_pembayaran         | Array  | Ya    | List item yang diretur (Lihat struktur di bawah)         |

### Struktur `data_pembayaran`

```json
[
  {
    "id_pembayaran": "uuid-transaksi-pembayaran",
    "pembayaran_detail": [
      {
        "id_pembayaran_detail": "uuid-item-detail",
        "id_rencana_tagihan": "uuid-rencana",
        "periode": 1,
        "frekuensi": 1,
        "persen": 100,
        "nominal_retur": 2500000
      }
    ]
  }
]
```

### _Business Logic_

- Create Retur: Membuat data di tabel Retur dan ReturDetail.
- Update Siswa:
  - Jika Jenis 2 (Sekolah), status_calon_siswa diubah jadi 5 (Mundur).
  - Menghapus relasi unit/jurusan pilihan kedua (unit_ps, program_jurusan_ps).
- Update Pembayaran: Jika jumlah retur == jumlah bayar awal, status pembayaran diubah jadi 5 (Retur Penuh).
- Update Orang Tua: Mengupdate data anak di tabel OrangTua (JSON Column) untuk menyamakan status.

## Detail & Cetak Bukti

Melihat rincian pengunduran diri yang sudah terjadi dan mencetak bukti.

### Get Detail

- **URL**: `/admin/pendaftaran/calon-siswa/pengunduran-diri/detail`
- **Method**: `GET`
- **Param**: `id_pengunduran_diri` (UUID).
- **Response**: Mengembalikan data pengunduran diri dan data pembayaran yang diretur, serta data retur berupa PDF yang dapat dicetak.

### Cetak PDF

- **URL**: `/admin/pendaftaran/calon-siswa/pengunduran-diri/cetak`
- **Method**: `GET`
- **Param**: `id_pengunduran_diri` (UUID).
- **Output**: File PDF
