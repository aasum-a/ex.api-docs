# FIX: 'Catch-22' Artisan Crash Saat Upgrade Filament v4

Dokumen ini adalah _runbook_ untuk mengatasi _error_ fatal "Catch-22" yang terjadi selama proses _upgrade_ dari Filament v3 ke v4.

## 1. Gejala (Symptom)

1.  Anda mengubah `composer.json` ke `filament/filament:^4.0`.
2.  `composer install` **sukses** mengunduh _package_ v4 ke `vendor/`.
3.  **Masalah Kritis:** Perintah `php artisan` APAPUN (termasuk `package:discover`) _crash_ dengan `FatalError` atau `TypeError` (biasanya terkait _type hint_ `\BackedEnum`).

### "Catch-22"

Ini adalah "lingkaran setan" karena:

- `artisan` _crash_ karena kode `app/Filament/` (sintaks **v3**) bentrok dengan `vendor/` (sintaks **v4**).
- Perintah perbaikan (`php artisan filament:upgrade`) **tidak bisa dijalankan** karena `artisan` keburu _crash_.

## 2. Diagnosis (Root Cause)

`artisan` (yang _booting_ menggunakan kode v4 dari `vendor/`) mencoba me-load _file_ aplikasi (misal: `app/Filament/ClientResource.php`) yang masih memiliki _method signature_ v3. Perbedaan _breaking change_ ini menyebabkan `FatalError` di level _autoloader_.

## 3. Solusi (Prosedur "Membuka Sumbatan" Manual)

Tujuannya adalah mengedit file v3 secara **minimalis** agar `artisan` bisa _booting_.

1.  **Identifikasi File Biang Kerok:** Lihat di _stack trace error_ `artisan` Anda (misal: `app/Filament/ClientResource.php`).
2.  **Lakukan "Patch" Manual:** Buka file tersebut dan lakukan perubahan minimalis berikut:

    - **Properti `$navigationIcon`:**

      - _Ganti:_ `protected static ?string $navigationIcon ...`
      - _Menjadi:_ `protected static \BackedEnum|string|null $navigationIcon ...`

    - **Properti `$navigationGroup`:**
      - _Ganti:_ `protected static ?string $navigationGroup ...`
      - _Menjadi:_ `protected static \UnitEnum|string|null $navigationGroup ...`

3.  **Hapus File `.BAK`:** Jika ada file PHP "sampah" (misal: `...php.BAK`) di `app/Filament/`, hapus sementara.
4.  **Verifikasi "Sumbatan":** Jalankan `php artisan --version`.
5.  **Eksekusi Upgrader Resmi:** Jika `artisan` berhasil _booting_, **SEGERA** jalankan:
    ```bash
    php artisan filament:upgrade
    ```
6.  Perintah ini akan mengambil alih dan me-_refactor_ sisa proyek Anda secara otomatis.
