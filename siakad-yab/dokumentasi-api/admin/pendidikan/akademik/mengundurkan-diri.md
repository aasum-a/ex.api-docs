# Mengundurkan Diri

Modul ini menangani proses siswa yang keluar dari sekolah (pindah sekolah atau drop out) sebelum masa studi selesai. Modul ini juga otomatis menghitung Retur (Pengembalian Biaya) jika siswa sudah membayar lunas tapi keluar di tengah jalan.

> \*\*⚠️ CRITICAL FINANCE LOGIC
> Modul ini memiliki logika perhitungan keuangan yang di-_harcode_ untuk menentukan presentase pengembalian uang berdasarkan bulan keluar.
>
> - Perubahan kebijakan potongan biaya memerlukan perubahan kode.
>   proses penyimpanan data memicu kalkulasi ulang tabel tagihan secara sinkronus.

## 1. List Data Siswa Keluar

- **URL**: `GET /admin/pendidikan/akademik/mengundurkan-diri`
- **Auth**: Session
- **Controller**: `MengundurkanDiriController@index`

> Description: Menampilkan daftar siswa yang telah diproses keluar. Mendukung pencarian server-side.

### Parameters

| Param           | Tipe   | Wajib | Keterangan                                      |
| :-------------- | :----- | :---- | :---------------------------------------------- |
| `draw`          | Int    | Tidak | Param Datatables                                |
| `search[value]` | String | Tidak | Pencarian (Nama, NIS, Unit)                     |
| `page`          | Int    | Tidak | Filter Status Hapus (`1 = Aktif`, `2 = Sampah`) |

---

## 2. Store Pengunduran Diri

- **URL**: `POST /admin/pendidikan/akademik/mengundurkan-diri/store`
- **Auth**: Session
- **Controller**: `MengundurkanDiriController@store`

> Description: Memproses siswa keluar.

> 1. **Validasi:** Cek data input & file bukti.
> 2. **Cek Tagihan:** Memastikan siswa tidak memiliki tunggakan...
> 3. **Hitung Retur:**
>
> - Sistem otomatis mengecek pembayaran masa depan yang sudah dilunasi.
> - Menghitung persentase pengembalian (refund) berdasarkan aturan:
>   - Bulan 1-3: 75% kembali.
>   - Bulan 4-6: 50% kembali.
>   - dst...
> - Membuat `Retur` dan `DetailRetur` di modul Keuangan.
>
> 4. **Update Status Akademik:**
>
> - Mengeluarkan siswa dari Kelas Sekolah & Asrama.
> - Mengupdate status siswa di master `peserta_didik` menjadi "Mengundurkan Diri".
>
> 5. **Upload File**: Menyimpan scan surat permohonan pindah.
> 6. **Trigger**: Memanggil `DataPembayaranController` untuk recount tagihan.

### Parameters

| Param                    | Tipe   | Wajib    | Keterangan                                       |
| :----------------------- | :----- | :------- | ------------------------------------------------ |
| id_peserta_didik         | UUID   | Ya       | ID Siswa                                         |
| tgl_mengundurkan_diri    | Date   | Ya       | Tanggal efektif keluar                           |
| jenis_mengundurkan_diri  | Int    | Ya       | `1 = Hanya Pesantren`, `2 = Sekolah & Pesantren` |
| alasan_mengundurkan_diri | String | Ya       | Alasan kepindahan                                |
| file_bukti_sekolah       | File   | Optional | PDF/Img bukti surat (Max 2MB)                    |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data Berhasil Disimpan"
}
```

---

## Restore Siswa

- **URL**: `POST /admin/pendidikan/akademik/mengundurkan-diri/restore`
- **Auth**: Session
- **Controller**: `MengundurkanDiriController@restore`

> Description: Membatalkan status keluar siswa.
>
> 1. Mengembalikan status siswa menjadi "Aktif" di master data.
> 2. Memasukkan kembali siswa ke kelas terakhirnya (jika kelas masih ada).
> 3. Rollback Keuangan:
>
> - Menghapus data `Retur` yang terbuat otomatis.
> - Mengembalikan status pembayaran menjadi "Lunas" (Active).
> - Mengaktifkan kembali Beasiswa jika sebelumnya diputus.

### Parameters

| Param        | Tipe  | Wajib | Keterangan                 |
| :----------- | :---- | :---- | :------------------------- |
| data_restore | Array | Ya    | Array ID Mengundurkan Diri |

---

## 4. Get Detail & File

- **URL**: `GET .../detail-mengundurkan-diri` (Data) & `.../get-file` (File)
- **Auth**: Session

> Description:
>
> - Detail: Mengambil informasi detail pengunduran diri beserta rincian retur uang yang diterima siswa.
> - Get File: Mengunduh atau melihat file bukti surat pindah.

---

## 5. Cetak Surat

- **URL**: `GET /admin/pendidikan/akademik/mengundurkan-diri/cetak-surat`
- **Auth**: Session
- **Controller**: `MengundurkanDiriController@cetak_surat`

> Description: Men-generate PDF Surat Keterangan Pindah atau Surat Rincian Pengembalian Biaya (Retur).

### Parameters

| Param                  | Tipe | Wajib | Keterangan                                  |
| :--------------------- | :--- | :---- | :------------------------------------------ |
| `id_mengundurkan_diri` | UUID | Ya    | ID Data                                     |
| `jenis_surat`          | Int  | Ya    | `1 = Surat Keterangan`, `2 = Rincian Biaya` |
