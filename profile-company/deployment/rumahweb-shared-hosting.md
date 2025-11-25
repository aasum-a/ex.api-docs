# Panduan Deployment: Shared Hosting cPanel (Metode Manual)

Dokumen ini menjelaskan prosedur manual untuk melakukan _deployment_ aplikasi Laravel/Filament ke _shared hosting_ (studi kasus: Rumahweb) yang memiliki batasan lingkungan.

## 1. Peringatan & Konteks Lingkungan

Prosedur ini adalah **workaround** dan **bukan** merupakan _best practice_ CI/CD. Ini dirancang khusus untuk mengatasi batasan _server_ berikut:

- Tidak ada akses Terminal/SSH.
- Tidak ada `git`.
- Tidak ada Node.js (proses _build_ aset tidak bisa dilakukan di _server_).
- Versi PHP terbatas (maksimum PHP 8.2).

Tujuan utama metode ini adalah membuat **paket build lengkap** secara lokal (termasuk `vendor/` dan `build/`), yang 100% kompatibel dengan _environment server_, lalu mengunggah arsip tersebut.

## 2. Tahap 1: Persiapan Lokal (Paket Build 8.2)

1.  **Validasi Versi PHP Lokal:** Pastikan CLI PHP lokal Anda menggunakan versi yang **sama** dengan _server_ (8.2). Verifikasi dengan `php -v`.
2.  **Bersihkan Dependensi Lama:** Hapus file `composer.lock` dan direktori `vendor/`.
3.  **Instal Ulang Dependensi PHP:** Jalankan `composer update --no-dev --prefer-dist --optimize-autoloader` (menggunakan CLI PHP 8.2).
4.  **Build Aset Frontend (Vite):** Jalankan `npm install` lalu `npm run build`.
5.  **Buat Arsip Deployment (.zip):** Buat file `.zip` dari proyek. **KECUALIKAN** `.git/`, `node_modules/`, dan `.env`.

## 3. Tahap 2: Eksekusi Deployment (cPanel)

1.  **Bersihkan Direktori Server:** Hapus total direktori aplikasi lama (misal: `laravel_app/`) dan `public_html/`.
2.  **Buat Ulang Struktur:** Buat ulang direktori `laravel_app/profile-company`.
3.  **Unggah & Ekstrak Arsip:** Unggah `.zip` ke `laravel_app/profile-company` dan ekstrak.
4.  **Pindahkan Direktori `public`:** Pindahkan **semua isi** dari `laravel_app/profile-company/public` ke `public_html`.
5.  **Konfigurasi Path `index.php`:** Edit `public_html/index.php`. Sesuaikan dua _path_ agar merujuk ke `laravel_app/profile-company`.
    ```php
    require __DIR__.'/../laravel_app/profile-company/vendor/autoload.php';
    $app = require_once __DIR__.'/../laravel_app/profile-company/bootstrap/app.php';
    ```
6.  **Buat File `.env` Server:** Salin `.env.example` ke `.env` (di `laravel_app/profile-company`). Isi `APP_ENV=production`, `APP_DEBUG=false`, `APP_KEY` (generate dari lokal), `APP_URL`, dan kredensial `DB_*`.
7.  **Eksekusi Perintah Artisan (via Cron Job):**
    - Buat _cron job_ untuk _storage link_:
      `ln -s /home/[user]/laravel_app/profile-company/storage/app/public /home/[user]/public_html/storage`
    - Buat _cron job_ untuk optimasi:
      `php /home/[user]/laravel_app/profile-company/artisan optimize`
    - Jalankan kedua _cron job_ **satu kali**, lalu **HAPUS** _cron job_ tersebut untuk keamanan.

## 4. Troubleshooting

Jika terjadi _error_ 500 setelah _deployment_, konsultasikan _runbook_ spesifik di bawah ini:

- [FIX: Error PHP Version Mismatch](../troubleshooting/fix-php-version-mismatch.md)
- [FIX: Error DB Access Denied](../troubleshooting/fix-db-access.md)
- [FIX: ViteManifestNotFoundException](../troubleshooting/fix-vite-manifest.md)
