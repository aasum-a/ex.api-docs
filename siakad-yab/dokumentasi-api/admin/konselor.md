# Konselor

Modul ini menangani pengelolaan data tenaga Konselor Sekolah (Guru BK).
Fitur:

- CRUD Data Biodata Konselor.
- **Auto-Generate User Account**: Saat data konselor dibuat, sistem otomatis membuat akun `User` dengan Role level 6.
- Integrasi data wilayah (Provinsi/Kabupaten/Kecamatan) dan Agama.

> ⚠️ LOGIC NOTE
>
> - **User Creation**: Password default untuk akun konselor baru adalah sama dengan generated username (kode_konselor).
> - **Soft Delete Sync**: Saat data konselor dihapus (soft delete), akun user terkait otomatis dinonaktifkan (`active = 2`). Saat di-restore, akun user diaktifkan kembali (`active = 1`).

## 1. List Data Konselor

- **URL**: `GET /admin/pendidikan/akademik/konselor`
- **Auth**: Session
- **Controller**: `KonselorController@index`

> Description: Menampilkan daftar konselor sekolah. Mendukung server-side searching dan filtering berdasarakan Unit Sekolah (jika user bukan Admin Unit).

### Parameters

| Param           | Tipe   | Wajib | Keterangan                                      |
| :-------------- | :----- | :---- | :---------------------------------------------- |
| `draw`          | Int    | Tidak | Param Datatables.                               |
| `search[value]` | String | Tidak | Keyword pencarian global.                       |
| `page`          | Int    | Tidak | Filter status hapus (`1 = aktif`, `2 = sampah`) |
| `filter_unit`   | Int    | Tidak | Filter berdasarkan ID Unit (untuk Super Admin)  |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_konselor": "K001",
      "nama_konselor": "Siti Aminah, S.Psi",
      "no_hp": "081234567890",
      "nama_unit": "SMA",
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Konselor

- **URL**: `POST /admin/pendidikan/akademik/konsoler/store`
- **Auth**: Session
- **Controller**: `KonselorController@store`

> Description: Menyimpan data baru atau memperbarui data konselor.
> Side Effect:
>
> - Create: Membuat record baru di tabel `konselor` dan tabel `user`
> - Update: Memperbarui data biodata dan sinkronisasi data akun (Email, Nama, Alamat, No HP) di tabel `user`.

### Parameters

| Param             | Tipe   | Wajib    | Keterangan                               |
| :---------------- | :----- | :------- | :--------------------------------------- |
| `id_konselor`     | UUID   | Optional | Isi jika dalam mode edit.                |
| `nama_konselor`   | String | Ya       | Nama lengkap tanpa gelar.                |
| `email`           | String | Ya       | Email unik (digunakan untuk login/user). |
| `no_hp`           | String | Ya       | Nomor handphone aktif.                   |
| `jenis_kelamin`   | Enum   | Ya       | `L` (laki-laki) atau `P` (Perempuan).    |
| `tempat_lahir`    | String | Ya       | Kota kelahiran.                          |
| `tanggal_lahir`   | Date   | Ya       | Format YYYY-MM-DD.                       |
| `alamat`          | String | Ya       | Alamat domisili.                         |
| `sts_aktif`       | Int    | Ya       | `1` = Aktif, `2` = Nonaktif.             |
| `nik`             | String | Tidak    | 16 digit angka (unik).                   |
| `sts_perkawinan`  | Int    | Ya       | `1 = Belum`, `2 = Menikah`, `3 = Cerai`. |
| `gelar_depan`     | String | Tidak    | Contoh: Dr., Drs.                        |
| `gelar_belakang`  | String | Tidak    | Contoh: S.Psi, M.Pd.                     |
| `id_agama_fr`     | UUID   | Tidak    | ID Refernsi Agama.                       |
| `id_provinsi_fr`  | UUID   | Tidak    | ID Wilayah Provinsi.                     |
| `id_kabupaten`    | UUID   | Tidak    | ID Wilayah Kabupaten.                    |
| `id_kecamatan_fr` | UUID   | Tidak    | ID Wilayah Kecamatan.                    |

> (Note: Parameter wilayah lain seperti rt, rw, dusun, kelurahan, pos, no_kk bersifat opsional)

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Get Detail Konselor

- **URL**: `GET /admin/pendidikan/akademik/konselor/get`
- **Auth**: Session
- **Controller**: `KonselorController`

> Description: Mengambil detail data satu konselor atau mencari list konselor untuk keperluan dropdown.

### Parameters

| Param       | Tipe   | Wajib    | Keterangan                                   |
| :---------- | :----- | :------- | :------------------------------------------- |
| id_konselor | UUID   | Optional | Ambil detail satu data spesifik.             |
| search      | String | Optional | Cari konselor berdasarkan Nama/Kode/NIK/ddl. |

### Response Example

```json
{
  "status": "success",
  "data": {
    "id_konselor": "uuid...",
    "nama_konselor": "Siti Aminah",
    "kode_konselor": "K001",
    "email": "siti@sekolah.com",
    ...
  }
}
```

---

## 4. Delete Konselor

- **URL**: `DELETE /admin/pendidikan/akademik/konselor/destroy`
- **Auth**: Session
- **Controller**: `KonselorController#destroy`

> Description: Menonaktifkan data konselor.
> Validasi: Data tidak dapat dihapus jika Konselor sudah memiliki **Riwayat Konseling** dengan siswa.
> Efek Samping: Akun user terkait akan dinonaktifkan (`active = 2`)

### Parameters

| Param         | Tipe | Wajib | Keterangan                     |
| :------------ | :--- | :---- | :----------------------------- |
| `id_konselor` | UUID | Ya    | ID Konselor yang akan dihapus. |

---

## 5. Restore Konselor

- **URL**: `POST /admin/pendidikan/akademik/konselor/restore`
- **Auth**: Session
- **Controller**: `KonselorController@restore`

> Description: Mengembalikan data konselor yang sudah dihapus.
> Efek Samping: Akun user terkait akan diaktifkan kembali (`active = 1`)

### Parameters

| Param       | Tipe | Wajib | Keterangan                        |
| :---------- | :--- | :---- | :-------------------------------- |
| id_konselor | UUID | Ya    | ID Konselor yang akan dipulihkan. |
