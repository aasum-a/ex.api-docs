# Catatan Arsitektur: Halaman Settings Kustom (Model Key-Value)

Dokumen ini mencatat arsitektur yang dipilih untuk halaman "General Settings" dan tantangan teknis spesifik yang muncul setelah migrasi ke Filament v4.

## 1. Konteks Keputusan

Setelah upaya adopsi `spatie/laravel-settings` gagal (lihat: [Catatan Migrasi v3 ke v4](upgrade-v3-ke-v4.md)), diputuskan untuk kembali ke arsitektur awal:

- **Arsitektur:** Menggunakan halaman Filament kustom (`.../GeneralSettings.php`) yang `extends \Filament\Pages\Page`.
- **Model Data:** Halaman ini secara manual mengambil/menyimpan data ke model `PageSetting` (struktur _key-value_).

## 2. Tantangan Teknis di Filament v4

Pendekatan manual ini menimbulkan tantangan pada komponen `FileUpload`.

- **Masalah:** `FileUpload` di v4 mengharapkan _state_ `array` (misal: `['path/logo.jpg']`).
- **Konflik:** Model _key-value_ kita menyimpan data sebagai `string` (misal: `'path/logo.jpg'`).

## 3. Investigasi Awal (Percobaan Gagal)

Upaya awal adalah "menipu" _state_ _form_ di dalam _method_ `mount()` dengan membungkus `string` menjadi `array` secara manual.

- **Kode Percobaan (`mount()`):**
  ```php
  public function mount(): void
  {
      // ... (logika pluck)
      foreach ($fileUploadKeys as $key) {
          if (isset($formDataForFill[$key]) && is_string($formDataForFill[$key])) {
              // Percobaan "membungkus" string menjadi array
              $formDataForFill[$key] = [$formDataForFill[$key]];
          }
      }
      $this->form->fill($formDataForFill);
  }
  ```
- **Hasil:** **GAGAL.** _Preview_ tetap tidak muncul.

## 4. Resolusi (Root Cause Sebenarnya)

Investigasi menemukan _bug_ ini **bukan** disebabkan oleh _logic_ `mount()`. Masalah sebenarnya adalah _bug_ lingkungan lokal (CORS) yang memblokir _browser_ memuat gambar.

- **Lihat FIX:** [FIX: FileUpload Preview Tidak Muncul (CORS Lokal)](../troubleshooting/fix-fileupload-cors.md)
