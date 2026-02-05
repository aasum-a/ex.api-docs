# Pindah Kelas & Keluar

Modul ini menangani perpindahan siswa antar kelas (dalam satu jurusan) dan proses siswa keluar dari unit tertentu (misal: Keluar dari Asrama tapi tetap Sekolah, atau sebaliknya).

> ⚠️ LOGIC NOTE:
>
> 1. Status Pindah:
>    - `5`: Pindah Kelas (Antar kelas paralel, misal X-A ke X-B). Siswa didaftarkan ke kelas baru, transkrip nilai di-generate ulang untuk kelas baru, kelas lama dinonaktifkan.
>    - `4`: Keluar Parsial (Keluar dari salah satu unit: Sekolah atau Pesantren).
>    - `1`: Keluar Total / Aktif Kembali (Logic handling status 1 agak kompleks, melibatkan cek status sebelumnya).
> 2. Side Effect: Perpindahan kelas akan memicu generate data transkrip nilai kosong di kelas tujuan.

---

## 1. Get Kelas Tujuan

- **URL**: GET /admin/pendidikan/akademik/pindah-keluar-kelas/get_kelas_tujuan
- **Auth**: Session
- **Controller**: PindahKeluarController@get_kelas_tujuan

> Description: Mengambil daftar kelas yang tersedia untuk tujuan perpindahan.
> Filter: Hanya menampilkan kelas pada Tingkat dan Jurusan yang sama dengan kelas asal (karena ini bukan Mutasi Jurusan)

### Parameters

| Param                  | Tipe | Wajib | Keterangan    |
| :--------------------- | :--- | :---- | :------------ |
| id_kelas_peserta_didik | UUID | Ya    | ID Kelas Asal |

### Response Example

```json
{
  "status": "ok",
  "data": [
    {
      "id_kelas_peserta_didik": "uuid...",
      "kode_kelas": "X-B",
      "nama_wali_kelas": "Siti"
    }
  ]
}
```

---

## 2. Store Pindah/Keluar

- **URL**: `POST /admin/pendidikan/akademik/pindah-keluar-kelas/store`
- **Session**: Session
- **Controller**: `PindahKeluarController@store`

> Description: Memproses perpindahan atau pengeluaran siswa.

**Flow (Pindah Kelas - Status 5)**:

1. Buat record `daftar_kelas_peserta_didik` baru di kelas tujuan.
2. Update `id_kelas_aktif` di tabel `peserta_didik`.
3. Generate transkrip nilai kosong untuk mapel di kelas tujuan.
4. Update status di kelas lama menjadi `5` (Pindah).
5. Simpan log `pindah_keluar`.

**Flow (Keluar Unit - Status 4)**:

1. Cek Unit mana yang ditinggalkan (Sekolah/Pesantren).
2. Update status di `daftar_kelas` unit tersebut menjadi `4` (Keluar).
3. Update `id_kelas_aktif` unit tersebut di `peserta_didik` menjadi NULL.
4. Simpan log `pindah_keluar`.

### Parameters

| Param                               | Tipe   | Wajib    | Keterangan                                           |
| :---------------------------------- | :----- | :------- | :--------------------------------------------------- |
| `id_peserta_didik_fr`               | UUID   | Ya       | ID Siswa                                             |
| `id_kelas_peserta_didik_sebelum_fr` | UUID   | Ya       | Kelas Asal                                           |
| `id_kelas_peserta_didik_sesudah_fr` | UUID   | Optional | Kelas Tujuan (Wajib jika Pindah Kelas)               |
| `sts_pindah_keluar`                 | Int    | Ya       | `5 = Pindah Kelas`, `4 = Keluar Unit`                |
| `sts_keluar_sekolah_psam`           | Int    | Optional | `1 = Pesantren`, `2 = Sekolah` (Wajib jika status 4) |
| `tanggal`                           | Date   | Ya       | Tanggal efektif                                      |
| `keterangan`                        | String | Ya       | Alasan pindah                                        |

