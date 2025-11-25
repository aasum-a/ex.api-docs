# FIX: Error DB Access Denied / Karakter '#' (Deploy)

Dokumen ini adalah _runbook_ untuk mengatasi _error_ 500 terkait koneksi _database_ setelah _deployment_ ke _shared hosting_.

## 1. Gejala (Symptom)

- Website HTTP Error 500.
- Log di `storage/logs/laravel.log` menunjukkan _error_ koneksi _database_ (misal: `Access denied for user '...'`), meskipun kredensial di `.env` sudah dipastikan 100% benar.

## 2. Diagnosis (Root Cause)

_Parser_ `.env` Laravel menganggap karakter `#` (pagar) sebagai awal dari komentar.

Jika _password_ _database_ Anda (yang di-generate cPanel) mengandung karakter `#` (misal: `pass#word123`), Laravel hanya akan membaca `pass` sebagai _password_ dan sisanya (`#word123`) dianggap sebagai komentar.

## 3. Solusi (Solution)

Terdapat dua opsi perbaikan:

- **Opsi 1 (Direkomendasikan):**

  1.  Masuk ke cPanel.
  2.  Buat ulang (generate) _password_ untuk _user_ _database_ Anda.
  3.  Pastikan _password_ baru **tidak mengandung** karakter `#`.
  4.  Perbarui file `.env` di _server_ dengan _password_ baru yang "bersih".

- **Opsi 2 (Workaround):**
  1.  Edit file `.env` di _server_.
  2.  Bungkus nilai `DB_PASSWORD` dengan tanda kutip ganda (`"`).
  ```env
  DB_PASSWORD="pass#word123"
  ```
