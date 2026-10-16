# Eegyo SIMS - Enterprise School Information Management System

<p align="center">
  <img src="logo_esims_white.png" alt="Eegyo SIMS Logo" width="250">
</p>

<p align="center">
  <strong>Solusi Sistem Informasi Manajemen Sekolah Terintegrasi, Modern, dan Skalabel</strong>
</p>

<p align="center">
  <a href="https://laravel.com"><img src="https://img.shields.io/badge/Laravel-13.x-FF2D20?style=for-the-badge&logo=laravel" alt="Laravel 13"></a>
  <a href="https://php.net"><img src="https://img.shields.io/badge/PHP-8.3+-777BB4?style=for-the-badge&logo=php" alt="PHP 8.3+"></a>
  <a href="https://mysql.com"><img src="https://img.shields.io/badge/MySQL-8.0+-4479A1?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL"></a>
  <a href="#"><img src="https://img.shields.io/badge/PWA-Ready-5A0FC8?style=for-the-badge&logo=pwa" alt="PWA Ready"></a>
  <a href="#"><img src="https://img.shields.io/badge/WebSocket-Laravel_Reverb-FF2D20?style=for-the-badge" alt="Reverb WebSocket"></a>
</p>

---

## 📌 Tentang Eegyo SIMS

**Eegyo SIMS** (Sistem Informasi Manajemen Sekolah) adalah platform web terpadu self-hosted yang dirancang untuk mengelola seluruh ekosistem operasional lembaga pendidikan, mulai dari tingkat SD/MI, SMP/MTs, SMA/SMK/MA, hingga Pesantren.

Dibangun di atas **Laravel 13.x** dan arsitektur database relasional ternormalisasi (**108+ tabel terstruktur**), aplikasi ini mengintegrasikan seluruh lini operasional: dari Penerimaan Peserta Didik Baru (PPDB), Manajemen Akademik & Rapor, Computer-Based Test (CBT), Learning Management System (LMS), Presensi Digital & IoT RFID, Keuangan & Billing Terintegrasi Payment Gateway, hingga Kantin Cashless & Koperasi.

---

## 🌟 Fitur Utama & Modul Sistem

### 1. 🎓 Manajemen Akademik & Kesiswaan

- **Master Data Terpusat**: Jenjang, angkatan, jurusan, kelas, kurikulum, mata pelajaran, dan pembagian ruang.
- **Buku Induk Digital**: Data lengkap biodata siswa, orang tua, wali, status beasiswa, dan mutasi keluar/masuk.
- **Jadwal & Roster Pelajaran**: Penjadwalan mata pelajaran otomatis per kelas, pemetaan jam mengajar guru, dan kalender akademik.
- **Rapor & Nilai**: Rekap nilai capaian belajar, nilai ekstrakurikuler, catatan wali kelas, dan cetak lembar rapor.

### 2. 📝 Asesmen, CBT & LMS Terintegrasi

- **Asesmen Beragam Tipe**: Mendukung Kuis Pilihan Ganda (auto-grading), Ujian Essay, dan Penugasan Terjadwal.
- **Sistem LMS Modern**: Modul pelatihan, sesi materi interaktif, video pembelajaran, forum diskusi, dan monitoring progres belajar.
- **Bank Soal & Pengacakan**: Manajemen bank soal ujian, timer pengerjaan, dan penguncian anti-kecurangan.

### 3. 💳 Keuangan, Tagihan & Payment Gateway

- **Manajemen Saldo Rekening**: Pemisahan pos dana (SPP, Tagihan Siswa, Koperasi, Kas Sekolah, Tabungan Siswa).
- **Multi Payment Gateway**: Integrasi siap pakai dengan **Duitku** dan **Tripay** (QRIS Dinamis, Virtual Account BCA, BRI, Mandiri, BNI, BSI, dan gerai retail Alfamart/Indomaret).
- **Pembayaran Otomatis & Notifikasi**: Verifikasi callback otomatis (webhook) dan update status lunas seketika.
- **Laporan Finansial Komprehensif**: Buku kas masuk/keluar, jurnal mutasi, dan rekap tunggakan tagihan per kelas/angkatan.

### 4. ⏱️ Presensi Siswa, Guru & Integrasi IoT RFID

- **Presensi Multi-Metode**: Presensi manual guru/piket, barcode/QR scanner, serta dukungan hardware **IoT RFID Scanner**.
- **Presensi Jam Mengajar**: Monitoring presensi guru per mata pelajaran dan rekap kehadiran bulanan.
- **Notifikasi Kehadiran Otomatis**: Pengiriman status keterlambatan atau ketidakhadiran siswa secara real-time ke akun orang tua.

### 5. ⚖️ Poin Kedisiplinan & Bimbingan Konseling (BK)

- **Paradigma Poin Kedisiplinan**: Sistem pencatatan komprehensif yang tidak hanya mencatat poin pelanggaran (minus), tetapi juga menghargai aksi kebaikan dan prestasi siswa (plus).
- **Bimbingan Konseling (BK)**: Portal pengaduan siswa/orang tua, riwayat konseling, pemanggilan orang tua, dan surat peringatan (SP).

### 6. 🍔 POS Kantin Digital & Koperasi Sekolah

- **Cashless School Canteen**: Pembelian makanan/minuman menggunakan saldo digital siswa (berbasis scan ID / barcode).
- **Real-time Live Order**: Pembaruan pesanan kasir seketika menggunakan **Laravel Reverb WebSocket**.
- **Manajemen Mitra Toko & Penarikan Saldo**: Setiap kantin memiliki portal terpisah dan fitur _Withdraw_ saldo dengan validasi bendahara.
- **Koperasi Sekolah**: Pengelolaan stok seragam, buku modul, dan alat tulis.

### 7. 📥 PPDB Online (Penerimaan Peserta Didik Baru)

- **Formulir Pendaftaran Dinamis**: Pengaturan kustom pertanyaan, upload berkas, dan pemilihan jalur pendaftaran.
- **Verifikasi Berkas & Seleksi**: Verifikasi dokumen pendaftar oleh tim panitia.
- **Pembayaran Formulir/Daftar Ulang**: Terintegrasi langsung dengan payment gateway.
- **Kelulusan & Cetak Bukti**: Pengumuman online dan cetak kartu bukti pendaftaran.

### 8. 📱 Progressive Web App (PWA) & Notifikasi Multi-Saluran

- **PWA Native**: Dapat diinstall langsung di perangkat Android, iOS, Windows, dan macOS dengan dukungan offline shell.
- **Push Notification (FCM API v1)**: Notifikasi push langsung ke perangkat pengguna menggunakan Firebase Cloud Messaging.
- **WhatsApp Gateway**: Pengiriman pengumuman dan tagihan via integrasi WhatsApp API.

---

## 👥 14 Multi-Role & Portal Terisolasi

Eegyo SIMS dirancang dengan pemisahan hak akses ketat (_Role-Based Access Control_):

| Role                | Deskripsi Portal                                                                   |
| :------------------ | :--------------------------------------------------------------------------------- |
| **Admin**           | Pengaturan sistem, lisensi, backup database, manajemen pengguna & sekolah.         |
| **Kepala Sekolah**  | Dashboard eksekutif, monitoring statistik akademik, approval berkas & anggaran.    |
| **Tata Usaha (TU)** | Manajemen data induk siswa & pegawai, administrasi persuratan, mutasi & arsip.     |
| **Bendahara**       | Manajemen pos saldo kas, penetapan tagihan/SPP, persetujuan withdraw kantin.       |
| **Guru**            | Pengelolaan kelas, input materi, jadwal ujian/CBT, tugas, dan rekap nilai siswa.   |
| **Wali Kelas**      | Pemantauan kelas binaan, presensi kolektif, poin kedisiplinan, dan cetak rapor.    |
| **Guru Piket**      | Presensi harian gerbang, pemantauan roster, dan pencatatan pelanggaran/kebaikan.   |
| **Guru BK**         | Pengelolaan pengaduan, konseling siswa, dan penanganan tindakan kedisiplinan.      |
| **Panitia PPDB**    | Verifikasi pendaftar baru, seleksi nilai/jalur, dan pengelolaan kuota masuk.       |
| **Koperasi**        | Pengelolaan katalog barang, inventaris seragam/buku, dan transaksi penjualan.      |
| **Kantin**          | POS kasir penjualan makanan/minuman, riwayat belanja, dan permohonan tarik saldo.  |
| **Siswa**           | Portal belajar, jadwal, nilai, CBT ujian online, saldo digital, dan kartu pelajar. |
| **Pendaftar**       | Portal calon siswa baru untuk pengisian formulir PPDB dan upload berkas.           |
| **Wali Murid**      | Pemantauan absensi, tagihan SPP, dan perkembangan akademik anak dari rumah.        |

---

## 💻 Kebutuhan Sistem (System Requirements)

- **PHP**: Versi **8.3+** (Sesuai spesifikasi `composer.json` : `^8.3`)
- **Ekstensi PHP Wajib**:
    - `BCMath`
    - `Ctype`
    - `cURL`
    - `DOM / XML`
    - `Fileinfo`
    - `Filter`
    - `GD` (untuk pengolahan gambar, foto profil, dan barcode/QR)
    - `Mbstring`
    - `OpenSSL`
    - `PCRE`
    - `PDO` & `pdo_mysql`
    - `Tokenizer`
- **Database**: MySQL 8.0+ atau MariaDB 10.4+
- **Web Server**: Apache (dengan `mod_rewrite` aktif) atau Nginx
- **Composer**: Versi 2.x

---

## 🚀 Panduan Instalasi Cepat

Eegyo SIMS dilengkapi dengan **Web Interactive Installation Wizard**, sehingga Anda tidak perlu mengetikkan puluhan perintah terminal saat pertama kali memasang aplikasi.

### Langkah 1: Unduh / Clone Repositori

Ekstrak arsip rilis Eegyo SIMS atau clone repositori ke web server Anda:

```bash
# Contoh direktori pada WampServer / XAMPP / Apache:
cd c:/wamp64/www/
git clone -b fresh https://github.com/Toffazal/eegyosims.git
cd eegyosims
```

### Langkah 2: Persiapkan Environment (.env)

Salin berkas `.env.example` menjadi `.env`:

```bash
cp .env.example .env
```

Buat sebuah database kosong di MySQL/phpMyAdmin (misalnya: `eegyosims_db`). Buka berkas `.env` lalu sesuaikan konfigurasi koneksi database:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=eegyosims_db
DB_USERNAME=root
DB_PASSWORD=
```

### Langkah 3: Jalankan Web Installation Wizard

1. Buka browser dan arahkan ke alamat URL aplikasi Anda (misal: `http://localhost/eegyosims` atau virtual host Anda).
2. Sistem akan **secara otomatis mendeteksi status belum terinstall** dan mengarahkan Anda ke antarmuka wizard: `http://localhost/eegyosims/install`.
3. Ikuti 5 langkah mudah di layar:
    - **Langkah 1 (Sistem & Lingkungan)**: Validasi ekstensi PHP, koneksi database, dan pembuatan symlink storage.
    - **Langkah 2 (Profil Sekolah)**: Input nama sekolah, NPSN, jenjang, alamat, dan kontak resmi.
    - **Langkah 3 (Pengaturan Aplikasi)**: Pilih tema visual default, nominal uang sekolah, dan format penomoran dokumen.
    - **Langkah 4 (Akun Super Administrator)**: Buat username, email, dan password untuk Administrator utama pertama.
    - **Langkah 5 (Eksekusi Otomatis)**: Sistem menjalankan 108 migrasi database dan pengisian seeder sistem secara background.
4. Klik tombol **"Masuk ke Aplikasi"** dan login menggunakan akun Admin yang baru saja Anda buat!

---

## 🛠️ Instalasi Alternatif via CLI (Developer Mode)

Bagi pengembang yang ingin melakukan instalasi penuh melalui terminal:

```bash
# 1. Install dependensi
composer install

# 2. Generate Application Key
php artisan key:generate

# 3. Jalankan migrasi dan seeder awal
php artisan migrate --force
php artisan db:seed --class=InitialSystemSeeder --force

# 4. Buat symlink storage publik
php artisan storage:link

# 5. Jalankan server lokal & websocket (opsional)
php artisan serve
php artisan reverb:start
```

---

## 🔄 Riwayat Perubahan & Pembaruan

Untuk melihat daftar lengkap transformasi dari versi legacy ke versi modern saat ini (termasuk normalisasi 108 tabel, perubahan skema kolom, dan fitur baru), silakan pelajari dokumen:
👉 [**UPDATE.md (Changelog & Migration History)**](UPDATE.md)

---

## 🔒 Keamanan & Praktik Terbaik

- **Mode Produksi**: Pastikan `APP_DEBUG=false` dan `APP_ENV=production` pada berkas `.env` di server live.
- **Izin Direktori**: Berikan hak akses tulis (_write permissions_) pada folder `storage/` dan `bootstrap/cache/`.
- **Private Storage**: Folder `storage/app/public` hanya boleh diakses melalui link publik (`public/storage`).

---

## 📄 Hak Cipta & Dukungan Teknis

Aplikasi ini dikembangkan dan dipelihara secara aktif.

- **Website Resmi**: [https://eegyosims.com](https://eegyosims.com)
- **Dokumentasi & Bantuan**: Silakan ajukan Issue di repositori atau hubungi tim pengembang melalui portal resmi.

---

<p align="center">Made with ❤️ for Better Education Management.</p>
