# Ringkasan Teknis: Proyek Profile Company

Dokumen ini menyediakan ringkasan teknis dari proyek _company profile_ "Mahardika Sablon" (internal: `profile-company`).

## 1. Tujuan Proyek

Aplikasi ini berfungsi sebagai _company profile_ dan _landing page_ utama untuk "Mahardika Sablon". Aplikasi ini juga memiliki _admin panel_ (CMS) untuk mengelola konten dinamis di halaman depan, seperti portofolio, klien, dan testimoni.

## 2. Tumpukan Teknologi (Tech Stack)

- **Framework Backend:** Laravel 12
- **Admin Panel:** Filament 4
- **Framework Frontend:** Blade (server-side) dengan Vite
- **Database:** MySQL
- **Lingkungan Lokal:** Laragon (Windows)
- **Lingkungan Produksi:** Shared Hosting (cPanel) di Rumahweb
- **Versi PHP (Kritis):**
  - Lokal (Development): PHP 8.3 (awal), di-downgrade ke 8.2 (untuk rilis)
  - Produksi (Server): PHP 8.2
