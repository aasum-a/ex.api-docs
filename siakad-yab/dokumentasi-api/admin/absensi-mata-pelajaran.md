# Modul Absensi Mata Pelajaran
Modul ini menangani pencatatan kehadiran siswa per mata pelajaran, rekapitulasi, dan pengiriman notifikasi Whatsapp ke orang tua siswa jika tidak hadir.

> ⚠️ **PERFORMANCE NOTE** Endpoint create memiliki masalah `N+1 Query`. Perlu optimasi query untuk memuat data absensi siswa secara *batch* agar tidak membebani database saat memuat kelas dengan jumlah siswa banyak.