# FIX: FileUpload Preview Tidak Muncul (CORS Lokal)

Dokumen ini adalah _runbook_ untuk mengatasi masalah di mana _preview_ gambar pada `FileUpload` Filament tidak muncul di lingkungan _development_ lokal.

## 1. Gejala (Symptom)

- Pada halaman _admin_, _preview_ untuk gambar yang sudah ada di _database_ tidak tampil (kosong, "Loading...").
- _Browser console_ menunjukkan _error_ **CORS (Cross-Origin Resource Sharing)** atau **404 Not Found** saat me-load gambar.
- **Bahaya Kritis:** Menyimpan _form_ saat _preview_ kosong akan dianggap "file dihapus" dan menimpa _database_ ke `null`.

## 2. Diagnosis (Root Cause)

Masalah ini **bukan** _bug_ kode Filament v4 atau _logic_ `mount()`.

Akar masalahnya adalah **konflik _domain origin_** di lingkungan lokal.

- **Penyebab:** Aplikasi diakses menggunakan `php artisan serve` (origin: `http://127.0.0.1:8000`).
- **Konflik:** _File storage_ di-serve oleh Laragon melalui _origin_ yang **berbeda** (misal: `http://profile-company.test/storage...`).
- **Hasil:** _Browser_ (di `127.0.0.1`) memblokir permintaan gambar dari `profile-company.test` karena melanggar kebijakan _Cross-Origin_.

## 3. Solusi (Solution)

1.  **Hentikan** penggunaan `php artisan serve`.
2.  Pastikan `APP_URL` di file `.env` lokal Anda diisi dengan _pretty URL_ Laragon:
    ```env
    APP_URL=[http://profile-company.test](http://profile-company.test)
    ```
3.  Akses aplikasi _admin_ **hanya** melalui _pretty URL_ tersebut (misal: `http://profile-company.test/admin`).
