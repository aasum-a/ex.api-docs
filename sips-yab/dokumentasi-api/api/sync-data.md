# Beasiswa

Modul ini adalah _receiver_ (penerima) data dari sistem eksternal (SIAKAD). Bertugas untuk melakukan sinkronisasi satu arah (SIAKAD -> SIPS) untuk data Master Beasiswa.

> **⚠️ SECURITY & LOGIC WARNING**
>
> 1.  **No Explicit Auth Check:**
>     Di dalam function controller `update_from_sync` dan `soft_delete_from_sync` **TIDAK ADA** validasi token atau pengecekan user/API Key. Keamanan sepenuhnya bergantung pada Middleware di `routes/api.php`. Jika middleware jebol, endpoint ini bisa diakses publik dan data bisa diacak-acak.
>
> 2.  **Complex Validation (JSON String):**
>     Parameter `id_tagihan_fr` wajib dikirim dalam format **JSON String Array** (contoh: `["uuid-1", "uuid-2"]`), bukan array native. Sistem akan me-reject jika format JSON invalid atau ID Tagihan tidak ada di database.
>
> 3.  **Dependency Check:**
>     Endpoint Soft Delete melakukan pengecekan relasi (`getHasManyRelations`). Data tidak akan bisa dihapus jika sudah digunakan di tabel transaksi lain (referential integrity check manual).

## Update / Upsert From Sync

- **URL**: `POST /api/sync-data/beasiswa/update`
- **Auth**: **CRITICAL CHECK** (Pastikan dilindungi Middleware API Key/IP Whitelist).
- **Deskripsi**: Menerima payload data beasiswa. Jika `id_beasiswa` sudah ada, data akan di-update. Jika belum ada, data baru akan dibuat (Upsert Logic).

### Parameters (Body / Form-Data)

| Param                     | Tipe     | Wajib       | Keterangan                                               |
| :------------------------ | :------- | :---------- | :------------------------------------------------------- |
| `id_beasiswa`             | UUID     | Ya          | ID Primary Key dari SIAKAD.                              |
| `id_unit_fr`              | UUID     | Ya          | ID Unit (Wajib exist di tabel unit).                     |
| `id_kategori_beasiswa_fr` | UUID     | Tidak       | ID Kategori (Wajib exist jika diisi).                    |
| `id_tagihan_fr`           | JSON Str | Ya          | List ID Tagihan yang tercover. Format: `["id1", "id2"]`. |
| `nama_beasiswa`           | String   | Ya          | Min 5, Max 100 chars.                                    |
| `nama_beasiswa_en`        | String   | Ya          | Min 5, Max 100 chars.                                    |
| `nominal`                 | Double   | Kondisional | Wajib isi jika `persen` kosong.                          |
| `persen`                  | Double   | Kondisional | Wajib isi jika `nominal` kosong.                         |
| `sts_aktif`               | Int      | Ya          | `1` (Aktif), `2` (Non-Aktif).                            |
| `sts_hapus`               | Int      | Ya          | `1` (Normal), `2` (Terhapus).                            |
| `updated_by`              | String   | Tidak       | Username pengubah data (untuk log).                      |

## Soft Delete From Sync

- **URL**: `POST /api/sync-data/beasiswa/soft-delete/{id_beasiswa}`
- **Auth**: **CRITICAL CHECK** (Pastikan dilindungi Middleware API Key/IP Whitelist).
- **Deskripsi**: Menandai data beasiswa sebagai terhapus (`sts_hapus = 2`) tanpa menghapus fisik row di database.

### Parameters (Path & Body)

| Param         | Lokasi | Tipe   | Wajib | Keterangan                                    |
| :------------ | :----- | :----- | :---- | :-------------------------------------------- |
| `id_beasiswa` | PATH   | UUID   | Ya    | ID data yang akan dihapus.                    |
| `sts_hapus`   | BODY   | Int    | Ya    | Kirim value `2` untuk soft delete.            |
| `updated_by`  | BODY   | String | Tidak | String penanda updater, misal: "SYNC SIAKAD". |

---

# Kategori Beasiswa

## Update / Upsert

- **URL**: `POST /api/sync-data/kategori-beasiswa/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Menerima data Master Kategori Beasiswa dari SIAKAD.
- **Logic Unik**: Jika `id_kategori_beasiswa` tidak ditemukan, sistem akan mencoba mencari berdasarkan `nama_kategori_beasiswa` sebelum membuat data baru.

### Parameters (Body)

| Param                       | Tipe   | Wajib | Keterangan                       |
| :-------------------------- | :----- | :---- | :------------------------------- |
| `id_kategori_beasiswa`      | UUID   | Ya    | Primary Key dari SIAKAD.         |
| `nama_kategori_beasiswa`    | String | Ya    | Min 2, Max 100 chars.            |
| `nama_kategori_beasiswa_en` | String | Ya    | Min 2, Max 100 chars.            |
| `sts_aktif`                 | Int    | Ya    | `1` (Aktif), `2` (Non-Aktif).    |
| `sts_hapus`                 | Int    | Tidak | Status soft delete.              |
| `created_by`                | String | Tidak | User pembuat (jika create baru). |
| `updated_by`                | String | Tidak | User pengupdate.                 |

## Soft Delete

- **URL**: `POST /api/sync-data/kategori-beasiswa/soft-delete/{id_kategori_beasiswa}`
- **Desc**: Menandai kategori sebagai terhapus.
- **Constraint**: Gagal jika kategori masih digunakan di tabel lain (`getHasManyRelations` check).

### Parameters

| Param                  | Lokasi | Tipe | Wajib | Keterangan      |
| :--------------------- | :----- | :--- | :---- | :-------------- |
| `id_kategori_beasiswa` | PATH   | UUID | Ya    | ID data target. |
| `sts_hapus`            | BODY   | Int  | Ya    | Kirim `2`.      |

---

# Kelompok Rencana Tagihan

## Update / Upsert

- **URL**: `POST /api/sync-data/kelompok-rencana-tagihan/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Receiver untuk update data Kelompok Rencana Tagihan dari SIAKAD.
- **Validasi**:
  - `nama_kelompok_rencana_tagihan` harus unik (kecuali record yang sedang diupdate).
  - Pengecekan unik mengabaikan data yang sudah dihapus (`sts_hapus != 2`).

### Parameters (Body)

| Param                              | Tipe   | Wajib | Keterangan                    |
| :--------------------------------- | :----- | :---- | :---------------------------- |
| `id_kelompok_rencana_tagihan`      | UUID   | Ya    | Primary Key.                  |
| `nama_kelompok_rencana_tagihan`    | String | Ya    | Min 5, Max 100. Unique.       |
| `nama_kelompok_rencana_tagihan_en` | String | Ya    | Min 5, Max 100. Unique.       |
| `sts_aktif`                        | Int    | Ya    | `1` (Aktif), `2` (Non-Aktif). |
| `sts_hapus`                        | Int    | Tidak | Status soft delete.           |
| `created_by`                       | String | Tidak | User pembuat.                 |
| `updated_by`                       | String | Tidak | User pengupdate.              |

## Soft Delete

- **URL**: `POST /api/sync-data/kelompok-rencana-tagihan/soft-delete/{id}`
- **Desc**: Menandai kelompok sebagai terhapus.
- **Constraint**: Gagal jika data masih digunakan di tabel lain (`getHasManyRelations`).

### Parameters

| Param                         | Lokasi | Tipe | Wajib | Keterangan      |
| :---------------------------- | :----- | :--- | :---- | :-------------- |
| `id_kelompok_rencana_tagihan` | PATH   | UUID | Ya    | ID data target. |
| `sts_hapus`                   | BODY   | Int  | Ya    | Kirim `2`.      |

## Manual Sync

- **URL**: `GET /api/sync-data/kelompok-rencana-tagihan/sync` _(Asumsi route exist berdasarkan pola)_
- **Desc**: Memaksa SIPS menarik data terbaru dari SIAKAD.
- **Logic Matching**:
  1. Cari by `id`.
  2. Jika ID null, cari by `nama` (Case Insensitive).
  3. Jika tidak ketemu, Create New.

---

# Pengajuan Beasiswa SIPS

## Sync Get

> **⚠️ BROKEN ROUTE WARNING**
> Endpoint ini terdaftar di `route:list` mengarah ke `BeasiswaCalonSiswaController@get`, namun method `get()` **TIDAK DITEMUKAN** di dalam controller tersebut. Kemungkinan _Dead Code_ atau salah referensi controller.

- **URL**: `GET /api/sync-data/pengajuan-beasiswa-sips/get`
- **Status**: ❌ 404/500 (Method Not Found)
- **Rekomendasi**: Hapus route ini dari `routes/api.php` jika tidak digunakan, atau buat function `get()` jika memang fitur ini diperlukan untuk sinkronisasi pengajuan beasiswa ke SIAKAD.

---

# Program Jurusan

## Update / Upsert

- **URL**: `POST /api/sync-data/program-jurusan/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Receiver untuk update data Program Jurusan dari SIAKAD.
- **Dependency**: Data **Unit (`id_unit_fr`)** wajib sudah ada di SIPS sebelum endpoint ini dipanggil. Jika tidak, proses akan gagal (422).

### Parameters (Body)

| Param                     | Tipe   | Wajib | Keterangan                     |
| :------------------------ | :----- | :---- | :----------------------------- |
| `id_program_jurusan`      | UUID   | Ya    | Primary Key.                   |
| `id_unit_fr`              | UUID   | Ya    | ID Unit Sekolah (Foreign Key). |
| `kode_program_jurusan`    | String | Ya    | Kode unik jurusan.             |
| `nama_program_jurusan`    | String | Ya    | Nama jurusan.                  |
| `nama_program_jurusan_en` | String | Ya    | Nama jurusan (Inggris).        |
| `sts_registrasi`          | Int    | Tidak | Default: 2.                    |
| `sts_aktif`               | Int    | Tidak | Default: 1.                    |
| `sts_hapus`               | Int    | Tidak | Default: 1.                    |
| `created_by`              | String | Ya    | User pembuat.                  |
| `updated_by`              | String | Ya    | User pengupdate.               |

## Soft Delete

- **URL**: `POST /api/sync-data/program-jurusan/soft-delete/{id}`
- **Desc**: Menandai jurusan sebagai terhapus.
- **Constraint**: Gagal jika jurusan masih digunakan di tabel lain.

### Parameters

| Param                | Lokasi | Tipe | Wajib | Keterangan      |
| :------------------- | :----- | :--- | :---- | :-------------- |
| `id_program_jurusan` | PATH   | UUID | Ya    | ID data target. |
| `sts_hapus`          | BODY   | Int  | Ya    | Kirim `2`.      |

## Manual Sync

- **URL**: `GET /api/sync-data/program-jurusan/sync` _(Asumsi route exist)_
- **Desc**: Memaksa SIPS menarik data terbaru dari SIAKAD.
- **Logic Matching**:
  1. Cari by `id`.
  2. Jika ID null, cari by `kode_program_jurusan`.
  3. Jika Unit Sekolah (`id_unit_fr`) tidak ditemukan, record tersebut akan di-skip.

---

# Rencana Tagihan

## Update / Upsert

- **URL**: `POST /api/sync-data/rencana-tagihan/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Receiver untuk update data Rencana Tagihan (Setting Nominal Biaya) dari SIAKAD.
- **Validasi Unik**: Sistem mengecek kombinasi unik (`tahun_ajaran`, `unit`, `jurusan`, `tagihan`, `tingkat`, dll) sebelum insert. Jika duplikat, return 422.

### Parameters (Body)

| Param                            | Tipe   | Wajib | Keterangan                     |
| :------------------------------- | :----- | :---- | :----------------------------- |
| `id_rencana_tagihan`             | UUID   | Ya    | Primary Key.                   |
| `id_tagihan_fr`                  | UUID   | Ya    | ID Master Tagihan.             |
| `id_unit_fr`                     | UUID   | Ya    | ID Unit Sekolah.               |
| `id_program_jurusan_fr`          | UUID   | Ya    | ID Jurusan.                    |
| `id_kelompok_rencana_tagihan_fr` | UUID   | Tidak | ID Kelompok (Opsional).        |
| `id_tahun_ajaran_fr`             | UUID   | Ya    | ID Tahun Ajaran.               |
| `nominal`                        | Double | Ya    | Nominal biaya.                 |
| `tingkat`                        | Int    | Ya    | Tingkat kelas (1, 2, 3, dst).  |
| `cicilan`                        | Int    | Ya    | Jumlah cicilan.                |
| `frekuensi_tagihan`              | Int    | Ya    | `1`, `2`, `3`.                 |
| `tagihan_registrasi`             | Int    | Ya    | `1` (Ditagihkan), `2` (Tidak). |
| `sts_aktif`                      | Int    | Ya    | `1`, `2`.                      |

## HARD DELETE

> **⚠️ DANGER ZONE**
> Endpoint ini melakukan **Penghapusan Fisik (Permanent Delete)** pada tabel `rencana_tagihan`. Data yang dihapus tidak dapat dikembalikan.

- **URL**: `POST /api/sync-data/rencana-tagihan/hard-delete`
- **Controller**: `RencanaTagihanController@destroy`
- **Desc**: Menghapus data rencana tagihan secara permanen.
- **Sync Behavior**: Jika request ini berasal dari user lokal (bukan dari sync SIAKAD), sistem akan mengirim trigger ke SIAKAD untuk menghapus data yang sama di sana (Two-way deletion).

### Parameters

| Param                | Lokasi | Tipe | Wajib | Keterangan                                                                         |
| :------------------- | :----- | :--- | :---- | :--------------------------------------------------------------------------------- |
| `id_rencana_tagihan` | BODY   | UUID | Ya    | ID data yang akan dimusnahkan.                                                     |
| `sync`               | BODY   | Bool | Tidak | Flag penanda. Jika kosong, trigger sync ke SIAKAD. Jika `TRUE`, hanya hapus lokal. |

## Manual Sync

- **URL**: `GET /api/sync-data/rencana-tagihan/sync` _(Asumsi route exist)_
- **Desc**: Memaksa SIPS menarik data terbaru dari SIAKAD.
- **Auto-Healing Logic**:
  Jika saat sync ditemukan bahwa data relasi (Master Tagihan, Unit, atau Jurusan) belum ada di SIPS, sistem akan **otomatis membuat (Create)** data master tersebut berdasarkan payload dari SIAKAD.

---

# Skema Potongan Pembayaran

## Update / Upsert

- **URL**: `POST /api/sync-data/skema-potongan-pembayaran/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Receiver untuk update Skema Potongan (Beasiswa/Potongan Khusus) dari SIAKAD.
- **Complex Data**: Parameter `id_rencana_tagihan_fr` dan `periode_potongan` wajib dikirim dalam format **JSON String**.

### Parameters (Body)

| Param                          | Tipe     | Wajib | Keterangan                                                                 |
| :----------------------------- | :------- | :---- | :------------------------------------------------------------------------- |
| `id_skema_potongan_pembayaran` | UUID     | Ya    | Primary Key.                                                               |
| `id_unit_fr`                   | UUID     | Ya    | ID Unit Sekolah.                                                           |
| `id_tahun_ajaran_fr`           | UUID     | Ya    | ID Tahun Ajaran.                                                           |
| `id_rencana_tagihan_fr`        | JSON Str | Ya    | Array ID Rencana Tagihan yang kena potongan. Contoh: `["uuid1", "uuid2"]`. |
| `nama_skema`                   | String   | Ya    | Nama Skema.                                                                |
| `jumlah_periode_pembayaran`    | Int      | Ya    | Total periode bayar.                                                       |
| `jumlah_periode_potongan`      | Int      | Ya    | Total periode yang dipotong.                                               |
| `frekuensi_potongan`           | Int      | Ya    | `1`, `2`, `3`.                                                             |
| `periode_potongan`             | JSON Str | Ya    | Array periode. Contoh: `["1", "2"]`.                                       |
| `tgl_batas_akhir_pembayaran`   | Date     | Ya    | Format: `YYYY-MM-DD`.                                                      |
| `sts_aktif`                    | Int      | Ya    | `1` (Aktif), `2` (Non-Aktif).                                              |
| `sts_hapus`                    | Int      | Ya    | `1` (Normal), `2` (Terhapus).                                              |

## Soft Delete

- **URL**: `POST /api/sync-data/skema-potongan-pembayaran/soft-delete/{id}`
- **Desc**: Menandai skema sebagai terhapus.
- **Risk**: Endpoint ini memiliki logic untuk **mengirim balik (callback)** request delete ke SIAKAD jika `APP_SIAKAD_SYNC=TRUE`. Berpotensi _Infinite Loop_ jika SIAKAD juga melakukan hal yang sama tanpa flag pencegah.

### Parameters

| Param                          | Lokasi | Tipe | Wajib | Keterangan      |
| :----------------------------- | :----- | :--- | :---- | :-------------- |
| `id_skema_potongan_pembayaran` | PATH   | UUID | Ya    | ID data target. |
| `sts_hapus`                    | BODY   | Int  | Ya    | Kirim `2`.      |

## Manual Sync

- **URL**: `GET /api/sync-data/skema-potongan-pembayaran/sync` _(Asumsi route exist)_
- **Desc**: Memaksa SIPS menarik data terbaru dari SIAKAD.
- **Logic Matching**: Cari by `id`. Jika tidak ketemu, create new.

---

# Tagihan

## Update / Upsert

- **URL**: `POST /api/sync-data/tagihan/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Receiver untuk update Master Tagihan dari SIAKAD.
- **Validasi**:
  - `id_tagihan` wajib unik. Jika ID sudah ada tapi record tidak ditemukan (misal anomali data), sistem akan melempar error 500 (`ID Tagihan already exists`).

### Parameters (Body)

| Param               | Tipe   | Wajib | Keterangan       |
| :------------------ | :----- | :---- | :--------------- |
| `id_tagihan`        | UUID   | Ya    | Primary Key.     |
| `nama_tagihan`      | String | Ya    | Min 2, Max 100.  |
| `nama_tagihan_en`   | String | Ya    | Min 2, Max 100.  |
| `frekuensi_tagihan` | Int    | Ya    | `1`, `2`, `3`.   |
| `jenis_tagihan`     | Int    | Ya    | `1`, `2`.        |
| `sts_aktif`         | Int    | Ya    | `1`, `2`.        |
| `sts_hapus`         | Int    | Ya    | `1`, `2`.        |
| `created_by`        | String | Tidak | User pembuat.    |
| `updated_by`        | String | Tidak | User pengupdate. |

## Soft Delete

- **URL**: `POST /api/sync-data/tagihan/soft-delete/{id}`
- **Desc**: Menandai tagihan sebagai terhapus.
- **Constraint**: Gagal jika tagihan masih digunakan di tabel lain.

### Parameters

| Param        | Lokasi | Tipe | Wajib | Keterangan      |
| :----------- | :----- | :--- | :---- | :-------------- |
| `id_tagihan` | PATH   | UUID | Ya    | ID data target. |
| `sts_hapus`  | BODY   | Int  | Ya    | Kirim `2`.      |

## Manual Sync

- **URL**: `GET /api/sync-data/tagihan/sync` _(Asumsi route exist)_
- **Desc**: Memaksa SIPS menarik data terbaru dari SIAKAD.
- **Logic Matching**: Cari by `id`. Jika tidak ketemu, create new.

---

# Tahun Ajaran

## Update / Upsert

- **URL**: `POST /api/sync-data/tahun-ajaran/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Receiver untuk update data Tahun Ajaran dari SIAKAD.
- **Validasi**: Tanggal `periode_akhir` harus setelah `periode_awal`.

### Parameters (Body)

| Param                       | Tipe   | Wajib | Keterangan                      |
| :-------------------------- | :----- | :---- | :------------------------------ |
| `id_tahun_ajaran`           | UUID   | Ya    | Primary Key.                    |
| `tahun_ajaran`              | String | Ya    | Contoh: "2024/2025".            |
| `periode_awal`              | Date   | Ya    | Tanggal mulai akademik.         |
| `periode_akhir`             | Date   | Ya    | Tanggal selesai akademik.       |
| `periode_awal_pendaftaran`  | Date   | Tidak | Tanggal buka pendaftaran SIPS.  |
| `periode_akhir_pendaftaran` | Date   | Tidak | Tanggal tutup pendaftaran SIPS. |
| `sts_aktif`                 | Int    | Ya    | `1` (Aktif), `2` (Non-Aktif).   |
| `sts_hapus`                 | Int    | Tidak | Status soft delete.             |
| `created_by`                | String | Ya    | User pembuat.                   |
| `updated_by`                | String | Ya    | User pengupdate.                |

## Soft Delete

- **URL**: `POST /api/sync-data/tahun-ajaran/soft-delete/{id}`
- **Desc**: Menandai tahun ajaran sebagai terhapus.
- **Constraint**: Gagal jika tahun ajaran masih digunakan di tabel lain (referential integrity).

### Parameters

| Param             | Lokasi | Tipe | Wajib | Keterangan      |
| :---------------- | :----- | :--- | :---- | :-------------- |
| `id_tahun_ajaran` | PATH   | UUID | Ya    | ID data target. |
| `sts_hapus`       | BODY   | Int  | Ya    | Kirim `2`.      |

## Manual Sync

- **URL**: `GET /api/sync-data/tahun-ajaran/sync`
- **Desc**: Memaksa SIPS menarik semua data Tahun Ajaran dari SIAKAD.
- **Logic Matching**:
  1. Cari by `id`.
  2. Jika tidak ketemu, create new.

---

# Unit

## Update / Upsert

- **URL**: `POST /api/sync-data/unit/update`
- **Auth**: **CRITICAL CHECK** (Middleware Protection).
- **Deskripsi**: Receiver untuk update data Master Unit Sekolah dari SIAKAD.
- **Catatan Penting**: Endpoint ini hanya mensinkronisasi data teks (Nama, Kode, Jenis). File fisik (PDF Tata Tertib, Gambar Logo) **TIDAK** ikut disinkronisasi melalui endpoint ini.

### Parameters (Body)

| Param          | Tipe   | Wajib | Keterangan                               |
| :------------- | :----- | :---- | :--------------------------------------- |
| `id_unit`      | UUID   | Ya    | Primary Key.                             |
| `kode_unit`    | String | Ya    | Kode Unit (misal: "TK-A").               |
| `nama_unit`    | String | Ya    | Nama Unit.                               |
| `nama_unit_en` | String | Ya    | Nama Unit (Inggris).                     |
| `jenis_unit`   | Int    | Tidak | `1` (Reguler), `2` (Pesantren/Boarding). |
| `pesantren`    | Int    | Tidak | `1` (Ya), `2` (Tidak).                   |
| `sts_aktif`    | Int    | Ya    | `1` (Aktif), `2` (Non-Aktif).            |
| `sts_hapus`    | Int    | Ya    | `1` (Normal), `2` (Terhapus).            |
| `created_by`   | String | Ya    | User pembuat.                            |
| `updated_by`   | String | Ya    | User pengupdate.                         |

## Soft Delete

- **URL**: `POST /api/sync-data/unit/soft-delete/{id}`
- **Desc**: Menandai unit sebagai terhapus.
- **Constraint**: Gagal jika unit masih digunakan di tabel lain.

### Parameters

| Param       | Lokasi | Tipe | Wajib | Keterangan      |
| :---------- | :----- | :--- | :---- | :-------------- |
| `id_unit`   | PATH   | UUID | Ya    | ID data target. |
| `sts_hapus` | BODY   | Int  | Ya    | Kirim `2`.      |

## Manual Sync

- **URL**: `GET /api/sync-data/unit/sync`
- **Desc**: Memaksa SIPS menarik data terbaru dari SIAKAD.
- **Logic Matching**: Cari by `id`. Jika tidak ketemu, create new.
- **Default Value**: Jika create new, `kode_daftar` akan di-generate otomatis random 5 karakter.
