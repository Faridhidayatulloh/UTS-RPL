# UTS-RPL
“SISTEM MONITORING KEHADIRAN MENGGUNAKAN TEKNOLOGI RFID (RADIO
FREQUENCY IDENTIFICATION) DAN ARDUINO UNTUKMEMPERMUDAH
ABSENSI GURU DI SMAS LENTERA” 
1. USE CASE
<img width="821" height="751" alt="image" src="https://github.com/user-attachments/assets/714142c9-724d-4e7c-a17a-03bf012467f7" />
memiliki 3 aktor utama, seperti berikut :
a) Operator
Peran: Administrator sistem yang mengelola data master dan konfigurasi sistem.
Fungsionalitas yang dapat dilakukan:
 Mengelola Data Guru: Melakukan CRUD (Create, Read, Update, Delete)
data guru termasuk nama, NIP, RFID tag, dan informasi lainnya
 Login: Mengakses sistem dengan autentikasi untuk keamanan
 Laporan: Melihat, menghasilkan, dan mencetak laporan kehadiran guru
dalam berbagai periode (harian, mingguan, bulanan)
b) Guru
Peran: Pengguna utama yang melakukan absensi
Fungsionalitas yang dapat dilakukan:
 Melakukan Absensi RFID: Menempelkan kartu RFID untuk mencatat
kehadiran (masuk dan pulang)
 Sistem akan otomatis mencatat waktu kehadiran guru
c) Kepala Sekolah
Peran: Pimpinan yang memonitor kehadiran guru untuk keperluan evaluasi dan
pengambilan keputusan.
Fungsionalitas yang dapat dilakukan:
 Laporan: Melihat dan mengakses laporan kehadiran seluruh guru untuk
monitoring dan evaluasi kinerja
Hubungan antar fungsionalitas:
 Use case "Mengelola Data Guru" memiliki relasi <<include>> dengan "Login",
artinya operator harus login terlebih dahulu sebelum dapat mengelola data
 Use case "Login" menjadi prasyarat untuk mengakses fitur-fitur administratif
sistem
