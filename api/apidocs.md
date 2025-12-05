# Dokumentasi API Bug Tracker

Selamat datang di dokumentasi teknis untuk sistem pelaporan Bug.
API ini memungkinkan tim security untuk melapor dan memantau celah keamanan.

## Informasi Dasar

- **Base URL**: `http://127.0.0.1:1001/api` (Localhost)
- **Format Response**: JSON
- **Authentication**: Public (Saat ini belum ada auth)

## Cara Menggunakan

1. Pastikan Anda terhubung ke jaringan internal.
2. Gunakan endpoint `/bugs` untuk melihat atau mengirim laporan.
3. Selalu sertakan header `Accept: application/json`.

## Referensi Lengkap (Endpoints)

Untuk melihat daftar lengkap endpoint, parameter wajib, tipe data, dan contoh request/response secara real-time, silakan akses dokumentasi interaktif kami di bawah ini:

[**Buka Dokumentasi Lengkap via Postman**](https://documenter.getpostman.com/view/25953220/2sB3dMwAYx)

> **Catatan:** Anda bisa langsung mencoba API ini dengan mengklik tombol "Run in Postman" pada link di atas.

## 🚨 Daftar Kode Error

Berikut adalah standar kode error yang mungkin Anda temui:

| Kode  | Arti                  | Kemungkinan Penyebab                     |
| :---- | :-------------------- | :--------------------------------------- |
| `200` | OK                    | Request berhasil.                        |
| `201` | Created               | Data berhasil ditambahkan.               |
| `422` | Unprocessable Content | Data tidak lengkap (Cek validasi input). |
| `500` | Server Error          | Masalah di server (Hubungi Andy).        |
