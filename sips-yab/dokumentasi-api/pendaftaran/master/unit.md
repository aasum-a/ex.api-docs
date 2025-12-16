# Unit

Modul ini mengelola data unit pendidikan (TK, SD, SMP, SMA, Pesantren). Modul ini memiliki fitur integrasi (Sync) dengan SIAKAD dan manajemen file dokumen legalitas.

## Get List Unit

Mengambil daftar unit beserta statusnya.

- **URL**: `GET /admin/pendaftaran/master/unit`
- **Auth**: Session Cookie

### Query Parameters

| Param              | Tipe | Keterangan                                                             |
| :----------------- | :--- | :--------------------------------------------------------------------- |
| `sts_aktif`        | Int  | Filter status aktif (1=Ya, 2=Tidak).                                   |
| `jenis_unit`       | Int  | Filter jenis sekolah.                                                  |
| `pesantren`        | Int  | Filter status pesantren (1=Ya, 2=Tidak).                               |
| `buka_pendaftaran` | Int  | Filter status PPDB (1=Buka, 2=Tutup).                                  |
| `group_unit`       | Bool | Jika `true`, menggabungkan unit sekolah dengan unit pesantren terkait. |

---

## Store Unit (Create / Update)

Menyimpan data unit dan mengupload dokumen pendukung (Tata Tertib, Perjanjian, dll).

- **URL**: `POST /admin/pendaftaran/master/unit/store`
- **Content-Type**: `multipart/form-data`

### Body Parameters

| Param              | Tipe   | Wajib | Keterangan                                             |
| :----------------- | :----- | :---- | :----------------------------------------------------- |
| `id_unit`          | UUID   | Ya    | ID Unit (Wajib sama dengan SIAKAD jika mode sync).     |
| `kode_daftar`      | String | Ya    | Kode unik pendaftaran (Generate otomatis jika kosong). |
| `urutan`           | Int    | Ya    | Urutan tampilan.                                       |
| `buka_pendaftaran` | Int    | Ya    | `1` (Buka), `2` (Tutup).                               |
| `pdf_tata_tertib`  | File   | Tidak | Format: **PDF**. Max: 20MB.                            |
| `pdf_form_mundur`  | File   | Tidak | Format: **PDF**. Max: 20MB.                            |
| `pdf_perjanjian`   | File   | Tidak | Format: **PDF**. Max: 20MB.                            |
| `gambar`           | File   | Tidak | Logo Unit. Format: **JPG/PNG**. Max: 50MB.             |

> **🔄 Sync Trigger:**
> Jika konfigurasi `siakad_sync` aktif, setiap perubahan data unit di sini akan otomatis dikirim (Push) ke server SIAKAD.

---

## Stream File (Download Securely)

Mengambil file dokumen unit (PDF) secara aman tanpa mengekspos struktur folder asli server.

- **URL**: `GET /admin/pendaftaran/master/unit/get-stream-files`
- **Auth**: Session Cookie

### Query Parameters

| Param     | Tipe | Wajib | Keterangan                                                                                     |
| :-------- | :--- | :---- | :--------------------------------------------------------------------------------------------- |
| `id_unit` | UUID | Ya    | ID Unit pemilik file.                                                                          |
| `type`    | Int  | Ya    | Jenis File: <br>`1` = Tata Tertib <br>`2` = Form Mundur <br>`3` = Form CO <br>`4` = Perjanjian |

### Security Mechanism

Endpoint ini memvalidasi keberadaan file di `Storage::disk('public')` dan mengembalikan respon _stream_ dengan header `Content-Type` yang sesuai. File tidak dapat diakses langsung lewat URL publik.

---

## Sync Data (Integrasi SIAKAD)

Sinkronisasi dua arah (Pull & Push) dengan server SIAKAD.

- **Pull (Get Sync)**: `GET /admin/pendaftaran/master/unit/get-sync`
  - Menarik semua data unit dari SIAKAD.
  - Melakukan _Create_ jika unit belum ada di SIPS.
  - Melakukan _Update_ jika unit sudah ada.
- **Push (Update From Sync)**: `POST /admin/pendaftaran/master/unit/update-from-sync`
  - Webhook receiver untuk menerima perubahan data dari SIAKAD secara _real-time_.
