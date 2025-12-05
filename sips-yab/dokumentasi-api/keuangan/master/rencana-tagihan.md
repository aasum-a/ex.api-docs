# Rencana Tagihan

Modul ini adalah _**Core**_ dari sistem keuangan. Menentukan berapa nominal yang harus dibayar oleh siswa berdasarkan:

1.  **Unit**
2.  **Jurusan**
3.  **Tahun Ajaran**
4.  **Jenis Tagihan**
5.  **Tingkat**

> **⚠️ DEPENDENCY WARNING**
>
> 1.  **Session State:** Endpoint ini mewajibkan user memiliki Session Tahun Ajaran aktif (`_get_tahun_ajaran_session()`).
> 2.  **Distributed Transaction:** Simpan/Hapus memicu request ke SIAKAD.
> 3.  **Hardcoded Logic:** Pembuatan data baru memaksa `frekuensi_tagihan = 3` (Perlu konfirmasi ke Developer maksud angka ini).

## Get

Mengambil daftar tarif tagihan sesuai Tahun Ajaran aktif di Session.

- **URL**: `GET /admin/keuangan/master/rencana-tagihan`
- **Auth**: Session Cookie

### Query Parameters

| Param                    | Tipe | Keterangan                                                           |
| :----------------------- | :--- | :------------------------------------------------------------------- |
| `filter_unit`            | UUID | Filter Unit.                                                         |
| `filter_program_jurusan` | UUID | Filter Jurusan.                                                      |
| `filter_tagihan`         | UUID | Filter Nama Tagihan.                                                 |
| `filter_aktif`           | Int  | `1` (Aktif), `2` (Nonaktif).                                         |
| `page`                   | Int  | **Warning:** Digunakan untuk filter `sts_hapus` (1=Aktif, 2=Sampah). |

---

## Store

Membuat atau Update nominal tagihan.

- **URL**: `POST /admin/keuangan/master/rencana-tagihan/store`
- **Logic Unik**: Sistem akan menolak jika kombinasi `Tahun Ajaran + Unit + Jurusan + Tagihan` sudah ada (Mencegah duplikasi tarif).

### Body Parameters

| Param                   | Tipe   | Wajib    | Keterangan                                    |
| :---------------------- | :----- | :------- | :-------------------------------------------- |
| `id_rencana_tagihan`    | UUID   | Opsional | Isi untuk **Edit**. Kosong untuk **Create**.  |
| `id_tagihan_fr`         | UUID   | Ya       | ID Master Tagihan.                            |
| `id_unit_fr`            | UUID   | Ya       | ID Unit.                                      |
| `id_program_jurusan_fr` | UUID   | Ya       | ID Jurusan.                                   |
| `tingkat`               | Int    | Ya       | Semester/Kelas berapa tagihan ini berlaku.    |
| `nominal`               | Double | Ya       | Jumlah Rupiah. Wajib > 0.                     |
| `tagihan_registrasi`    | Int    | Ya       | `1` (Ditagih saat daftar ulang), `2` (Tidak). |
| `sts_aktif`             | Int    | Ya       | `1` (Aktif), `2` (Nonaktif).                  |

> **Defaults (Create Mode):**
> Jika membuat data baru, sistem otomatis mengeset:
>
> - `cicilan` = 1
> - `frekuensi_tagihan` = 3 (Magic Number, arti tidak jelas)
> - `sts_hapus` = 1

---

## Delete

Menghapus rencana tagihan.

- **URL**: `POST /admin/keuangan/master/rencana-tagihan/destroy`
- **Behavior**: Melakukan `delete()` lokal (Cek model apakah Soft/Hard delete) dan mengirim sinyal `hard-delete` ke SIAKAD.
- **Validasi**: Tidak bisa dihapus jika `id_rencana_tagihan` sudah terikat di tabel transaksi (Relasi dicek via `getHasManyRelations`).

---

## Sync Functionality

Endpoint khusus sinkronisasi data dengan SIAKAD.

### A. Pull from SIAKAD (`get_sync`)

Menarik data Rencana Tagihan dari SIAKAD ke SIPS.

- **URL**: `GET /admin/keuangan/master/rencana-tagihan/get-sync`
- **Side Effect**: Jika Master Data (Unit/Tagihan/Jurusan) dari SIAKAD belum ada di SIPS, fungsi ini akan **membuatnya secara otomatis** (Auto-Create Dependencies).

### B. Push Receiver (`update_from_sync`)

Menerima webhook update dari SIAKAD.

- **URL**: `POST /admin/keuangan/master/rencana-tagihan/update-from-sync`
- **Validasi**: Memastikan semua Foreign Key (`id_tahun_ajaran`, `id_unit`, dll) valid sebelum menyimpan.
