# Pendaftaran & State Machine (Detail)

Modul ini adalah _Core Logic_ pendaftaran siswa. Menggunakan pendekatan _Wizard_ (bertahap 1-4). State pendaftaran dikunci menggunakan tabel `progres_pendaftaran`.

> **⚠️ SECURITY AUDIT SUMMARY**
>
> 1.  **Vulnerable IDOR**: Semua endpoint `POST` menerima `id_calon_siswa` dari input user tanpa memvalidasi kepemilikan sesi.
> 2.  **Payment Bypass**: Logic `next_step` di Step 3 tidak memvalidasi status pembayaran di sisi server.
> 3.  **Performance**: Upload file ke Google Drive bersifat _Synchronous_. Request akan timeout jika koneksi lambat.

## 1. Get View & Data Step

Mengambil data (JSON) untuk merender halaman _wizard_ berdasarkan nomor step.

- **URL**: `GET /dashboard/tahap-pendaftaran/step/{stepNumber}`
- **Auth**: Session Cookie
- **Controller**: `showStep($stepNumber)`

### Logic Check

1.  **State Gatekeeper**: Mengecek tabel `progres_pendaftaran`. Jika user mencoba akses Step 4 tapi Step 2 belum 'selesai' (status 2), user akan di-_redirect_ mundur.
2.  **Dynamic Loader**: Memanggil method `data_step_{n}()` secara dinamis.

### Parameters

| Param        | Tipe    | Wajib | Keterangan                 |
| :----------- | :------ | :---- | :------------------------- |
| `stepNumber` | Integer | Ya    | Tahapan (1, 2, 3, atau 4). |

---

## 2. Store Step 1: Formulir Awal

Menyimpan biodata dasar Calon Siswa dan Orang Tua. Endpoint ini juga menginisialisasi data `DataLengkap` jika belum ada.

- **URL**: `POST /dashboard/tahap-pendaftaran/step/1`
- **Auth**: Session Cookie
- **Method**: `store_step_1`

### Request Body (Payload)

Parameter ini divalidasi ketat menggunakan `Validator::make`.

#### Identitas Siswa

| Param            | Tipe   | Wajib  | Validasi/Keterangan                      |
| :--------------- | :----- | :----- | :--------------------------------------- |
| `id_calon_siswa` | UUID   | **Ya** | `exists:calon_siswa`. **⚠️ Rawan IDOR**. |
| `nama_siswa`     | String | Ya     | Max 255 chars.                           |
| `nik`            | String | Ya     | `digits:16` (Harus angka 16 digit).      |
| `no_kk`          | String | Ya     | `digits:16` (Harus angka 16 digit).      |
| `nisn`           | String | Ya     | Max 30 chars.                            |
| `npsn`           | String | Ya     | Max 30 chars.                            |
| `email`          | String | Ya     | Format Email valid.                      |
| `phone_number`   | String | Ya     | `regex:/^[0-9]+$/` (Hanya angka).        |
| `jenis_kelamin`  | String | Ya     | `in:L,P`.                                |
| `tempat_lahir`   | String | Ya     | Max 30 chars.                            |
| `tanggal_lahir`  | Date   | Ya     | Format YYYY-MM-DD.                       |
| `kwn`            | String | Ya     | `in:WNI,WNA`.                            |
| `id_agama_fr`    | UUID   | Ya     | `exists:data_agama`.                     |

#### Pilihan Sekolah

| Param                   | Tipe | Wajib  | Validasi/Keterangan                      |
| :---------------------- | :--- | :----- | :--------------------------------------- |
| `jenis_pendaftaran`     | Int  | Ya     | `in:1,2,3` (1:Baru, 2:Pindahan).         |
| `jenis_program_sekolah` | Int  | Ya     | `in:1,2` (1:Reguler, 2:Boarding).        |
| `id_unit`               | UUID | Ya     | Unit Reguler.                            |
| `id_program_jurusan`    | UUID | Ya     | Jurusan Reguler.                         |
| `id_unit_ps`            | UUID | _Cond_ | Wajib jika `jenis_program_sekolah` != 1. |
| `id_program_jurusan_ps` | UUID | _Cond_ | Wajib jika `jenis_program_sekolah` != 1. |
| `tingkat_pendaftaran`   | Int  | Tidak  | Default 1.                               |

#### Alamat & Orang Tua

| Param             | Tipe   | Wajib | Validasi/Keterangan |
| :---------------- | :----- | :---- | :------------------ |
| `id_provinsi_fr`  | UUID   | Ya    | Wilayah.            |
| `id_kabupaten_fr` | UUID   | Ya    | Wilayah.            |
| `id_kecamatan_fr` | UUID   | Ya    | Wilayah.            |
| `desa_kelurahan`  | String | Ya    | Max 50 chars.       |
| `alamat`          | String | Ya    | Text bebas.         |
| `rt`              | String | Ya    | Regex angka, max 5. |
| `rw`              | String | Ya    | Regex angka, max 5. |
| `nama_ayah`       | String | Ya    | Max 50 chars.       |
| `nik_ayah`        | String | Ya    | `digits:16`.        |
| `nama_ibu`        | String | Ya    | Max 50 chars.       |
| `nik_ibu`         | String | Ya    | `digits:16`.        |
| `nama_wali`       | String | Tidak | Max 50 chars.       |
| `nik_wali`        | String | Tidak | `digits:16`.        |
| `asal_sekolah`    | String | Tidak | Max 100 chars.      |
| `mengetahui`      | String | Tidak | Max 100 chars.      |
| `alasan`          | String | Tidak | Max 100 chars.      |

---

## 3. Store Step 2: Persetujuan

Menandai user telah menyetujui "Syarat & Ketentuan". Trigger notifikasi via Email/WhatsApp/Database.

- **URL**: `POST /dashboard/tahap-pendaftaran/step/2`
- **Auth**: Session Cookie
- **Method**: `store_step_2`

### Request Body

| Param               | Tipe    | Wajib | Validasi/Keterangan             |
| :------------------ | :------ | :---- | :------------------------------ |
| `setuju_perjanjian` | Boolean | Ya    | Harus bernilai `1` atau `true`. |

---

## 4. Store Step 3: Pembayaran & Beasiswa

Endpoint ini memiliki **dua fungsi (Switch Logic)** berdasarkan parameter `type`.

- **URL**: `POST /dashboard/tahap-pendaftaran/step/3`
- **Auth**: Session Cookie
- **Method**: `store_step_3`

### Request Body (Umum)

| Param  | Tipe   | Wajib | Keterangan                         |
| :----- | :----- | :---- | :--------------------------------- |
| `type` | String | Ya    | `in:pengajuan_beasiswa,next_step`. |

### Skenario A: `type` = "pengajuan_beasiswa"

Digunakan untuk upload bukti dokumen beasiswa. Nama parameter bersifat dinamis berdasarkan ID Beasiswa dari database.

| Param                                | Tipe | Wajib  | Keterangan                                         |
| :----------------------------------- | :--- | :----- | :------------------------------------------------- |
| `id_beasiswa_registrasi_detail_{ID}` | UUID | Tidak  | ID Detail Beasiswa yang dipilih.                   |
| `id_dokumen_fr_{ID}`                 | File | _Cond_ | Wajib jika detail beasiswa dipilih. (PDF/JPG/PNG). |

### Skenario B: `type` = "next_step"

Digunakan untuk konfirmasi lanjut ke tahap 4.

> **⚠️ LOGIC WARNING:** Logic ini hanya mengupdate flag `dokumen=1` dan `biodata=1` di database **tanpa** mengecek status pembayaran.

| Param                      | Tipe | Wajib | Keterangan                    |
| :------------------------- | :--- | :---- | :---------------------------- |
| (Tidak ada param tambahan) | -    | -     | Cukup kirim `type=next_step`. |

---

## 5. Store Step 4: Finalisasi & Dokumen

Menyimpan data rinci (Data Lengkap, Data Wali Rinci) dan melakukan upload file ke Google Drive.

- **URL**: `POST /dashboard/tahap-pendaftaran/step/4`
- **Auth**: Session Cookie
- **Method**: `store_step_4`

### Request Body (Dokumen Upload)

Validasi file bersifat **Conditional**: Jika file sudah ada di DB sebelumnya, maka `nullable`. Jika belum, `required`.

| Param                 | Tipe | Format         | Max Size |
| :-------------------- | :--- | :------------- | :------- |
| `file_foto`           | File | JPG, JPEG, PNG | 2MB      |
| `file_ijazah`         | File | PDF, JPG, PNG  | 2MB      |
| `file_skhun`          | File | PDF, JPG, PNG  | 2MB      |
| `file_akte_lahir`     | File | PDF, JPG, PNG  | 2MB      |
| `file_kartu_keluarga` | File | PDF, JPG, PNG  | 2MB      |
| `file_perjanjian...`  | File | PDF, JPG, PNG  | 2MB      |
| `file_pernyataan...`  | File | PDF, JPG, PNG  | 2MB      |
| `file_tes_...`        | File | PDF, JPG, PNG  | 2MB      |

### Request Body (Biodata Rinci - Data Lengkap)

Selain data Step 1 yang dikirim ulang, terdapat data tambahan berikut:

| Param                  | Tipe    | Wajib | Keterangan              |
| :--------------------- | :------ | :---- | :---------------------- |
| `nama_panggilan`       | String  | Ya    | Max 30 chars.           |
| `anak_ke`              | Numeric | Tidak | Urutan anak.            |
| `js_kandung`           | Numeric | Tidak | Jumlah saudara kandung. |
| `js_tiri`              | Numeric | Tidak | Jumlah saudara tiri.    |
| `js_angkat`            | Numeric | Tidak | Jumlah saudara angkat.  |
| `bahasa`               | String  | Tidak | Bahasa sehari-hari.     |
| `berat_t`, `berat_m`   | Numeric | Tidak | Berat badan (kg).       |
| `tinggi_t`, `tinggi_m` | Numeric | Tidak | Tinggi badan (cm).      |
| `darah`                | String  | Tidak | Golongan darah.         |
| `penyakit`             | String  | Tidak | Riwayat penyakit.       |
| `kebutuhan_khusus`     | String  | Tidak | -                       |
| `no_kip`, `nama_kip`   | String  | Tidak | Data bantuan sosial.    |
| `no_kps`, `no_kks`     | String  | Tidak | Data bantuan sosial.    |
| `bank`, `no_rekening`  | String  | Tidak | Data bank siswa.        |

### Request Body (Data Rinci Orang Tua)

Field ini berlaku untuk suffix `_ibu`, `_ayah`, dan `_wali`.

| Param                  | Tipe   | Wajib | Keterangan          |
| :--------------------- | :----- | :---- | :------------------ |
| `id_pendidikan_{X}_fr` | UUID   | Tidak | ID Data Pendidikan. |
| `pekerjaan_{X}`        | String | Tidak | Jenis pekerjaan.    |
| `penghasilan_{X}`      | String | Tidak | Range penghasilan.  |
| `email_{X}`            | String | Tidak | Email orang tua.    |
| `no_hp_{X}`            | String | Tidak | No HP.              |
| `alamat_ortu`          | String | Tidak | Alamat lengkap.     |

---

## 6. Helper Data Endpoints (Dynamic Dropdown)

Endpoint pendukung yang dipanggil via AJAX untuk mengisi data referensi (_dropdown_) berdasarkan input **Unit Sekolah** yang dipilih user.

### Get Tingkat (via SIAKAD integration)

Mengambil daftar tingkat pendidikan secara _real-time_ dari API eksternal (SIAKAD).

> **⚠️ PERFORMANCE WARNING:** Endpoint ini melakukan **HTTP Request Synchronous** ke server SIAKAD. Jika server SIAKAD down atau timeout, endpoint ini akan gagal **(Error 500/Timeout)** dan user tidak bisa memilih tingkat pendaftaran

- **URL**: `Get /dashboard/sesi/get-tingkat-from-siakad-pilih-sesi-data`
- **Auth**: `Session Cookie`
- **Controller**: `TahapPendaftaranController@getTingkatFromSiakad`

#### Parameters

| Param   | Tipe | Wajib | Keterangan                                   |
| :------ | :--- | :---- | :------------------------------------------- |
| id_unit | UUID | Ya    | ID Unit Sekolah yang dipilih (dari step 1/4) |

#### Response Example (Success)

```json
{
  "status": "success",
  "code": 200,
  "message": "Data berhasil diambil",
  "data": {
    "all": [1, 2, 3, 4, 5, 6], // List angka tingkat yang tersedia
    "max": 6,
    "min": 1
  }
}
```

#### Response Example (Error SIAKAD)

```json
{
  "status": "error",
  "code": 500, // atau 401 jika auth SIAKAD salah
  "message": "Pengaturan Web Services belum di atur / Internal Server Error"
}
```

---

### Get Jenis Kelas (Program Jurusan)

Mengambil daftar Program Jurusan dari _database_ lokal SIPS berdasarkan Unit.

- **URL:** `GET /dashboard/sesi/get-jenis-kelas-pilih-sesi-data`
- **Auth:** `Session Cookie`
- **Controller:** `TahapanPendaftaranController@getJenisKelas`

### Parameters

| Param      | Tipe | Wajib | Keterangan                                            |
| :--------- | :--- | :---- | :---------------------------------------------------- |
| id_unit    | UUID | Ya    | Wajib dikirim jika konteksnya Unit Reguler            |
| id_unit_ps | UUID | Ya    | Wajib dikirim jika konteksnya Unit Boarding/Pesantren |

> **Catatan**: Controller akan memprioritaskan parameter id*unit. Jika \_Null*, baru membaca id_unit_ps. Query akan memfilter jurusan yang sts_registrasi = 1 (Buka Pendaftaran)

### Response Example

```json
{
  "status": "success",
  "code": 200,
  "message": "Data tingkat berhasil diambil.",
  "data": [
    {
      "id_program_jurusan": "uuid-v4-xxxx",
      "id_unit_fr": "uuid-v4-yyyy",
      "kode_program_jurusan": "REG",
      "nama_program_jurusan": "Reguler",
      "nama_program_jurusan_en": "Regular"
    },
    {
      "id_program_jurusan": "uuid-v4-zzzz",
      "id_unit_fr": "uuid-v4-yyyy",
      "kode_program_jurusan": "TAH",
      "nama_program_jurusan": "Tahfidz",
      "nama_program_jurusan_en": "Tahfidz"
    }
  ]
}
```
