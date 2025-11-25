“SISTEM MONITORING KEHADIRAN MENGGUNAKAN TEKNOLOGI RFID (RADIO
FREQUENCY IDENTIFICATION) DAN ARDUINO UNTUKMEMPERMUDAH
ABSENSI GURU DI SMAS LENTERA” 

1. USE CASE
<img width="821" height="751" alt="image" src="https://github.com/user-attachments/assets/66b1f8d5-5111-4c09-a7b3-4363db2b033a" />
# Sistem Absensi Guru

memiliki 3 aktor utama, seperti berikut :

## a) Operator
**Peran**: Administrator sistem yang mengelola data master dan konfigurasi sistem.

**Fungsionalitas yang dapat dilakukan**:
- Mengelola Data Guru: Melakukan CRUD (Create, Read, Update, Delete) data guru termasuk nama, NIP, RFID tag, dan informasi lainnya
- Login: Mengakses sistem dengan autentikasi untuk keamanan
- Laporan: Melihat, menghasilkan, dan mencetak laporan kehadiran guru dalam berbagai periode (harian, mingguan, bulanan)

## b) Guru
**Peran**: Pengguna utama yang melakukan absensi

**Fungsionalitas yang dapat dilakukan**:
- Melakukan Absensi RFID: Menempelkan kartu RFID untuk mencatat kehadiran (masuk dan pulang)
- Sistem akan otomatis mencatat waktu kehadiran guru

## c) Kepala Sekolah
**Peran**: Pimpinan yang memonitor kehadiran guru untuk keperluan evaluasi dan pengambilan keputusan.

**Fungsionalitas yang dapat dilakukan**:
- Laporan: Melihat dan mengakses laporan kehadiran seluruh guru untuk monitoring dan evaluasi kinerja

## Hubungan antar fungsionalitas:
- Use case "Mengelola Data Guru" memiliki relasi <<include>> dengan "Login", artinya operator harus login terlebih dahulu sebelum dapat mengelola data
- Use case "Login" menjadi prasyarat untuk mengakses fitur-fitur administratif sistem

2. CLASS DIAGRAM
<img width="391" height="910" alt="image" src="https://github.com/user-attachments/assets/389f6194-5147-4dd9-b56b-56c04996b735" />

# Diagram Kelas Sistem Absensi RFID

Diagram ini menjelaskan bagaimana beberapa kelas saling berinteraksi dalam sistem absensi RFID, yaitu: RFID, Arduino, Guru, Server, dan Absen. Masing-masing kelas memiliki atribut dan fungsi (metode) yang mendukung alur sistem.

## a) Kelas RFID
**Atribut**:
- tagID: String → Menyimpan kode unik RFID yang terbaca.

**Metode**:
- scanTag() → Digunakan untuk membaca/mendeteksi kartu RFID.

**Hubungan**:
RFID dibaca oleh Arduino. Setelah tag RFID terbaca, Arduino akan menindaklanjuti dengan mengirim data tag ke server untuk dicocokkan dengan data guru.

## b) Kelas Arduino
**Atribut**:
- port: String → Port komunikasi (USB/Serial).
- statusKoneksi: Boolean → Status apakah Arduino terhubung ke server/komputer.

**Metode**:
- bacaRFID() → Memanggil fungsi dari RFID untuk membaca tag ID.
- kirimDataKeServer() → Mengirim data tagID ke Server.

**Hubungan**:
Arduino membaca RFID dan mengirim data ke Server.

## c) Kelas Guru
**Atribut**:
- idGuru: String
- namaGuru: String
- nip: String
- rfidTag: String → Kode RFID unik milik guru yang terdaftar.

**Metode**:
- getDataGuru() → Mengambil data guru dari database.
- updateGuru() → Memperbarui data guru, termasuk jika ada penggantian kartu RFID.

**Hubungan**:
Server mencocokkan tag dari RFID dengan rfidTag milik objek Guru. Satu guru dapat memiliki banyak data Absen (relasi 1 ke *).

## d) Kelas Server
**Atribut**:
- ipAddress: String
- database: DBConnection → Koneksi ke database sistem absensi.

**Metode**:
- terimaDataRFID() → Menerima data tagID dari Arduino.
- simpanDataAbsen() → Menyimpan data absensi ke tabel Absen.
- generateLaporan() → Menghasilkan laporan absensi.

**Hubungan**:
Server: Menerima data dari Arduino. Mencocokkan tagID dengan data Guru. Menyimpan data absensi ke kelas Absen.

## e) Kelas Absen
**Atribut**:
- idAbsen: String
- idGuru: String
- waktuMasuk: DateTime
- waktuPulang: DateTime
- status: String → Menentukan apakah ini absensi masuk atau pulang.

**Metode**:
- catatMasuk() → Menyimpan waktu masuk.
- catatPulang() → Menyimpan waktu pulang.
- getRiwayatAbsen() → Mengambil catatan absensi guru.

**Hubungan**:
Absen dimiliki oleh Guru dalam hubungan 1 ke banyak (0..*). Server yang bertugas membuat dan menyimpan data Absen ke database.

## Alur Kerja (Ringkasan dari Interaksi Kelas)
RFID membaca kartu → menghasilkan tagID.
Arduino mengambil tagID → lalu mengirimkannya ke Server.
Server menerima data → mencocokkannya dengan rfidTag milik Guru.
Jika cocok → Server membuat/memperbarui data Absen (masuk/pulang).
Server menyimpan data ke database.
Guru memiliki riwayat absensi yang tersimpan di objek Absen.

# 3. Prinsip SOLID yang Dipilih: Single Responsibility Principle (SRP)

SRP menyatakan bahwa setiap kelas harus memiliki satu tanggung jawab utama.

## Penerapan SRP pada Sistem Absensi RFID

Dalam sistem ini, SRP diterapkan dengan memisahkan setiap tanggung jawab ke kelasnya masing-masing:

| Kelas | Tanggung Jawab |
|-------|----------------|
| RFIDReader | Mengambil UID kartu RFID dari Arduino |
| AttendanceService | Memvalidasi data kartu, menentukan status kehadiran (hadir / terlambat), memproses log absensi |
| TeacherRepository | Mengambil atau menyimpan data guru ke database |
| AttendanceRepository | Menyimpan data log absensi ke database |
| NotificationService | Mengirim notifikasi (opsional) ke admin/kepala sekolah |
| TimeProvider | Menyediakan waktu server (agar tidak tergantung waktu di Arduino) |

**Dengan SRP**:
- Perubahan hardware tidak memengaruhi logika bisnis.
- Perubahan aturan absensi tidak mengubah kelas database.
- Sistem lebih mudah diuji dan dikembangkan.

# Tiga (3) Creational Design Pattern yang Digunakan

## a) Factory Method Pattern

**Tujuan**:
Menciptakan objek perangkat pembaca kartu (reader) tanpa mengikat kode pada implementasi tertentu.

**Penerapan dalam Sistem Absensi**:
Sistem mungkin menggunakan RFID sekarang, tetapi di masa depan bisa ditambah:
- Barcode scanner,
- Face recognition,
- NFC smartphone.

Dengan DeviceFactory, Anda dapat membuat:
Reader reader = DeviceFactory.createReader("RFID");

**Keuntungan**:
- Menambah jenis perangkat baru tanpa mengubah kode inti sistem.
- Arduino bisa diganti modulnya tanpa gangguan di sisi aplikasi server.

## b) Singleton Pattern

**Tujuan**:
Memastikan sebuah kelas hanya memiliki satu instance yang digunakan bersama oleh seluruh sistem.

**Penerapan dalam Sistem Absensi**:
- DatabaseConnection
Sistem hanya membutuhkan satu koneksi database yang dipakai seluruh service.
- ConfigurationManager
Menyimpan konfigurasi seperti jam kerja, port serial Arduino, dan lokasi sekolah.

Contoh konsep:
DatabaseConnection db = DatabaseConnection.getInstance();

**Keuntungan**:
- Menghindari pemborosan koneksi database.
- Konfigurasi lebih terkontrol dan konsisten.

## c) Builder Pattern

**Tujuan**:
Membangun objek yang kompleks secara bertahap tanpa constructor panjang.

**Penerapan dalam Sistem Absensi**:
Objek AttendanceRecord (log absensi) biasanya memiliki banyak field:
- idGuru
- namaGuru
- uidRFID
- waktuScan
- status (Hadir / Terlambat / Pulang)
- lokasiDevice
- keterangan opsional

Dengan Builder:
AttendanceRecord record = new AttendanceRecord.Builder()
- .setGuruId("GR001")
- .setUid("A3F94C2")
- .setWaktuScan(LocalDateTime.now())
- .setStatus("Hadir")
- .build();

**Keuntungan**:
- Menghindari constructor yang panjang dan tidak fleksibel.
- Objek log absensi lebih mudah dibuat, dibaca, serta aman dari kesalahan input.
4. ACTIVITY DIAGRAM DAN SEQUENCE DIAGRAM
<img width="1087" height="737" alt="image" src="https://github.com/user-attachments/assets/6be04581-123a-4bba-9873-407c956c284f" />
# Alur Proses Sistem Absensi RFID

Berikut penjelasan alur proses pada diagram tersebut secara berurutan dan sederhana:

## a) Guru menempelkan kartu RFID
Proses dimulai ketika seorang guru menempelkan kartu RFID ke pembaca RFID.

## b) Arduino membaca Tag RFID
Arduino mencoba membaca kode unik (Tag) dari kartu tersebut.

## c) Tag RFID terdeteksi?
Ada dua kemungkinan:

**Jika Tidak terdeteksi**
→ Sistem menampilkan pesan: "Kartu Tidak Terbaca"
→ Proses selesai.

**Jika Terdeteksi**
→ Arduino mengirimkan data Tag RFID ke server.

## d) Server mencocokkan Tag dengan database guru
Server memeriksa apakah Tag yang dikirim Arduino terdapat dalam data guru yang sudah terdaftar.

**Guru terdaftar?**

**Jika Tidak**
→ Tampilkan pesan "Guru Tidak Terdaftar"
→ Proses selesai.

**Jika Ya**
→ Lanjut ke penentuan status absensi.

## e) Penentuan Status (Masuk/Pulang)
Server memutuskan apakah guru tersebut sedang melakukan:
- Absensi masuk, atau
- Absensi pulang,

berdasarkan data absensi sebelumnya.

## f) Server menyimpan data ke tabel Absensi
Informasi absensi (ID guru, waktu, status) dicatat ke database.

## g) Tampilkan notifikasi "Absensi Berhasil"
AttendanceRecord record = new AttendanceRecord.Builder()
    .setGuruId("GR001")
    .setUid("A3F94C2")
    .setWaktuScan(LocalDateTime.now())
    .setStatus("Hadir")
    .build();```
## h) "Laporan Selesai"

<img width="790" height="1007" alt="image" src="https://github.com/user-attachments/assets/052c9166-1306-4df1-98dc-0755bde476a3" />
# Sequence Diagram Sistem Absensi RFID

## a) Login Sequence
**Aktor**: Operator, Guru, Kepala Sekolah  
**Tujuan**: Masuk ke sistem menggunakan akun masing-masing.  

**Alur**:
1. Pengguna (Operator/Guru/Kepala Sekolah) memasukkan username & password
2. Sistem menerima data dan mengirimkan untuk validasi ke Database
3. Database mengembalikan status validasi (berhasil/gagal)
4. Sistem menampilkan hasil login ke pengguna

**Inti proses**: Login memverifikasi identitas pengguna sebelum dapat mengakses menu.

## b) Mengelola Data Guru
**Aktor**: Operator  
**Tujuan**: Menginput, mengedit, atau menghapus data guru.  

**Alur**:
1. Operator meminta form data guru ke Sistem
2. Sistem menampilkan form
3. Operator melakukan input/edit/hapus data guru
4. Sistem mengirim perubahan data ke Database
5. Database menyimpan data dan mengirimkan konfirmasi berhasil
6. Sistem menampilkan notifikasi keberhasilan kepada Operator

**Inti proses**: Operator bertanggung jawab memperbarui data guru yang digunakan dalam absensi RFID.

## c) Melakukan Absensi RFID
**Aktor**: Guru  
**Tujuan**: Melakukan absensi dengan men-tap kartu RFID.  

**Alur**:
1. Guru men-tap kartu RFID pada RFID Reader
2. RFID Reader membaca UID dan mengirimkan ID RFID ke Sistem
3. Sistem mengirim permintaan verifikasi ke Database
4. Database mencari data guru berdasarkan UID RFID
5. Jika data ditemukan, database mengirimkan data guru ke Sistem
6. Sistem mencatat waktu absensi (masuk / pulang) lalu mengirim ke Database
7. Database menyimpan data absensi, lalu mengirim konfirmasi penyimpanan
8. Sistem memberi status kehadiran ke Guru (berhasil)

**Inti proses**: Absensi dilakukan otomatis ketika guru men-tap kartu, dan sistem melakukan verifikasi sebelum menyimpan waktu absensi.

## d) Melihat Laporan
**Aktor**: Operator, Guru, Kepala Sekolah  
**Tujuan**: Menampilkan laporan kehadiran harian/bulanan/tahunan.  

**Alur**:
1. Aktor meminta laporan kehadiran
2. Sistem meneruskan permintaan laporan ke Database
3. Database melakukan query data absensi
4. Database mengirimkan hasil data absensi ke Sistem
5. Sistem menampilkan laporan ke pengguna

**Inti proses**: Semua pihak dapat melihat laporan sesuai hak akses, misalnya guru melihat riwayat pribadi, kepala sekolah melihat seluruh laporan.
# 5. Alur State Lifecycle Data Guru
<img width="1288" height="692" alt="image" src="https://github.com/user-attachments/assets/26f109fa-021b-4e0c-831a-41b21a0f2657" />

Diagram ini menggambarkan perjalanan atau siklus hidup data seorang guru dalam sistem, mulai dari pertama kali dimasukkan hingga dinonaktifkan atau dihapus. Setiap state menunjukkan kondisi data guru, dan panah menandakan perubahan kondisi sesuai tindakan yang terjadi.

## a) Input Data Guru Baru → State: Draft
Proses dimulai ketika admin memasukkan data guru baru ke dalam sistem.

Pada tahap ini:
- Data guru belum aktif
- Menunggu proses validasi dan persetujuan
- Dapat dilakukan koreksi jika ada kesalahan

State ini disebut **Draft**.

## b) Verifikasi dan Persetujuan → State: Active
Setelah data guru diverifikasi dan disetujui oleh pihak berwenang (admin atau operator sekolah), status data berubah menjadi: **Active**

Pada state ini:
- Guru dinyatakan resmi terdaftar dalam sistem
- Data guru dapat digunakan untuk berbagai proses, seperti absensi RFID
- Sistem menganggap guru aktif bekerja di sekolah

## c) Perubahan Data Guru → State: Updated → Kembali ke Active
Jika ada pembaruan atau perubahan pada data guru (misalnya ganti kartu RFID, update nama, NIP, atau jabatan), maka:
- Data berpindah sementara ke state **Updated**
- Setelah perubahan disimpan, status kembali ke **Active**

Alurnya: **Active → Updated → Active**

State ini menunjukkan bahwa data guru bisa diperbarui kapan saja tanpa menonaktifkan keaktifannya.

## d) Guru Cuti / Non-Aktif Sementara → State: Suspended
Jika guru cuti, tidak aktif sementara, atau sedang masa tidak bekerja aktif sementara, maka data berpindah ke state: **Suspended**

Pada state ini:
- Guru sementara tidak berstatus aktif
- Fitur tertentu (contoh: absensi masuk kelas) mungkin dinonaktifkan

## e) Guru Aktif Kembali → Suspended → Active
Jika guru sudah kembali dari cuti atau kembali aktif bekerja, maka:  
**Suspended → Active**

Status guru dipulihkan dan dapat digunakan kembali dalam sistem.

## f) Guru Keluar / Resign → State: Deleted
Jika guru resign, pensiun, atau keluar dari sekolah, maka data guru dipindahkan ke state: **Deleted**

Pada state ini:
- Guru tidak akan muncul dalam daftar aktif
- Tidak bisa digunakan untuk absensi
- Biasanya data disimpan hanya untuk arsip atau keperluan laporan

Ini adalah state akhir dari lifecycle data guru.

## Ringkasan Alur Utam
Draft → Data guru baru dimasukkan
Active → Data sudah diverifikasi dan disetujui
Updated → Perubahan data, lalu kembali aktif
Suspended → Guru non-aktif sementara / cuti
Active → Guru aktif kembali
Deleted → Guru keluar / resign (state akhir)

# Alur State Lifecycle – Data Kehadiran (Attendance Record)
<img width="766" height="786" alt="image" src="https://github.com/user-attachments/assets/9ba97dfa-8e8d-4c03-95a0-45bec8e401e3" />
Diagram ini menggambarkan bagaimana sebuah data kehadiran (absensi) guru terbentuk, divalidasi, diproses, disimpan, dan akhirnya diarsipkan. Setiap state menunjukkan kondisi data absensi dari awal hingga akhir siklusnya.

## a) Guru Tap Kartu RFID → State: Created
Proses dimulai ketika Guru menempelkan kartu RFID ke reader.

Pada state **Created**:
- UID (kode unik kartu), waktu tap, dan identitas awal dibentuk
- Data belum divalidasi
- Hanya menjadi "draft" dari catatan kehadiran

## b) Validasi UID & Jam Scan → State: Validated
Setelah record terbentuk, server melakukan validasi:
- Mengecek apakah UID RFID terdaftar sebagai guru
- Mengecek apakah waktu dan format scan valid
- Memastikan scan tidak duplikat atau rusak

Jika seluruh pengecekan benar, status berubah menjadi: **Validated**

Pada kondisi ini data absensi telah dianggap valid secara teknis, tetapi belum ditentukan status kehadirannya.

## c) Tentukan Status Kehadiran → State: Processed
Setelah data tervalidasi, sistem menentukan jenis absensi berdasarkan waktu:
- Masuk
- Pulang
- Terlambat
- Lembur
- Atau kehadiran sesuai aturan sekolah

Setelah status ditentukan, data masuk ke state: **Processed**

Pada state ini:
- Record sudah lengkap
- Sudah memiliki status kehadiran
- Siap disimpan secara permanen

## d) Simpan Record ke Database → State: Saved
System kemudian menyimpan data absensi ke database.

State berikutnya: **Saved**

Pada state ini:
- Record telah tersimpan permanen
- Dapat ditampilkan di dashboard, laporan, atau rekap harian
- Dapat dikoreksi jika ditemukan kesalahan
- Saved adalah state yang paling sering diakses admin atau sistem

## e) Jika Ada Kesalahan → State: Corrected (Opsional)
Admin bisa melakukan koreksi jika terjadi kesalahan pada absensi, misalnya:
- Guru lupa tap pulang
- RFID error saat tap
- Jam otomatis salah
- Scan ganda
- Guru izin tapi tercatat hadir

Jika admin melakukan koreksi, state berpindah menjadi: **Corrected**

Ini menjamin data absensi tetap akurat meskipun ada kesalahan input awal.

## f) Rekap Harian / Bulanan / Tahunan → State: Archived (State Akhir)
Ketika data absensi sudah masuk rekap:
- Rekap harian
- Rekap bulanan
- Rekap tahunan
- Atau proses backup

Record dipindahkan ke state terakhir: **Archived**

Pada state ini:
- Data tidak lagi bisa diubah (readonly)
- Hanya digunakan untuk laporan, audit, atau arsip sekolah
- Menjadi bagian dari dokumen historis absensi guru

State ini mengakhiri siklus data kehadiran.

## Ringkasan Alur Dalam Bentuk Naratif Singkat
- **Created** → Guru tap kartu RFID, data awal dibuat
- **Validated** → Sistem validasi UID & waktu scan
- **Processed** → Sistem menentukan status absensi (masuk/pulang/terlambat)
- **Saved** → Data absensi disimpan ke database
- **Corrected** (opsional) → Admin memperbaiki jika ada kesalahan, lalu kembali ke Saved
- **Archived** → Data final diarsipkan untuk rekap jangka panjang






