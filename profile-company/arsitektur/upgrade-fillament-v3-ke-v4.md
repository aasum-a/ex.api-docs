# Catatan Keputusan: Migrasi Filament v3 ke v4

Dokumen ini mencatat alasan dan proses keputusan untuk melakukan _upgrade_ paksa (force upgrade) proyek dari Filament v3 ke Filament v4.

## 1. Konteks (Masalah Awal)

Migrasi ini **bukan** bagian dari _roadmap_ awal. Pemicunya adalah investigasi _bug_ kritis pada halaman kustom `GeneralSettings.php` terkait `FileUpload`.

## 2. Investigasi & Opsi Solusi

Dalam upaya perbaikan, solusi _best practice_ adalah menggunakan `spatie/laravel-settings` dan `filament/spatie-laravel-settings-plugin`.

## 3. Konflik Dependensi (Pemicu Utama)

Upaya instalasi _plugin_ Spatie gagal.

- **Diagnosis:** Ditemukan konflik. Proyek menggunakan **Laravel 12** (membutuhkan Filament v4), namun `composer.json` masih mengunci **Filament v3** (`^3.2`). _Plugin_ Spatie membutuhkan v4.

## 4. Keputusan Arsitektur: Force Upgrade

Keputusan strategis diambil: **Melakukan _full upgrade_ paksa ke Filament v4.**

**Alasan:**

1.  **Membayar "Utang Teknis":** Kondisi (Laravel 12 + Filament 3) adalah _technical debt_.
2.  **Membuka Kunci Fitur:** Diperlukan untuk mengadopsi _package_ modern.
3.  **Standarisasi:** Menyelaraskan proyek dengan versi _major_ terbaru.

## 5. Hasil & Konsekuensi

Proses _upgrade_ ini memicu _error_ fatal "Catch-22" pada `artisan`.

- **Lihat FIX:** [FIX: 'Catch-22' Artisan Crash Saat Upgrade Filament v4](../troubleshooting/fix-artisan-crash-upgrade.md)

Ironisnya, instalasi `spatie/laravel-settings` tetap gagal setelah _upgrade_ (karena _cache_ korup). Diputuskan untuk kembali memperbaiki halaman _settings_ kustom manual.
