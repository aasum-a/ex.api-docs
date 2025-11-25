# FIX: ViteManifestNotFoundException (Deploy)

Dokumen ini adalah _runbook_ untuk mengatasi _error_ 500 terkait Vite Manifest setelah _deployment_ manual ke _shared hosting_.

## 1. Gejala (Symptom)

- Website HTTP Error 500.
- Log di `storage/logs/laravel.log` menunjukkan `ViteManifestNotFoundException` atau `Unable to locate manifest file at ...`.

## 2. Diagnosis (Root Cause)

Aplikasi Laravel (di _server_) tidak dapat menemukan file `build/manifest.json` karena konfigurasi _path_ Vite tidak sesuai untuk _shared hosting_ (terutama setelah memindahkan isi `public/` ke `public_html/`).

## 3. Solusi (Solution)

Solusi terbaik adalah menentukan _path_ aset secara manual (hardcode) di file Blade utama (misal: `app.blade.php` atau `welcome.blade.php`) **sebelum** menjalankan `npm run build` di Tahap 1 _deployment_.

1.  Dapatkan nama file aset yang di-hash dengan menjalankan `npm run build` di lokal. Anda akan melihat file di `public/build/assets/` (misal: `app-Dke83k.css`).
2.  Buka file layout Blade Anda.
3.  Ganti _helper_ `@vite` standar:

    ```blade
    {{-- Ganti ini: --}}
    @vite(['resources/css/app.css', 'resources/js/app.js'])

    {{-- Menjadi ini (sesuaikan path/nama file): --}}
    <link rel="stylesheet" href="{{ asset('build/assets/app-Dke83k.css') }}">
    <script src="{{ asset('build/assets/app-Jk29aD.js') }}"></script>
    ```

4.  Buat ulang "Paket Build Lengkap" `.zip` (yang kini berisi Blade yang sudah di-hardcode) dan lakukan _deployment_ ulang.
