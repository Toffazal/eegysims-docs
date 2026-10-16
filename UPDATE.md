# Riwayat Pembaruan & Transformasi (UPDATE.md)

Dokumen ini mencatat seluruh transformasi arsitektur, pembaruan skema database, penambahan fitur utama, serta normalisasi tabel yang dilakukan pada versi terbaru **Eegyo SIMS** dibandingkan dengan versi legacy (versi lama).

---

## 📌 Ringkasan Pembaruan Utama (Highlights)

1. **Normalisasi Menyeluruh Database (108+ Tabel)**
   * Standarisasi seluruh *Foreign Key* ke konvensi standar Laravel Eloquent (`nama_relasi_id`).
   * Penghapusan kolom penanggalan legacy ganda (`tanggal` & `waktu` varchar) digantikan oleh native timestamps (`created_at` & `updated_at`).
   * Penerapan relasi *Foreign Key Constraint* dengan *referential integrity* (Cascade Delete / Restrict).
2. **Pencatatan Versi Dinamis & Manajemen Lisensi**
   * Pencatatan versi aplikasi otomatis disinkronkan ke tabel `pengaturan->versi` saat proses pembaruan dijalankan (`UpdateController`).
   * Validasi tipe lisensi (`lifetime` vs `subscription`), validasi kedaluwarsa (`expires_at`), serta proteksi kode menggunakan **ionCube PHP Loader** (PHP 8.3).
   * Standarisasi 13 master roles terintegrasi.
3. **Pembaruan Konsep "Poin Kedisiplinan Siswa"**
   * Transformasi sistem pelanggaran lama: sebelumnya hanya berfokus pada hukuman/pelanggaran. Sekarang mengadopsi sistem poin kedisiplinan yang seimbang: perilaku baik bernilai poin positif (**+**), sedangkan pelanggaran bernilai poin negatif (**-**).
4. **Pembaruan Modul Asesmen & CBT**
   * Pengelompokan seluruh tabel asesmen dengan prefix terstruktur `asesmen_*` (`asesmen_essay`, `asesmen_kuis`, `asesmen_jawaban_essay`, `asesmen_nilai_kuis`, `asesmen_tugas`, dll.).
5. **Kantin Digital & POS Real-time (Laravel Reverb)**
   * Modernisasi kantin sekolah menjadi sistem *cashless* dengan saldo siswa.
   * Penambahan fitur penarikan dana (*Withdraw*) saldo toko kantin ke bendahara.
   * Integrasi WebSocket real-time untuk monitor order kasir tanpa reload halaman.
6. **Integrasi Multi Payment Gateway Modern**
   * Dukungan bawaan untuk **Duitku** dan **Tripay** (QRIS, Virtual Account, Minimarket) dengan webhook callback otomatis.
7. **PWA Native & Multi-Channel Notification**
   * Dukungan penuh Progressive Web App (bisa di-install di smartphone/laptop).
   * Notifikasi via Firebase Cloud Messaging (FCM v1) dan WhatsApp Gateway.

---

## 🗄️ Detail Normalisasi & Perubahan Skema Database

Berikut adalah rincian transformasi tabel dan kolom antara versi lama dan versi baru:

### 1. Modul Asesmen & CBT
* **`essay` ➔ `asesmen_essay`**:
  * Nama tabel diubah menjadi `asesmen_essay` (Class model: `AsesmenEssay`).
  * Kolom `asesmen` diubah menjadi `asesmen_id` (berelasi ke tabel `asesmen`).
* **`jawaban` ➔ `asesmen_jawaban`**:
  * Kolom `asesmen` diubah menjadi `asesmen_id`.
* **`kuis` ➔ `asesmen_kuis`**:
  * Kolom `asesmen` diubah menjadi `asesmen_id`.
  * Kolom `kode` diubah menjadi `kode_kuis`.
* **`jawaban_essay` ➔ `asesmen_jawaban_essay`**:
  * Kolom `asesmen` diubah menjadi `asesmen_id` (relasi ke `asesmen`).
  * Kolom `essay` diubah menjadi `essay_id` (relasi ke `asesmen_essay`).
  * Kolom `kelas` diubah menjadi `kelas_id` (relasi ke `kelas`).
  * Kolom `guru` diubah menjadi `guru_id` (relasi ke `pegawai`).
  * Kolom `siswa` diubah menjadi `siswa_id` (relasi ke `siswa`).
* **`jawaban_kuis` ➔ `asesmen_jawaban_kuis`**:
  * Kolom `asesmen` diubah menjadi `asesmen_id`.
  * Kolom `kelas` diubah menjadi `kelas_id`.
  * Kolom `guru` diubah menjadi `guru_id` (relasi ke `pegawai`).
  * Kolom `siswa` diubah menjadi `siswa_id`.
  * Kolom `kuis` diubah menjadi `kuis_id` (relasi ke `asesmen_kuis`).
* **`nilai_kuis` ➔ `asesmen_nilai_kuis`**:
  * Kolom `asesmen` diubah menjadi `asesmen_id`.
  * Kolom `siswa` diubah menjadi `siswa_id`.
* **`tugas` ➔ `asesmen_tugas`**:
  * Nama tabel dan model diperbarui menjadi `AsesmenTugas`.
  * Kolom `asesmen` diubah menjadi `asesmen_id`.
* **`tugas_siswa` ➔ `asesmen_tugas_siswa`**:
  * Kolom `asesmen` diubah menjadi `asesmen_id`.
  * Kolom `kelas` diubah menjadi `kelas_id`.
  * Kolom `guru` diubah menjadi `guru_id` (relasi ke `pegawai`).
  * Kolom `siswa` diubah menjadi `siswa_id`.
* **Tabel `asesmen`**:
  * Kolom `kelas` diubah menjadi `kelas_id` (relasi ke `kelas`).
  * Kolom `guru` diubah menjadi `guru_id` (relasi ke `pegawai`).

---

### 2. Modul LMS (Learning Management System)
* **`pelatihan` ➔ `lms_pelatihan`**:
  * Kolom `guru` diubah menjadi `guru_id` (relasi ke `pegawai`).
* **`lms_sesi`**:
  * Kolom `pelatihan` diubah menjadi `pelatihan_id` (relasi ke `lms_pelatihan`).
* **`lms_materi`**:
  * Kolom `pelatihan` diubah menjadi `pelatihan_id`.
  * Kolom `sesi` diubah menjadi `sesi_id` (relasi ke `lms_sesi`).
* **`lms_kuis`**:
  * Kolom `sesi` diubah menjadi `sesi_id`.
  * Kolom `kode` diubah menjadi `kode_kuis`.
* **`lms_jawaban`**:
  * Kolom `sesi` diubah menjadi `sesi_id`.
  * Kolom `kode` diubah menjadi `kode_kuis`.
* **`lms_jawaban_kuis`**:
  * Kolom `sesi` diubah menjadi `sesi_id`.
  * Kolom `user` diubah menjadi `siswa_id` (relasi ke `siswa`).
  * Kolom `kuis` diubah menjadi `kuis_id`.
* **`lms_nilai_kuis`**:
  * Kolom `sesi` diubah menjadi `sesi_id`.
  * Kolom `user` diubah menjadi `siswa_id` (relasi ke `siswa`).
* **`lms_tugas`**:
  * Kolom `sesi` diubah menjadi `sesi_id`.
* **`lms_tugas_peserta`**:
  * Kolom `pelatihan` diubah menjadi `pelatihan_id`.
  * Kolom `sesi` diubah menjadi `sesi_id`.
  * Kolom `tugas` diubah menjadi `tugas_id`.
  * Kolom `guru` diubah menjadi `guru_id` (relasi ke `pegawai`).
  * Kolom `user` diubah menjadi `siswa_id` (relasi ke `siswa`).
* **`lms_pembelian`**:
  * Kolom `user` diubah menjadi `siswa_id`.
  * Kolom `guru` diubah menjadi `guru_id`.
  * Kolom `pelatihan` diubah menjadi `pelatihan_id`.
* **`lms_riwayat`**:
  * Kolom `user` diubah menjadi `siswa_id`.
  * Kolom `pelatihan` diubah menjadi `pelatihan_id`.

---

### 3. Modul Kedisiplinan & Bimbingan Konseling (BK)
* **Tabel `pelanggaran`**:
  * Penambahan kolom baru `jenis` dengan tipe `enum('kebaikan', 'pelanggaran')` default `'pelanggaran'`.
* **Tabel `pelanggaransiswa` (Model `PelanggaranSiswa`)**:
  * Kolom `uid` diubah menjadi `siswa_id` (relasi ke `siswa`).
  * Kolom `guru` diubah menjadi `guru_id` (relasi ke `pegawai`).
  * **Perubahan Logika Poin**:
    * *Versi Lama*: Menghitung akumulasi pelanggaran (kebaikan bernilai negatif untuk mengurangi pelanggaran).
    * *Versi Baru*: Menghitung **Total Skor Kedisiplinan**, di mana kebaikan bernilai positif (+poin) dan pelanggaran bernilai negatif (-poin).

---

### 4. Modul Keuangan & Tabungan
* **`bayar_tagihan`**:
  * `idTagihanSiswa` ➔ `tagihansiswa_id`.
  * `idTagihan` ➔ `tagihan_id`.
  * `idSiswa` ➔ `siswa_id`.
* **`tagihansiswa`**:
  * `idSiswa` ➔ `siswa_id`.
  * `idTagihan` ➔ `tagihan_id`.
* **`mutasi`**:
  * `idSiswa` ➔ `siswa_id`.
  * `saldo` ➔ `saldo_id` (relasi ke tabel `saldo`).
* **`tabungan`**:
  * `idUser` ➔ `user_id` (relasi ke `pegawai`).
  * `idSiswa` ➔ `siswa_id` (relasi ke `siswa`).
* **`pembayaran`**:
  * `uid` ➔ `siswa_id`.
* **`pemasukan`**:
  * `user` ➔ `user_id` (relasi ke `pegawai`).
  * `tujuan` ➔ `tujuan_id` (relasi ke `pegawai`).
* **`pengeluaran`**:
  * `user` ➔ `user_id` (relasi ke `pegawai`).
  * `penerima` ➔ `penerima_id` (relasi ke `pegawai`).
  * `sumber` ➔ `sumber_id` (relasi ke tabel `saldo`).

---

### 5. Modul Kantin & POS Digital
* **Tabel `kantin`**:
  * Kolom legacy `idUser` dihapus. Hak akses dikelola secara native melalui tabel `users` dan pivot `role_user`.
* **Tabel `produk`**:
  * `idKantin` ➔ `kantin_id` (relasi ke `kantin`).
* **Tabel `pembelian`**:
  * `idSiswa` ➔ `siswa_id`.
  * `idKantin` ➔ `kantin_id`.
  * Kolom legacy `tanggal` dan `waktu` dihapus, diganti native `created_at` dan `updated_at`.
* **Tabel `pembelian_detail` (Baru)**:
  * Menampung rincian item belanja (produk, qty, harga satuan, subtotal).
* **Tabel `withdraw` (Baru)**:
  * Pengajuan pencairan saldo pendapatan oleh kantin kepada bendahara sekolah (`kantin_id`, nominal, status, bukti bayar).

---

### 6. Modul Presensi & RFID
* **`rekap_absen` ➔ `presensi_siswa`**:
  * Nama tabel diganti menjadi `presensi_siswa`.
  * `uid` ➔ `siswa_id`.
  * `kelas` ➔ `kelas_id`.
* **`presensi_guru`**:
  * `guru` ➔ `guru_id` (relasi ke `pegawai`).
  * `waktu` ➔ `masuk` (jam masuk presensi).
  * `pulang` ➔ `keluar` (jam keluar presensi).
* **Tabel `scanner_rfid`**:
  * Endpoint integrasi untuk alat pembaca kartu RFID IoT gerbang sekolah.

---

### 7. Modul Administrasi Akademik & Pengguna
* **Tabel `kelas`**:
  * `jurusan` ➔ `jurusan_id` (relasi ke `jurusan`).
  * `kelas` ➔ `tingkatan` (misal: VII, VIII, IX, X, XI, XII).
  * `wali_kelas` berelasi formal dengan `pegawai`.
  * `ketua_kelas`, `sekretaris`, dan `bendahara` berelasi formal dengan `siswa`.
  * Kolom login legacy (`username` dan `password` di kelas) dihapus.
  * Ditambahkan kolom timestamps.
* **Tabel `roster`**:
  * `kelas` ➔ `kelas_id`.
  * `guru` ➔ `guru_id`.
* **Tabel `jadwal_ujian`**:
  * `idSiswa` ➔ `siswa_id`.
  * `idRuangan` ➔ `ruangan_id` (relasi ke `ruangan`).
  * Kolom `tanggal` digantikan oleh timestamps.
* **Tabel `beasiswa`**:
  * `siswa` ➔ `siswa_id`.
* **Tabel `berkas`**:
  * `user` ➔ `pegawai_id` (relasi ke `pegawai`).
* **Tabel `ditolak`**:
  * `diSiswa` ➔ `siswa_id` (relasi ke `siswa`).
  * Kolom `tanggal` digantikan oleh timestamps.
* **Tabel `hafalan`**:
  * Standarisasi relasi kolom `siswa_id` ke tabel `siswa`.
* **Tabel `laporan_belajar`**:
  * `guru` ➔ `guru_id` (relasi ke `pegawai`).
* **Tabel `pengaduan`**:
  * `siswa` ➔ `siswa_id`.
  * `guru` ➔ `guru_id`.
  * Kolom `tanggal` dan `waktu` digantikan oleh timestamps.
* **Tabel `peringkat`**:
  * `idSiswa` ➔ `siswa_id`.

---

### 8. Modul Media & Galeri
* **Tabel `banner` (Dihapus)**:
  * Tabel `banner` dihapus karena redundansi fungsi.
  * Seluruh aset media dialihkan ke tabel terpadu `galeri`.
* **Tabel `galeri`**:
  * Kolom `key` diubah menjadi `keygaleri` (contoh value: `banner` untuk slider beranda, `promosi` untuk popup promosi).
* **Penyesuaian Relasi Galeri di Tabel Lain**:
  * Kolom `galeri` pada tabel `ekstrakurikuler` diubah menjadi `keygaleri`.
  * Kolom `galeri` pada tabel `fasilitas` diubah menjadi `keygaleri`.
  * Kolom `galeri` pada tabel `jurusan` diubah menjadi `keygaleri`.

---

## 🚀 Fitur Baru & Peningkatan Arsitektur

### 1. Pelacakan Versi Dinamis & Peningkatan Sistem Lisensi
* **Pencatatan Versi Otomatis**: Melalui pembaruan di `UpdateController`, nomor versi sistem dari paket rilis secara otomatis disimpan dan disinkronkan ke tabel `pengaturan` (kolom `versi`). Halaman Tentang Aplikasi dan footer dashboard selalu merefleksikan versi riil terkini.
* **Manajemen Lisensi Fleksibel (`LicenseService`)**: Mendukung tipe lisensi `lifetime` (tanpa batas waktu) dan `subscription` (dengan validasi tanggal kedaluwarsa `expires_at`).
* **Proteksi Kode ionCube Loader**: Mendukung enkripsi berkas PHP menggunakan ionCube PHP Loader v13+ / v14+ pada environment PHP 8.3.
* **Master Roles Terstandarisasi (13 Roles)**: Mendukung 13 role baku:
  1. `super_admin`
  2. `admin`
  3. `kepala_sekolah`
  4. `guru`
  5. `siswa`
  6. `wali_murid`
  7. `koperasi`
  8. `kantin`
  9. `bendahara`
  10. `sarpras`
  11. `perpustakaan`
  12. `bk`
  13. `petugas_absen`
* **Data Awal Master (Initial Data)**:
  * Saldo bawaan: `SPP`, `Tagihan`, `Koperasi`, `Akademik` (`sistem = 1`).
  * Angkatan default: `2022`, `2023`, `2024`, `2025`, `2026`.

### 2. Kestabilan Sistem & Standar PSR-4 (Kompatibilitas Linux / cPanel)
* Menyelaraskan seluruh nama class Model dan file fisik (misal: `Jenjang` dan `PelanggaranSiswa`) agar memenuhi standar PSR-4 Composer. Mencegah fatal error *Class Not Found* pada hosting Linux yang bersifat *case-sensitive*.
* Penambahan *Defensive Fallback* pada [AppConfigService.php](file:///app/Services/AppConfigService.php) dan [helpers.php](file:///app/Helpers/helpers.php), sehingga sistem tidak melempar error 500 jika dipanggil sebelum database selesai diinisialisasi.

### 3. PWA (Progressive Web App) Native
* Service Worker (`sw.js`) dengan strategi caching aset statis untuk kecepatan buka halaman optimal.
* Web App Manifest lengkap dengan ikon multi-resolusi dan splash screen.

### 4. Pembersihan Dependensi Produksi
* Seluruh paket development (`phpunit`, `mockery`, `fakerphp`, `pint`, `breeze`, dll.) telah dipangkas dari dependensi produksi, menghemat puluhan megabyte ruang penyimpanan dan ribuan file.
* Aset CSS dan JS dikompilasi sebelumnya (*pre-built*) menggunakan Vite sehingga pengguna akhir tidak memerlukan Node.js / NPM di server hosting produksi.

---

> Dokumen ini disusun untuk memudahkan pemahaman transisi sistem. Jika terdapat perubahan tambahan, Anda dapat menambahkannya di bagian catatan manual di bawah ini.

### ✍️ Catatan Tambahan (Manual Notes)
*(Tuliskan catatan khusus atau konfigurasi tambahan sekolah Anda di sini)*
