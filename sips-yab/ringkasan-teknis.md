# Technical Summary

> In this section...

---

## Prerequisites

Sebelum Anda memulai, pastikan Anda telah mempersiapkan hal berikut di mesin lokal Anda:

- **PHP:** `^8.4.x`
- **Node.js:** `^22.x`
- **Composer:** Version 2.x
- **NPM**

> Tips: Menggunakan Laragon (untuk Windows) atau Laravel Valet/Herd (untuk macOS) dapat membantu Anda menginstalasi dependensi ini dengan mudah.

## Installation Guide

Ikuti langkah-langkah di bawah ini untuk menginstalasi dan menjalankan proyek secara lokal.

### 1. Clone the Repository

Pertama, clone repositori ini ke mesin lokal Anda.

#### Ganti dengan URL repositori Anda

```bash
git clone [https://github.com/your-username/your-repository.git](https://github.com/your-username/your-repository.git)
cd your-repository
```

### 2. Install Dependencies

Install semua dependensi yang dibutuhkan untuk PHP dan Node.js.

#### Install PHP dependencies

```bash
composer install
```

#### Install Node.js dependencies

```bash
npm install
```

### 3 Environment Configuration

Langkah ini adalah yang paling kritis. Anda akan mengonfigurasi variabel lingkungan lokal Anda.

#### a. Create `.env` file

Salin file .env.example dan beri nama .env, Semua konfigurasi lokal Anda akan disimpan di dalam file ini.

```bash
cp .env.example .env
```

#### b. Generate Application Key

Setiap aplikasi Laravel memerlukan kunci aplikasi yang unik. maka dari itu jalankan perintah berikut pada terminal.

```bash
php artisan key:generate
```

#### c. Configure Database

Buka file `.env` dan perbarui detail koneksi database untuk sesuai dengan pengaturan lokal Anda.

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE={Your Database Name}
DB_USERNAME=root
DB_PASSWORD=
```

#### d. Generate JWT Secret Key

Proyek ini menggunakan JWT untuk autentikasi. untuk membuat secret key Anda bisa menjalankan perintah berikut:

```bash
php artisan jwt:secret
```

> **Catatan:** Jika kunci rahasia sudah ada dan Anda perlu memperbarui kembali, gunakan flag `--force`:
> `php artisan jwt:secret --force`

#### e. Configure Captcha

Proyek ini menggunakan Captcha Provider untuk melindungi form dari spam. Konfigurasi Captcha Provider di file `.env`.

```ini
CAPTCHA_DRIVER="{Your Captcha Driver}"
CAPTCHA_SITE_KEY="{Your Site Key}"
CAPTCHA_SECRET_KEY="{Your Site Secret}"
CAPTCHA_LOCALE="id"
```

For more details, visit the [official documentation](https://laravel-captcha.rahuldey.dev/docs/2.x/).

#### f. Configure WhatsApp Bot

Aplikasi ini mengintegrasikan dengan bot WhatsApp untuk mengirim notifikasi. Anda perlu menjalankan layanan bot [wa-bot-yab](https://github.com/IT-YAB/wa-bot-yab) secara terpisah.

1.  Clone dan jalankan layanan bot dengan mengikuti instruksi di repositori.
2.  Setelah bot berjalan, perbarui file `.env` dengan URL dan kunci API-nya.

<!-- end list -->

```ini
APP_WHATSAPP_URL={Your Bot URL} #URL of your running bot, for example: http://127.0.0.1:5000/api/broadcast
APP_WHATSAPP_KEY="{Your Secret Key}" #The secret key from the bot's configuration
APP_WHATSAPP_OTP_TIMEOUT=2 #in minute
```

### 4 Database Migration

Jalankan migrasi database untuk membuat semua tabel yang diperlukan. gunakan flag `--seed` yang akan menjalankan seeder untuk mengisi database dengan data awal.

```bash
php artisan migrate:fresh && php artisan db:seed
```

### 5 Create Storage Link

Untuk membuat file yang diunggah di `storage/app/public` dapat diakses dari web, buatlah symlink simbolis.

```bash
php artisan storage:link
```

> **Pemecahan Masalah:** Jika perintah gagal karena direktori `public/storage` sudah ada, hapus direktori tersebut terlebih dahulu dan jalankan perintah kembali atau gunakan Pembuatan Simbolik Manual (Tingkat Lanjut).

### 6 Add Required Boilerplate File

Proyek ini memerlukan file boilerplate tertentu untuk berfungsi dengan benar.

1.  **Unduh file** dari tautan berikut:
    - **Link:** [Unduh File Boilerplate dari Google Drive](https://drive.google.com/file/d/19nWT8-uONGYI61-SHm_bXVX21ktsebxV/view?usp=sharing)
2.  **Letakkan file** di dalam folder `storage/app/` proyek Anda.

---

## Running the Development Server

Proyek ini menggunakan satu perintah tunggal untuk memulai server PHP dan penggabung aset Vite.

**To start the server, run:**

```bash
composer run dev
```

- Aplikasi Anda akan tersedia di **`http://127.0.0.1:8000`**.
- Server Vite akan menangani hot-reloading untuk aset CSS dan JavaScript Anda.

Untuk menghentikan kedua server, tekan **`Ctrl + C`** di terminal.

> **Tips:** Jika Anda ingin menjalankan di port lain, maka ubahlah nomor port di `vite.config.js` untuk Vite dan `.env` untuk PHP.

---

## Available Scripts

Berikut adalah beberapa perintah yang paling umum tersedia untuk proyek ini.

| Perintah              | Keterangan                                                   |
| :-------------------- | :----------------------------------------------------------- |
| `composer run dev`    | Memulai kedua server Vite dan server PHP untuk pengembangan. |
| `npm run dev`         | Memulai hanya server Vite untuk pengembangan aset.           |
| `npm run build`       | Mengkompilasi dan meminimalkan aset untuk produksi.          |
| `php artisan serve`   | Memulai hanya server PHP untuk pengembangan Laravel.         |
| `php artisan test`    | Menjalankan test suite aplikasi (PHPUnit).                   |
| `php artisan migrate` | Menjalankan migrasi database.                                |

### Documentations for Google Drive Storage

untuk mendapatkan update dan dokumentasi lebih lanjut, Anda dapat mengunjungi repo ini `https://github.com/yaza-putu/laravel-google-drive-storage`
