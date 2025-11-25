# Panduan Setup Lingkungan Lokal

Panduan ini mencakup langkah-langkah untuk menjalankan proyek `profile-company` di lingkungan _development_ lokal.

## 1. Prasyarat

- Laragon (atau XAMPP/MAMP)
- Composer
- Node.js (NPM)
- Pastikan CLI PHP Anda tervalidasi di versi 8.2 (sesuai target produksi). Jalankan `php -v`.

## 2. Prosedur Instalasi

1.  **Clone Repository:**

    ```bash
    git clone [repository-url] profile-company
    cd profile-company
    ```

2.  **Instal Dependensi:**

    ```bash
    composer install
    npm install
    ```

3.  **Konfigurasi Environment:**

    - Salin `.env.example` menjadi `.env`.
    - Buat _database_ baru (misal: `mahardika_db`).
    - Atur koneksi `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` di `.env`.

4.  **Inisialisasi Aplikasi:**

    ```bash
    php artisan key:generate
    php artisan migrate --seed
    php artisan storage:link
    ```

5.  **Build Aset Frontend:**
    ```bash
    npm run dev
    ```

## 3. Peringatan Kritis (Wajib Baca)

Lingkungan lokal proyek ini memiliki dua keunikan yang harus ditangani:

### 1. Masalah CORS (Laragon vs Artisan Serve)

**JANGAN** gunakan `php artisan serve`. Menggunakan `artisan serve` (`127.0.0.1:8000`) akan menyebabkan _bug preview_ `FileUpload` di admin panel (error CORS).

**Solusi:**

- Akses proyek **HANYA** melalui _pretty URL_ yang disediakan Laragon (misal: `http://profile-company.test`).
- Pastikan `APP_URL` di file `.env` lokal Anda diisi dengan _pretty URL_ yang sama: `APP_URL=http://profile-company.test`

### 2. Kesesuaian Versi PHP

Lingkungan produksi menggunakan PHP 8.2. Jika Anda melakukan _deployment_ (Tahap 1, membuat _build_ lokal), Anda **wajib** menjalankan `composer install/update` menggunakan CLI PHP 8.2, bukan 8.3, untuk menghindari _error_ dependensi di _server_.
