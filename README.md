Sistem memiliki 3 aktor utama:

1️⃣ Operator

Peran: Administrator yang mengelola data master dan konfigurasi sistem.

Fungsionalitas:

Mengelola Data Guru
Melakukan CRUD (Create, Read, Update, Delete) terhadap data guru seperti:

Nama

NIP

RFID Tag

Informasi terkait lainnya

Login
Autentikasi untuk mengakses fitur sistem secara aman.

Laporan Kehadiran
Melihat, menghasilkan, dan mencetak laporan kehadiran berdasarkan:

Harian

Mingguan

Bulanan

2️⃣ Guru

Peran: Pengguna utama yang melakukan absensi.

Fungsionalitas:

Absensi RFID
Menempelkan kartu RFID pada reader untuk mencatat:

Waktu masuk

Waktu pulang

Sistem otomatis mencatat waktu kehadiran guru.

3️⃣ Kepala Sekolah

Peran: Pimpinan yang memonitor kehadiran untuk evaluasi dan pengambilan keputusan.

Fungsionalitas:

Laporan Kehadiran
Melihat seluruh laporan kehadiran guru untuk analisis dan monitoring.

🔗 Hubungan Antar Use Case

Mengelola Data Guru memiliki relasi <<include>> dengan Login
➝ Operator harus login sebelum mengakses fitur pengelolaan data.

Login merupakan prasyarat untuk seluruh fitur administratif sistem.
