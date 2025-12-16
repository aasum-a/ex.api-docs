# Gelombang Pendaftaran

Modul ini mengatur periode penerimaan siswa baru. Terdiri dari data **Header** (Periode Tanggal) dan **Detail** (Biaya & Diskon yang berlaku).

---

## Gelombang

Mengatur rentang waktu pendaftaran.

### Get List

- **URL**: `GET /admin/pendaftaran/master/gelombang`
- **Auth**: Session Cookie

### Store (Create/Update)

- **URL**: `POST /admin/pendaftaran/master/gelombang/store`
- **Logic Validasi**:
  - Tanggal Akhir > Tanggal Awal.
  - Periode gelombang tidak boleh bertabrakan (overlapping) dengan gelombang lain dalam satu Unit.

**Body Parameters:**
| Param | Tipe | Wajib | Keterangan |
| :--- | :--- | :--- | :--- |
| `id_unit_fr` | UUID | Ya | Unit sekolah. |
| `gelombang` | String | Ya | Nama (Misal: "Gelombang Dini"). |
| `periode_awal` | Date | Ya | Tanggal buka. |
| `periode_akhir` | Date | Ya | Tanggal tutup. |
| `tanggal_tes_...`| Date | Tidak | Jadwal tes (CO, Psikotes, Interview). |

---

## Gelombang Detail (Biaya & Diskon)

Mengatur **Rincian Biaya** yang berlaku pada gelombang tersebut. Misal: Di Gelombang 1, biaya "Uang Gedung" dapat diskon 50%.

### Get List Detail

- **URL**: `GET /admin/pendaftaran/master/gelombang-detail`
- **Query Param**: `id_gelombang` (UUID, Wajib).

### Store Detail

- **URL**: `POST /admin/pendaftaran/master/gelombang-detail/store`

**Body Parameters:**
| Param | Tipe | Wajib | Keterangan |
| :--- | :--- | :--- | :--- |
| `id_gelombang_fr` | UUID | Ya | Parent ID. |
| `id_tagihan_fr` | UUID | Ya | Item tagihan (Misal: SPP, Gedung). |
| `id_program_jurusan_fr`| UUID | Ya | Jurusan spesifik. |
| `nominal` | Double | _ | Nilai rupiah yang harus dibayar. |
| `persen` | Int | _ | Diskon dalam persen (0-100). |

> **Business Logic:**
> User wajib mengisi minimal salah satu dari `nominal` ATAU `persen`. Jika keduanya 0 atau kosong, sistem akan menolak.

---

## Restore Data (Soft Delete Handling)

Mengembalikan data yang terhapus (Soft Delete), dengan validasi integritas data.

- **URL**: `POST /admin/pendaftaran/master/gelombang-detail/restore`
- **Logic**: Sebelum restore, sistem mengecek apakah ada data aktif lain yang memiliki kombinasi unik yang sama (`gelombang + tagihan + unit + jurusan`). Jika ada duplikat, restore ditolak.
