# Tagihan

> in this section...

---

## Tagihan Get

Endpoint ini digunakan untuk mengambil daftar tagihan dengan berbagai filter, atau mengambil detail satu tagihan spesifik.

### Informasi Teknis

- **URL**: `/admin/keuangan/master/tagihan/get`
- **Method**: `GET`
- **Auth**: `Session Cookie`
- **Controller**: `Admin\Keuangan\Master\TagihanController@get`

### Query Parameters

| Parameter       | Tipe   | Keterangan                                                                |
| :-------------- | :----- | :------------------------------------------------------------------------ |
| `id_tagihan`    | String | **Opsional**. Jika diisi, akan mengembalikan detail 1 objek tagihan saja. |
| `search`        | String | **Opsional**. Mencari data berdasarkan nama tagihan.                      |
| `jenis_tagihan` | Int    | Filter berdasarkan jenis (`1`= System, `2`= Manual).                      |
| `sts_hapus`     | Int    | Filter status hapus (`1`= Aktif, `2`= Terhapus). Default: `1`.            |

### Response

**Sukses (`200 OK`) - List Data**

```json
{
  "status": "success",
  "code": 200,
  "message": "Data berhasil ditemukan",
  "data": [
    {
      "id_tagihan": "UUID-1",
      "nama_tagihan": "SPP Juli",
      "frekuensi_tagihan": 3,
      "sts_aktif": 1
    },
    { ... }
  ]
}
```

---

## Tagihan Store

Endpoint ini berfungsi sebagai **Upsert** (Update or Insert). Digunakan untuk membuat tagihan baru ATAU memperbarui tagihan yang sudah ada.

> 🔄 **Fitur Sinkronisasi:**
> Endpoint ini terintegrasi langsung dengan **SIAKAD**. Jika penyimpanan ke Server SIAKAD gagal, maka penyimpanan data di lokal SIPS akan otomatis dibatalkan (_Rollback_) untuk menjaga konsistensi data.

### Informasi Teknis

- **URL**: `/admin/keuangan/master/tagihan/store`
- **Method**: `POST`
- **Auth**: `Session Cookie` (Admin Only)
- **Controller**: `Admin\Keuangan\Master\TagihanController@store`

### Request Parameters

| Parameter                | Tipe   | Wajib?          | Keterangan                                                                          |
| :----------------------- | :----- | :-------------- | :---------------------------------------------------------------------------------- |
| `id_tagihan`             | String | **Kondisional** | Kirim ID ini jika ingin **Mengupdate** data. Kosongkan jika ingin **Membuat Baru**. |
| `nama_tagihan`           | String | **Ya**          | Nama tagihan (Min 5, Max 100 karakter).                                             |
| `nama_tagihan_en`        | String | **Ya**          | Nama tagihan dalam Bahasa Inggris.                                                  |
| `id_kategori_tagihan_fr` | Int    | Tidak           | ID Kategori (Foreign Key).                                                          |
| `sts_aktif`              | Int    | **Ya**          | Status Tagihan.<br>`1` = Aktif<br>`2` = Tidak Aktif                                 |

### Response

### Sukses (`201 Created`)

Berhasil disimpan di lokal SIPS dan tersinkronisasi ke SIAKAD.

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil disimpan"
}
```

### Error Validasi (`422 Unprocessable Content`)

Jika input tidak sesuai dengan format yang telah ditentukan.

```json
{
  "status": "error",
  "code": 422,
  "message": "Data gagal disimpan",
  "data": {
    "nama_tagihan": ["the nama tagihan must be at least 5 characters."]
  }
}
```

### Error Sinkronisasi (`500 Server Error`)

Terjadi jika server SIAKAD down atau menolak data (Data Duplikat). Data lokal tidak akan disimpan.

```json
{
  "status": "error",
  "code": 500,
  "message": "Ditemukan data duplikasi pada database: SPP Bulan Juni..."
}
```

---

## Tagihan Destroy

Endpoint ini digunakan untuk menghapus data tagihan secara "halus" (mengubah status menjadi tidak aktif). Data tidak hilang permanen dari database.

> 🛡️ **Proteksi Data:**
> Sistem akan menolak penghapusan jika:
>
> 1. Tagihan **sedang digunakan** di tabel lain (Relasi Data).
> 2. Tagihan merupakan **Jenis Sistem** (jenis_tagihan = 1)

### Informasi Teknis

- **URL**: `/admin/keuangan/master/tagihan/destroy`
- **Method**: `DELETE`
- **Auth**: `Session Cookie`
- **Controller**: `Admin\Keuangan\Master\TagihanController@destroy`

### Query Parameters

| Parameter | Tipe | Wajib | Keterangan |
| :-------------- | :----- | |:---------------------------------|
| `id_tagihan` | String | Ya | ID Tagihan yang akan dihapus |

### Response

**Success (`201 Created`)**
Status data berubah menjadi terhapus (sts_hapus = 2) dan tersinkronisasi ke SIAKAD

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil dihapus"
}
```

**Error (`422 Unprocessable Content`)**
Jika data sedang digunakan oleh siswa/transaksi lain.

```json
{
  "status": "error",
  "code": 422,
  "message": "Data tidak bisa dihapus karena digunakan di tabel lain"
}
```

---

## Tagihan Restore

Endpoint ini digunakan untuk mengembalikan data tagihan yang sebelumnya dihapus menjadi aktif kembali.

### Informasi Teknis

- **URL**: `/admin/keuangan/master/tagihan/restore`
- **Method**: `POST`
- **Auth**: `Session Cookie`
- **Controller**: `Admin\Keuangan\Master\TagihanController@restore`

### Query Parameters

| Parameter | Tipe | Wajib | Keterangan |
| :-------------- | :----- | |:---------------------------------|
| `id_tagihan` | String | Ya | ID Tagihan yang akan dipulihkan. |

### Response

**Success (`201 Created`)**
Status data berubah menjadi terhapus (sts_hapus = 2) dan tersinkronisasi ke SIAKAD

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil dihapus"
}
```

---

## Tagihan Get-Sync

Endpoint ini memicu proses penarikan data terbaru dari Server **SIAKAD** untuk menyamakan data lokal **SIPS**.

> 🔁 **Mekanisme Sync**:
>
> 1.  **SIPS** akan mengirimkan seluruh data lokalnya ke **SIAKAD**,
> 2.  **SIAKAD** melakukan validasi/merging,
> 3.  **SIAKAD** mengembalikan data master terbaru,
> 4.  **SIPS** mengupdate database lokal

### Informasi Teknis

- **URL**: `/admin/keuangan/master/tagihan/get-sync`
- **Method**: `GET`
- **Auth**: `Session Cookie`
- **Controller**: `Admin\Keuangan\Master\TagihanController@get-sync`

### Response

**Success (`201 Created`)**

```json
{
  "status": "success",
  "code": 201,
  "message": "Data berhasil dihapus"
}
```

### Response

**Error (`401 Created`)**
Jika konfigurasi API Key / URL SIAKAD di file .env belum diatur atau salah.

```json
{
  "status": "error",
  "code": 401,
  "message": "Pengaturan Web Services belum diatur"
}
```
