# Skema Potongan Pembayaran

Modul ini menangani aturan diskon atau pemotongan biaya. Biasanya digunakan untuk kasus: "Bayar 1 Semester di muka, gratis 1 bulan SPP".

> **⚠️ DATA STRUCTURE WARNING**
> Modul ini menyimpan relasi ke `Rencana Tagihan` menggunakan kolom **JSON** (`id_rencana_tagihan_fr`), bukan tabel pivot (Many-to-Many).
>
> - **Input API:** Data array harus dikirim dalam bentuk **Stringified JSON**, bukan Array Object murni.
> - **Performance:** List data memicu N+1 Query yang berat. Gunakan pagination kecil (`length=10`) untuk menjaga performa.

## Get

Mengambil daftar skema potongan.

- **URL**: `GET /admin/keuangan/master/skema-potongan-pembayaran`
- **Auth**: Session Cookie

### Query Parameters

| Param                    | Tipe   | Keterangan                                                           |
| :----------------------- | :----- | :------------------------------------------------------------------- |
| `filter_unit`            | UUID   | Filter Unit.                                                         |
| `filter_rencana_tagihan` | String | Mencari ID Rencana Tagihan di dalam JSON kolom (Full Table Scan).    |
| `page`                   | Int    | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |

---

## Store

Menyimpan aturan skema baru.

- **URL**: `POST /admin/keuangan/master/skema-potongan-pembayaran/store`
- **Content-Type**: `multipart/form-data` (Recommended due to parsing logic)

### Body Parameters

| Param                          | Tipe        | Wajib    | Keterangan                                                                                  |
| :----------------------------- | :---------- | :------- | :------------------------------------------------------------------------------------------ |
| `id_skema_potongan_pembayaran` | UUID        | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.                                                |
| `id_unit_fr`                   | UUID        | Ya       | ID Unit.                                                                                    |
| `nama_skema`                   | String      | Ya       | Nama promo/potongan.                                                                        |
| `id_rencana_tagihan_fr`        | JSON String | Ya       | **PENTING:** Array ID Rencana Tagihan yang di-stringify. <br>Contoh: `["uuid-1", "uuid-2"]` |
| `jumlah_periode_pembayaran`    | Int         | Ya       | Syarat jumlah bayar (misal: Bayar **6** bulan).                                             |
| `jumlah_periode_potongan`      | Int         | Ya       | Reward potongan (misal: Gratis **1** bulan).                                                |
| `frekuensi_potongan`           | Int         | Ya       | `1`, `2`, atau `3` (Perlu konfirmasi arti kode ini).                                        |
| `periode_potongan`             | JSON String | Ya       | Array periode/bulan yang dipotong. <br>Contoh: `["Juli", "Agustus"]`                        |
| `tgl_batas_akhir_pembayaran`   | Date        | Ya       | Format: `YYYY-MM-DD`. Batas promo berlaku.                                                  |
| `sts_aktif`                    | Int         | Ya       | `1` (Aktif), `2` (Nonaktif).                                                                |

### Logic Validation Note

Controller melakukan `json_decode` manual terhadap `id_rencana_tagihan_fr` dan `periode_potongan`. Pastikan client mengirim data dalam format yang valid untuk di-decode (String JSON), bukan raw array.

---

## Delete & Restore

Standar Soft Delete & Restore dengan sinkronisasi ke SIAKAD.

- **Delete**: `POST /admin/keuangan/master/skema-potongan-pembayaran/destroy`
- **Restore**: `POST /admin/keuangan/master/skema-potongan-pembayaran/restore`

**Param:** `id_skema_potongan_pembayaran` (UUID).

---

## Sync Functionality

Mekanisme sinkronisasi dua arah dengan SIAKAD.

- **Pull (`get-sync`)**: Menarik data skema dari SIAKAD. Melakukan insert jika ID belum ada.
- **Push Receiver (`update-from-sync`)**: Menerima data dari SIAKAD.
  - **Alert:** Endpoint ini melakukan validasi `unique` check manual. Jika ID bentrok dengan data yang sudah ada tapi berbeda konten, sistem akan me-return Error 500 (Duplicate logic check).
