# Mutasi Program

Modul ini menangani perpindahan siswa antar Program Jurusan (misal: IPA ke IPS) atau antar Unit (misal: Sekolah ke Pesantren). Modul ini menghitung secara otomatis konversi biaya pendidikan yang sudah dibayarkan di jurusan lama ke jurusan baru.

> ⚠️ CRITICAL LOGIC WARNING
>
> 1. **Reverse Logic Variable**: Dalam kode `get_konversi_tagihan`, variable `$data_kurang` menampung data tagihan baru (Siswa Kurang Bayar), sedangkan `$data_lebih` menampung data refund (Siswa Lebih Bayar) - Jangan tertukar.
> 2. **Hardcoded Rules**: Persentase pengakuan dana dan cicilan saat pindah jurusan di-hardcoded di controller.
> 3. **JSON Foreign Keys**: Relasi ke tabel `pembayaran` dan `retur` disimpang sebagai JSON String, bukan foreign key standar. Join SQL langsung tidak dimungkinkan.

## List Riwayat Mutasi

- **URL**: `GET /admin/pendidikan/akademik/mutasi-program`
- **Auth**: Session
- **Controller**: `MutasiProgramController@index`

> Description: Menampilkan daftar riwayat mutasi siswa. Mendukung filter server-side.

### Parameters

| Param           | Tipe   | Wajib | Keterangan          |
| :-------------- | :----- | :---- | :------------------ |
| `draw`          | Int    | Tidak | Param Datatables    |
| `search[value]` | String | Tidak | Pencarian Global    |
| `page`          | Int    | Tidak | Filter status hapus |

## 2. Form Mutasi

- **URL**: `GET /admin/pendidikan/akademik/mutasi-program/get`
- **Auth**: Session
- **Controller**: `MutasiProgramController@get`

> Description: Mengambil data detail siswa untuk persiapan form mutasi. Sistem akan otomatis mencari opsi Jurusan dan Kelas tujuan yang tersedia di Unit yang sama (atau Unit tujuan jika pindah unit).

### Parameters

| Param            | Tipe | Wajib | Keterangan   |
| :--------------- | :--- | :---- | :----------- |
| id_peserta_didik | UUID | Ya    | ID Siswa     |
| id_unit          | Int  | Ya    | ID Unit Asal |

---

## 3. Hitung Simulasi Konversi Biaya

- **URL**: `GET /admin/pendidikan/akademik/mutasi-program/get-konversi-tagihan`
- **Auth**: Session
- **Controller**: `MutasiProgramController@get_konversi_tagihan`

> Description: Endpoint krusial. Sebelum mutasi disimpan, endpoint ini menghitung simulasi: "kalau pindah sekarang, duit SPP/Gedung yang udah dibayar nasibnya gimana?.
> **Output**: List tagihan yang...
>
> - Impasse: Nominal sama, tinggal pindah buku.
> - Kurang Bayar: Jurusan baru lebih mahal, siswa harus nambah bayar (Generate Tagihan Baru).
> - Lebih Bayar: Jurusan baru lebih murah/sudah lewat periode, uang dikembalikan (Generate Retur).

### Parameters

| Param              | Tipe | Wajib | Keterangan                    |
| :----------------- | :--- | :---- | :---------------------------- |
| `id_peserta_didik` | UUID | Ya    | ID Siswa                      |
| `tgl_mutasi`       | Date | Ya    | Tanggal efektif pindah        |
| `jurusan_awal`     | UUID | Ya    | Jurusan lama                  |
| `jurusan_dituju`   | UUID | Ya    | Jurusan Baru                  |
| `jenis_mutasi`     | Int  | Ya    | `1 = Jurusan`, `2 = Boarding` |

### Response Example

```json
{
  "status": "ok",
  "data": {
    "data_tagihan": {
      "1": [
        // Periode Bulan 1
        {
          "nama_tagihan_lama": "SPP IPA",
          "nominal_bayar_lama": 500000,
          "nama_tagihan_baru": "SPP IPS",
          "nominal_bayar_baru": 450000,
          "selisih_bayar": 50000, // Refund 50rb
          "id_rencana_tagihan_baru": "uuid..."
        }
      ]
    }
  }
}
```

---

## 4. Eksekusi Mutasi

- **URL**: `POST /admin/pendidikan/akademik/mutasi-program/store`\
- **Auth**: Session
- **Controller**: `MutasiProgramController@store`

> Description: Menyimpan mutasi secara permanen.
> Flow Logic:
>
> 1. Simpan Header: Insert ke `mutasi_program_peserta_didik`
> 2. Akademik:
>
> - Update jurusan di master `peserta_didik`
> - Pindahkan siswa ke kelas baru (`daftar_kelas_peserta_didik`)
> - Generate Transkrip Nilai di kelas baru.
> - Matikan status siswa di kelas lama.
>
> 3. Keuangan:
>
> - Loop hasil simulasi konversi (parameter `mutasi`)
> - Jika kurang bayar: Buat invoice `pembayaran` baru
> - Jika lebih bayar: Buat invoice `retur` baru
> - Simpan detail konversi ke tabel `konversi_tagihan_program_siswa`
>
> 4. Trigger: Generate ulang tabel pembantu keuangan

### Parameters

| Param        | Tipe | Wajib    | Keterangan                                                   |
| :----------- | :--- | :------- | :----------------------------------------------------------- |
| mutasi       | JSON | Ya       | Hasil simulasi konversi dari endpoint `get-konversi-tagihan` |
| tgl_mutasi   | Date | Ya       | Tanggal Mutasi                                               |
| kelas_dituju | UUID | Ya       | ID Kelas Baru                                                |
| file_bukti   | file | Optional | SK Mutasi                                                    |

---

## 5. Detail Mutasi

- **URL**: `GET /admin/pendidikan/akademik/mutasi-program/detail-mutasi-program`
- **Auth**: Session
- **Controller**: `MutasiProgramController@detail_mutasi_program`

> Description: Menampilkan detail riwayat mutasi, termasuk rincian konversi keuangan (Tagihan mana yang dipindah, mana yang diretur, mana yang jadi tagihan baru).
