# Penilaian Kurikulum

Modul ini digunakan untuk mengatur komponen penilaian yang berlaku dalam suatu kurikulum. Data ini akan menjadi "Template" kolom nilai saat guru menginput transkrip nilai.

Fitur Utama:

- Menambahkan jenis penilaian ke dalam kurikulum (e.g. UH-1, UTS, UAS).
- Mengatur urutan penilaian kolom nilai.
- Validasi unik kombinasi {Kurikulum, Semester, Jenis Penilaian}.
- Hard Delete: Menghapus komponen penilaian secara permanen jika belum digunakan.

> ⚠️ LOGIC NOTE
>
> - Data Integrity: Method `_checkIsUsed` sangat krusial. Sistem menolak penghapusan jenis penilaian jika sudah ada **satu saja** siswa yang memiliki nilai (angka/huruf) pada komponen tersebut di `transkrip_nilai_detail`.
> - Dependency: Modul ini bergantung pada Master `Jenis Penilaian'

---

## 1. List Skema Penilaian

- **URL**: `GET /admin/pendidikan/akademik/penilaian-mata-pelajaran-kurikulum`
- **Auth**: Session
- **Controller**: `PenilaianMataPelajaranKurikulumController@index`

> Description: Menampilkan daftar komponen penilaian yang telah didefinisikan untuk suatu kurikulum.

### Parameters

| Param          | Tipe | Wajib | Keterangan                               |
| :------------- | :--- | :---- | :--------------------------------------- |
| `id_kurikulum` | UUID | Ya    | ID Kurikulum yang ingin dilihat skemanya |
| `draw`         | Int  | Tidak | Param Datatables                         |
| `page`         | Int  | Tidak | Filter status hapus                      |

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "nama_jenis_penilaian": "Ulangan Harian 1",
      "semester": "Ganjil",
      "urutan": 1,
      "sts_aktif": 1
    },
    {
      "nama_jenis_penilaian": "Ujian Tengah Semester",
      "semester": "Ganjil",
      "urutan": 2,
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Komponen Penilaian

- **URL**: `POST /admin/pendidikan/akademik/penilaian-mata-pelajaran-kurikulum/store`
- **Auth**: Session
- **Controller**: `PenilaianMataPelajaranKurikulumController@store`

> Description: Menambahkan atu mengubah komponen penilaian.
> Validasi: Dalam satu kurikulum dan semester yang sama, tidak boleh ada dua komponen penilaian dengan Jenis Penilaian yang sama.

### Parameters

| Param                                   | Tipe | Wajib    | Keterangan                        |
| :-------------------------------------- | :--- | :------- | :-------------------------------- |
| `id_penilaian_mata_pelajaran_kurikulum` | UUID | Optional | Isi jika mode edit                |
| `id_kurikulum_fr`                       | UUID | Ya       | ID Kurikulum                      |
| `id_semester_fr`                        | UUID | Ya       | ID Semester berlakuknya nilai ini |
| `id_jenis_penilaian_fr`                 | UUID | Ya       | ID Master Jenis Penilaian         |
| `urutan`                                | Int  | Ya       | Urutan kolom pada transkrip       |
| `sts_aktif`                             | Int  | Ya       | `1 = Aktif`, `2 = Nonaktif`       |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

---

## 3. Get Detail Komponen

- URL: GET /admin/pendidikan/akademik/penilaian-mata-pelajaran-kurikulum/get
- Auth: Session
- Controller: PenilaianMataPelajaranKurikulumController@get

> Description: Mengambil detail satu komponen penilaian.

### Parameters

| Param                                   | Tipe   | Wajib    | Keterangan        |
| :-------------------------------------- | :----- | :------- | :---------------- |
| `id_penilaian_mata_Pelajaran_kurikulum` | UUID   | Optional | ID Data           |
| `search`                                | String | Optional | Keyword pencarian |

---

## 4. Delete Komponen Penilaian

- **URL**: `DELETE /admin/pendidikan/akademik/penilaian-mata-pelajaran-kurikulum/destroy`
- **Auth**: Session
- **Controller**: `PenilaianMataPelajaranKurikulumController@destroy`

> Description: Mengapus komponen penilaian (permanent/hard-delete)
> Validasi: Sistem akan memblokir penghapusan jika komponen ini sudah digunakan untuk menginput nilai siswa (cek tabel `transkrip_nilai_detail`).

### Parameters

| Param                                   | Tipe | Wajib | Keterangan                |
| :-------------------------------------- | :--- | :---- | :------------------------ |
| `id_penilaian_mata_pelajaran_kurikulum` | UUID | Ya    | ID data yang akan dihapus |

---

## 5. Get Semester by Kurikulum

- **URL**: `GET /admin/pendidikan/akademik/penilaian-mata-pelajaran-kurikulum/get_semester`
- **Auth**: Session
- **Controller**: `PenilaianMataPelajaranKurikulumController@get_semester`

> Description: Mengambil daftar semester yang tersedia pada Tahun Ajaran kurikulum tersebut. Digunakan untuk dropdown saat `create`.

### Parameters

| Param          | Tipe | Wajib | Keterangan   |
| :------------- | :--- | :---- | :----------- |
| `id_kurikulum` | UUID | Ya    | ID Kurikulum |
