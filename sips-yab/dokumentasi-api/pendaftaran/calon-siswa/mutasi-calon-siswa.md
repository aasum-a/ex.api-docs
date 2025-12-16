# Mutasi Calon Siswa

Modul ini memfasilitasi perubahan data pendaftaran (Unit, Jurusan, atau Jalur) bagi siswa yang **sudah melakukan pembayaran awal**.

> **⚠️ TECHNICAL DEBT**
>
> 1.  **Endpoint Store Kosong:** Saat ini endpoint `store` tidak memiliki implementasi logic. Proses finalisasi mutasi kemungkinan ditangani di sisi Frontend atau modul lain.
> 2.  **Shared Logic:** Perhitungan tagihan mutasi menggunakan method statis dari `DataPembayaranController`. Perubahan pada modul pembayaran akan berdampak langsung ke sini.

## Get

Menyiapkan data awal untuk form mutasi, termasuk daftar unit/jurusan tujuan yang tersedia.

- **URL**: `GET /admin/pendaftaran/calon-siswa/mutasi-calon-siswa/create`
- **Auth**: Session Cookie

### Query Parameters

| Param            | Tipe | Wajib | Keterangan                   |
| :--------------- | :--- | :---- | :--------------------------- |
| `id_calon_siswa` | UUID | Ya    | ID Siswa yang akan dimutasi. |

### Logic Data

Sistem akan otomatis mengecualikan (exclude) Unit/Jurusan yang sedang dipilih siswa saat ini dari daftar tujuan, supaya siswa tidak memutasi ke jurusan yang sama.

---

## Get Perbandingan Tagihan

Menghitung estimasi tagihan baru jika mutasi dilakukan. Sistem membandingkan tagihan lama vs tagihan baru.

- **URL**: `POST /admin/pendaftaran/calon-siswa/mutasi-calon-siswa/get-perbandingan-tagihan`

### Body Parameters

| Param                       | Tipe   | Keterangan                               |
| :-------------------------- | :----- | :--------------------------------------- |
| `id_calon_siswa`            | UUID   | ID Siswa.                                |
| `id_unit_dituju`            | UUID   | Unit tujuan mutasi.                      |
| `id_program_jurusan_dituju` | UUID   | Jurusan tujuan.                          |
| `jenis_pendaftaran_dituju`  | Int    | Jalur pendaftaran baru.                  |
| `payment_option`            | String | `normal` (Bulanan), `sekaligus` (Lunas). |
| `periode`                   | JSON   | (Optional) Array bulan cicilan.          |

### Response Data

JSON berisi rincian:

1.  **Tagihan Lama:** Nominal yang sudah ditagihkan/dibayar.
2.  **Tagihan Baru:** Nominal untuk jurusan tujuan.
3.  **Selisih:** Apakah siswa harus nambah bayar (Kurang Bayar) atau ada kelebihan bayar (Lebih Bayar).

```json
{
  "status": "success",
  "data": {
    "total_tagihan_lama": 5000000,
    "total_tagihan_baru": 7500000,
    "selisih": 2500000,
    "status_mutasi": "KURANG_BAYAR"
  }
}
```
