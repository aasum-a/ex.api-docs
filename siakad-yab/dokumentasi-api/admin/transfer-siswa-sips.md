# Transfer Siswa SIPS (Migrasi PPDB)

Modul ini menangani proses pemindahan data Calon Siswa Baru (PPDB) dari sistem pendaftaran eksternal (SIPS) masuk ke dalam sistem akademik utama (SIAKAD). Fungsi Utama:

- Mengambil data calon siswa yang sudah lolos seleksi & bayar daftar ulang di SIPS.
- Mengkonversi data pembayaran formulir/daftar ulang di SIPS menjadi data Keuangan di SIAKAD.
- Membuat akun User Orang Tua secara otomatis.

> ⚠️ INTEGRATION NOTE Modul ini bekerja dengan Dua Koneksi Database (mysql untuk SIAKAD dan mysql_sips untuk SIPS). Pastikan konfigurasi database di .env sudah benar dan user database memiliki akses ke kedua schema jika berada di host yang sama.

## 1. List Calon Siswa (SIPS)

- **URL**: `GET /admin/pendidikan/akademik/peserta-didik/baru`
- **Auth**: Session
- **Controller**: `TransferPesertaDidikSIPSController@index_baru`

> Description: Menampilkan daftar calon siswa dari database SIPS yang statusnya sudah siap dimigrasi (lulus seleksi dan administrasi lengkap).

### Parameters

| Param               | Tipe | Wajib | Keterangan                         |
| :------------------ | :--- | :---- | :--------------------------------- |
| draw, start, length | Int  | Tidak | Param Datatables                   |
| filter_divisi       | Int  | Tidak | Filter Unit Sekolah                |
| filter_program      | Int  | Tidak | Filter Program (Reguler/Pesantren) |

---

## 2. Preview Tagihan SIPS

- **URL**: `GET /admin/pendidikan/akademik/peserta-didik/baru/get-tagihan-siakad`
- **Auth**: Session
- **Controller**: `TransferPesertaDidikSIPSController@getTagihanDistincByUnitForTransferSIPS`

> Description: Mengambil rincian tagihan yang harus dibayar siswa di unit tujuan (SIAKAD) untuk dibandingkan dengan pembayaran yang sudah dilakukan di SIPS. Digunakan untuk mapping konversi pembayaran.

### Parameters

| Param              | Tipe | Wajib | Keterangan                  |
| :----------------- | :--- | :---- | :-------------------------- |
| id_calon_siswa     | UUID | Ya    | ID Calon Siswa di SIPS      |
| id_program_jurusan | UUID | Ya    | ID Jurusan Tujuan di SIAKAD |

### Response Example

```json
{
  "status": "success",
  "data": {
    "id_kelas": [
      {
        "id_rencana_tagihan": "...",
        "nama_tagihan": "SPP Juli",
        "nominal": 500000
      }
    ],
    "id_kelas_ps": []
  }
}
```

---

## 3. Eksekusi Migrasi

- **URL**: `POST /admin/pendidikan/akademik/peserta-didik/baru/migrate-sips-to-siakad`
- **Auth**: Session
- **Controller**: `TransferPesertaDidikSIPSController@migrate_sips_to_siakad`

> Description: Proses ini pemindahan data. Flow Logic:
>
> 1. Copy Data Master: Salin data Peserta Didik, Orang Tua, dan Wilayah dari SIPS ke SIAKAD.
> 2. Generate Akun: Buat user login untuk Orang Tua (Ayah/Ibu/Wali).
> 3. Konversi Keuangan:
>    - Mapping pembayaran di SIPS ke pos tagihan di SIAKAD (berdasarkan input user).
>    - Buat transaksi pembayaran di SIAKAD senilai nominal yang sudah dibayar di SIPS.
>    - Jika ada selisih lebih, buat transaksi retur.
> 4. Update SIPS: Tandai data di SIPS dengan status 4 (Sudah Migrasi) agar tidak bisa ditarik ulang.

### Parameters

| Param                    | Tipe | Wajib    | Keterangan                        |
| :----------------------- | :--- | :------- | :-------------------------------- |
| id_calon_siswa           | UUID | Ya       | ID Siswa SIPS                     |
| id_program_jurusan       | UUID | Ya       | Jurusan Sekolah Tujuan            |
| id_program_jurusan_ps    | UUID | Optional | Jurusan Pesantren (jika boarding) |
| data_konversi_per_faktur | JSON | Ya       | Mapping pembayaran SIPS -> SIAKAD |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 4. Get Pembayaran SIPS

- **URL**: `GET /admin/pendidikan/akademik/peserta-didik/baru/baru-get-payment`
- **Auth**: Session
- **Controller**: `TransferPesertaDidikSIPSController@get_pembayaran_sips`

> Description: Menampilkan detail riwayat pembayaran siswa di SIPS untuk keperluan verifikasi sebelum migrasi.

### Parameters

| Param          | Tipe | Wajib | Keterangan     |
| :------------- | :--- | :---- | :------------- |
| id_calon_siswa | UUID | Ya    | ID Calon Siswa |

