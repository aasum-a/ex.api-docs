# Kelas Peserta Didik

Modul ini adalah entitas penghubung utama yang mendefinisikan "Kelas Aktif" pada Tahun Ajaran tertentu. Modul ini menghubungkan: **Kurikulum** + **Wali Kelas** + **Kelas** + **Tahun Ajaran**.

> ⚠️ **ARCHITECTURE NOTE**
> Controller ini sering dipanggil secara langsung (_direct instatiation_) oleh controller lain (`GenerateNIS` & `DaftarKelas`). Perubahan pada logic atau return value di method `getKelasPesertaDidikByTaAndUnit` dapat menyebabkan _breaking changes_ di modul lain.

## 1. List Kelas

- **URL**: `GET /admin/pendidikan/akademik/kelas-peserta-didik`
- **Auth**: Session
- **Controller**: `KelasPesertaDidikController@index`

> Description: Menampilkan daftar kelas yang aktif.
> Filter: Unit Sekolah, Tingkat, dan Tahun Ajaran.

### Response Example

```json
{
  "status": "success",
  "data": [
    {
      "kode_kelas": "X-A",
      "nama_pengajar": "Budi (Wali Kelas)",
      "nama_kurikulum": "Merdeka Belajar X",
      "tingkat": 10,
      "sts_aktif": 1
    }
  ]
}
```

---

## 2. Store Kelas

- **URL**: `POST /admin/pendidikan/akademik/kelas-peserta-didik/store`
- **Auth**: Session
- **Controller**: `KelasPesertaDidikController@store`

> Description: Membuat atau meng-update definisi kelas aktif.
> Validasi Unik: Sistem melakukan validasi kompleks (_Rule::unique_) untuk mencegah duplikasi kelas dengan kombinasi Kurikulum, Tahun Ajaran, dan Wali Kelas yang sama.

### Parameters

| Param                    | Tipe | Wajib    | Keterangan                   |
| :----------------------- | :--- | :------- | :--------------------------- |
| `id_kelas_peserta_didik` | UUID | Optional | Isi jika edit.               |
| `id_kurikulum_fr`        | UUID | Ya       | ID Kurikulum.                |
| `id_wali_kelas_fr`       | UUID | Ya       | ID Guru Wali.                |
| `id_pendamping_...`      | UUID | Tidak    | Id Pendamping Wali Kelas.    |
| `id_kelas_fr`            | UUID | Ya       | ID Master Kelas              |
| `sts_aktif`              | Int  | Ya       | `1` = Aktif, `2` = Nonaktif. |

### Response Example

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

## 3. Get Opsi Kelas Kosong (Helper)

- **URL**: `GET /admin/pendidikan/akademik/kelas-peserta-didik/get_kelas_kosong`
- **Auth**: Session
- **Controller**: `KelasPesertaDidikController@get_kelas_kosong_peserta_didik`

> Description: Mengambalikan daftar Master Kelas (Ruangan) yang belum digunakan/dipakai pada Tahun Ajaran aktif. Digunakan di form `create` agar user tidak memilih ruang kelas yang sudah dipakai.

### Parameters

| Param                  | Tipe | Wajib    | Keterangan                  |
| :--------------------- | :--- | :------- | :-------------------------- |
| id_kurikulum_fr        | UUID | Ya       | Untuk filter tingkat kelas. |
| id_kelas_peserta_didik | UUID | Optional | ID Kelas saat ini           |

---

## 4. Get Kelas Running

- **Method**: `public static getKelasPesertaDidikRunning($request)`
- **Akses**: Internal (Dipanggil static oleh modul lain, misal Dahboard atau Profil Siswa).

> Description: Mencari ID Kelas di mana seorang siswa terdaftar pada Tahun Ajaran tertentu. Memisahkan logika pencarian untuk Unit Reguler dan Unit Pesantren (Boarding).

### Parameters

| Param            | Tipe | Wajib | Keterangan      |
| :--------------- | :--- | :---- | :-------------- |
| id_peserta_didik | UUID | Ya    | ID Siswa        |
| id_tahun_ajaran  | UUID | Ya    | ID Tahun Ajaran |

### Response Example

```json
{
  "status": "success",
  "data": {
    "id_kelas": { "id_kelas_peserta_didik": "uuid..." }, // Kelas Sekolah
    "id_kelas_ps": { "id_kelas_peserta_didik": "uuid..." } // Kelas Asrama
  }
}
```

---

## 5. Get List Kelas by TA

- **Method**: `getKelasPesertaDidikByTaAndUnit`
- **Akses**: Internal (Dipanggil via `new instance` oleh `GenerateNISController`, `DaftarKelasController`).

> Description: Mengambil Collection lengkap data kelas peserta didik dengan sorting yang kompleks (naturan sort berdasarkan nama kelas dan jurusan)
> Fitur Spesial: `validasi_nis`: jika `TRUE`, hanya mengembalikan kelas yang memiliki siswa tanpa NIS (digunakan modul Generate NIS).

---

## 6. Hapus & Restore

- **URL**: `DELETE .../destroy` & `POST .../restore`
- **Auth**: Session

> **Description**
>
> - Destroy: soft delete, mencegah hapus jika kelas sudah memiliki Absensi, Anggota (Daftar Kelas), Jadwal (Kelas Mapel), atau Transkrip Nilai.
> - Restore: mengembalikan data soft delete. mencegah restore jika data duplikat sudah ada di active records.
