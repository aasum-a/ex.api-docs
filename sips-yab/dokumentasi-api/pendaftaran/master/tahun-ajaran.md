# Master Data: Tahun Ajaran

Modul ini mengatur periode akademik dan periode pendaftaran. Data ini menjadi acuan utama (`session`) untuk semua transaksi di aplikasi.

## Get List Tahun Ajaran

Mengambil daftar tahun ajaran.

- **URL**: `GET /admin/pendaftaran/master/tahun-ajaran`
- **Auth**: Session Cookie

### Query Parameters

| Param          | Tipe | Keterangan                                                           |
| :------------- | :--- | :------------------------------------------------------------------- |
| `filter_aktif` | Int  | Filter status aktif (1=Ya, 2=Tidak).                                 |
| `page`         | Int  | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |

---

## Store Tahun Ajaran (Create / Update)

Menyimpan periode tahun ajaran baru.

- **URL**: `POST /admin/pendaftaran/master/tahun-ajaran/store`
- **Content-Type**: `application/json`

### Body Parameters

| Param                       | Tipe   | Wajib    | Keterangan                                   |
| :-------------------------- | :----- | :------- | :------------------------------------------- |
| `id_tahun_ajaran`           | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**. |
| `tahun_ajaran`              | String | Ya       | Nama Tahun (Contoh: "2024/2025").            |
| `periode_awal_pendaftaran`  | Date   | Ya       | Format: `Y-m-d`.                             |
| `periode_akhir_pendaftaran` | Date   | Ya       | Format: `Y-m-d`. Harus setelah tanggal awal. |
| `sts_aktif`                 | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                 |

### Logic Validasi Tanggal

Sistem melakukan validasi kustom untuk mencegah **Overlapping Periode**:

1.  Tanggal Akhir harus > Tanggal Awal.
2.  Periode baru tidak boleh bertabrakan dengan rentang periode tahun ajaran yang sudah ada di database (Logic validasi `first` & `last` record).

---

## Sync Data (Integrasi SIAKAD)

Sinkronisasi data tahun ajaran dengan server SIAKAD.

- **Pull (Get Sync)**: `GET /admin/pendaftaran/master/tahun-ajaran/get-sync`

  - Mengirim seluruh data tahun ajaran lokal ke SIAKAD.
  - Menerima balasan data terbaru dari SIAKAD dan melakukan update/insert di lokal.
  - **Konsep:** _Full Sync_ (data lokal diperbarui total mengikuti SIAKAD).

- **Push (Update From Sync)**: `POST /admin/pendaftaran/master/tahun-ajaran/update-from-sync`
  - Webhook untuk menerima perubahan spesifik satu data dari SIAKAD.
