# Dashboard

Berikut adalah halaman dashboard sistem yang memuatkan statistik proses pendaftaran calon siswa/i Al Ma'soem secara real-time.

Pada halaman ini, Anda dapat melihat jumlah calon siswa/i yang telah mendaftar, jumlah calon siswa/i yang telah terverifikasi, dan jumlah calon siswa/i yang telah selesai pendaftaran.

Halaman ini juga memuatkan fitur untuk mencari data calon siswa/i berdasarkan nama, NISN, atau status pendaftaran.


> **⚠️ LOGIC NOTE**
> Endpoint ini memiliki *Side Effect*: Jika status `formulir` pengguna masih `0` (Belum mulai), sistem akan otomatis mengubahnya menjadi `1` (Sedang mengisi) saat endpoint ini diakses.

## 1. Get Dashboard
Menampilkan halaman utama dashboard beserta status progres pendaftaran saat ini.

- **URL**: `GET /dashboard`
- **Auth**: Session Cookie
- **Controller**: `DashboardController@index`

### Logic Flow
1.  **Session & Auth Check**: Mengambil user aktif dan data sesi anak yang dipilih (`_get_data_anak_session_active`).
2.  **Auto-Start Progress**:
    * Mengecek tabel `progres_pendaftaran`.
    * Jika `formulir == '0'`, update menjadi `'1'` (Start).
3.  **Step Calculation**:
    * Sistem meloop array `$steps` (Formulir -> Persetujuan -> Pembayaran -> Biodata).
    * Mengecek status setiap step di database.
    * Step terakhir yang berstatus `1` (On Progress) atau `2` (Done) akan dianggap sebagai `currentStepNumber`.

### Response Data (View Variables)
Endpoint ini mengembalikan View `pages.main.dashboard.index` dengan variable berikut:

| Variable | Tipe | Keterangan |
| :--- | :--- | :--- |
| `title` | String | Judul Halaman ("Dashboard"). |
| `steps` | Array | Konfigurasi urutan langkah pendaftaran (ID, Judul, Deskripsi, Status). |
| `currentStepNumber` | Integer | Nomor langkah yang sedang aktif (1-4). Digunakan frontend untuk *highlight* timeline/wizard. |

### Structure Object: `$steps`
```json
[
    {
        "number": 1,
        "id": "formulir",
        "status": "2", // 0:Belum, 1:Proses, 2:Selesai
        "title": "Formulir Pendaftaran",
        "description": "...",
        "button_text": "Lanjut..."
    },
    ...
]
