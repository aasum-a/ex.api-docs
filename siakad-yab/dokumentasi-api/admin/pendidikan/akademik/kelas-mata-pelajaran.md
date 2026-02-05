# Kelas Mata Pelajaran

Modul ini menangani pemetaan Mata Pelajaran ke dalam Kelas dan penugasan Guru Pengajar

> ⚠️ **PERFORMANCE NOTE Logic**: `generate_kelas_mata_pelajaran` menggunakan query berulang di dalam loop (N+1) untuk mengecek ketersediaan data. Perlu optimasi _batch checking_. Selain itu, pemanggilan internal method via `Request` object menambah _overhead_ pemrosesan yang tidak perlu.

## 1. List Mapel per Kelas

- **URL**: `GET /admin/pendidikan/akademik/kelas-mata-pelajaran`
- **Auth**: Session
- **Controller**: `KelasMataPelajaranController@index`

> Description: Menampilkan daftar mata pelajaran yang diajarkan di suatu kelas beserta guru pengampunya. **Auto-Generate**: Jika data mapel belum ada (misalnya kelas baru), sistem akan otomatis men-generate list mapel berdasarkan Struktur Kurikulum kelas tersebut saat endpoint ini dipanggil via AJAX.

### Parameters

| Param                    | Tipe | Wajib | Keterangan                                      |
| :----------------------- | :--- | :---- | :---------------------------------------------- |
| `id_kelas_peserta_didik` | UUID | Ya    | ID Kelas yang akan dilihat                      |
| `page`                   | Int  | Tidak | Filter Status Hapus: `1` = Aktif, `2` = Sampah. |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "nama_mata_pelajaran": "Matematika",
      "nama_pengajar": "Budi Santoso",
      "kelompok": "A (Wajib)",
      ...
    }
  ]
}
```

## 2. Generate Mapel Kelas

- **URL**: `POST /admin/pendidikan/akademik/kelas-mata-pelajaran/store`
- **Method**: `generate_kelas_mata_pelajaran`

> Description: Menyalin struktur mata pelajaran dari Master Kurikulum ke dalam tabel `kelas_mata_pelajaran` untuk kelas spesifik.
> **Logic**:
>
> 1. Ambil list mata pelajaran yang sesuai dengan tingkat dan jurusan kelas.
> 2. Loop mata pelajaran tersebut.
> 3. Cek apakah sudah ada di kelas target.
> 4. Jika belum, insert data baru (Guru kosong `NULL` di awal).

## 3. Update Guru Pengajar

- **URL**: `POST /admin/pendidikan/akademik/kelas-mata-pelajaran/store`
- **Auth**: Session
- **Controller**: `KelasMataPelajaranController@store`

> Description: Mengupdate Guru Pengajar untuk mata pelajaran tertentu di kelas tersebut.
> Side Effect: Sistem juga akan mengupdate field `id_pengajar_fr` di tabel `transkrip_nilai` yang berelasi, memastikan nilai siswa terhubung dengan guru yang baru.

### Parameters

| Param                     | Tipe | Wajib | Keterangan            |
| :------------------------ | :--- | :---- | :-------------------- |
| `id_kelas_mata_pelajaran` | UUID | Ya    | ID Record yang diedit |
| `id_pengajr_fr`           | UUID | Ya    | ID Guru baru          |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data Berhasil disimpan"
}
```

## 4. Hapus Mapel dari Kelas

- **URL**: `DELETE /admin/pendidikan/akademik/kelas-mata-pelajaran/destroy`
- **Auth**: Session
- **Controller**: `KelasMataPelajaranController@destroy`

> Description: Menghapus mata pelajaran dari kelas (Soft Delete).
> Validasi: Data tidak bisa dihapus jika sudah memiliki riwayat **Nilai (Transkrip)** atau **Absensi**.

### Parameters

| Param                     | Tipe | Wajib | Keterangan     |
| :------------------------ | :--- | :---- | :------------- |
| `id_kelas_mata_pelajaran` | UUID | Ya    | ID Mapel kelas |

## 5. Restore Mapel

- **URL**: `POST /admin/pendidikan/akademik/kelas-mata-pelajaran/restore`
- **Auth**: Session
- **Controller**: `KelasMataPelajaranController@restore`

> Description: Mengembalikan mata pelajaran yang sudah dihapus.
> Validasi: Mencegah restore jika mapel yang sama sudah ada di daftar aktif.

## 6. Detail Info Mapel

- **URL**: `GET /admin/pendidikan/akademik/kelas-mata-pelajaran/get`
- **Auth**: Session
- **Controller**: `KelasMataPelajaranController@get`

> Description: Mengambil detail satu mata pelajaran kelas, atau list mata pelajaran berdasarkan filter tertentu.

### Parameters

| Param                     | Tipe | Wajib    | Keterangan                       |
| :------------------------ | :--- | :------- | :------------------------------- |
| `id_kelas_mata_pelajaran` | UUID | Optional | Get Detail Single Data           |
| `id_kelas_peserta_didik`  | UUID | Optional | Get List Mapel di Kelas tersebut |
