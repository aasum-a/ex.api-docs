# Beasiswa Calon Siswa

Endpoint ini digunakan untuk manajemen pengajuan beasiswa oleh calon siswa dan approval oleh admin.

## List Data (Datatables)

- **URL**: `GET /api/beasiswa-calon-siswa`
- **Controller**: `Admin\Keuangan\CalonSiswa\BeasiswaCalonSiswaController@index`
- **Desc**: Mengambil data list pengajuan beasiswa (Server-side Datatables).

## Lihat Detail Pengajuan

- **URL**: `GET /api/beasiswa-calon-siswa/lihat-pengajuan`
- **Param**: `id_calon_siswa_fr` (UUID)
- **Desc**: Mengambil history pengajuan beasiswa spesifik siswa.

## Update Status

- **URL**: `POST /api/beasiswa-calon-siswa/update-status`
- **Body**:
  - `id_beasiswa_calon_siswa`: UUID
  - `sts_pengajuan`: 1 (Pending), 2 (Approved), 3 (Rejected)
- **Desc**: Admin mengubah status persetujuan beasiswa.

## Get Data Sync

- **URL**: `GET /api/sync-data/pengajuan-beasiswa-sips/get`
- **Desc**: Endpoint internal untuk kebutuhan sinkronisasi data pengajuan.

---
