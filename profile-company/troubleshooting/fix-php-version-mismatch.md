# FIX: Error PHP Version Mismatch (Deploy)

Dokumen ini adalah _runbook_ untuk mengatasi _error_ 500 saat _deployment_ yang disebabkan oleh ketidakcocokan versi PHP.

## 1. Gejala (Symptom)

- Website HTTP Error 500 setelah _deployment_ "Paket Build Lengkap" ke _shared hosting_.
- Log _error_ server (bukan `laravel.log`) menampilkan _error_ fatal: `Composer detected issues: ... require a PHP version ">= 8.3.0".`

## 2. Diagnosis (Root Cause)

_Server_ produksi (Rumahweb) berjalan pada PHP 8.2.

"Paket Build Lengkap" (.zip) yang diunggah, khususnya direktori `vendor/` dan file `composer.lock`, di-generate di lingkungan lokal menggunakan CLI PHP 8.3.

_Server_ PHP 8.2 menolak mengeksekusi _package_ yang secara eksplisit membutuhkan PHP 8.3.

## 3. Solusi (Solution)

Ulangi **Tahap 1: Persiapan Lokal** dari panduan _deployment_ dengan sangat teliti.

1.  Pastikan terminal lokal Anda 100% menggunakan CLI PHP 8.2 (verifikasi `php -v`).
2.  Hapus file `composer.lock` dan direktori `vendor/` di lokal.
3.  Jalankan `composer update --no-dev` (menggunakan PHP 8.2).
4.  Buat ulang arsip `.zip` dan unggah kembali ke _server_.
