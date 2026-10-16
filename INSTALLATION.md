# 📖 Panduan Lengkap Instalasi & Deployment Eegyo SIMS

Dokumen ini memuat panduan lengkap mengenai cara melakukan instalasi, konfigurasi server, serta integrasi layanan pihak ketiga (**Laravel Reverb**, **Cloudflare R2**, dan **Firebase Cloud Messaging / FCM**) untuk aplikasi **Eegyo SIMS**.

---

## 📑 Daftar Isi
1. [Persyaratan Sistem (System Requirements)](#1-persyaratan-sistem-system-requirements)
2. [Deployment di VPS (Ubuntu / Debian + Nginx) - Rekomendasi](#2-deployment-di-vps-ubuntu--debian--nginx---rekomendasi)
   - [A. Upload & Persiapan Direktori](#a-upload--persiapan-direktori)
   - [B. Konfigurasi Web Server (Nginx Virtual Host)](#b-konfigurasi-web-server-nginx-virtual-host)
   - [B.2. Konfigurasi OpenLiteSpeed (OLS / CyberPanel / aaPanel)](#b2-jika-menggunakan-openlitespeed-ols--cyberpanel--aapanel)
   - [C. Menjalankan Wizard Instalasi](#c-menjalankan-wizard-instalasi)
   - [D. Konfigurasi Background Worker (Supervisor)](#d-konfigurasi-background-worker-supervisor)
   - [E. Konfigurasi Cron Job (Scheduler)](#e-konfigurasi-cron-job-scheduler)
3. [Deployment di Shared Hosting (cPanel / DirectAdmin)](#3-deployment-di-shared-hosting-cpanel--directadmin)
   - [A. Pengaturan Struktur File yang Aman](#a-pengaturan-struktur-file-yang-aman)
   - [B. Setup Database & Wizard](#b-setup-database--wizard)
   - [C. Pembuatan Symlink Storage](#c-pembuatan-symlink-storage)
   - [D. Cron Job di cPanel](#d-cron-job-di-cpanel)
   - [E. Catatan Khusus Reverb di Shared Hosting](#e-catatan-khusus-reverb-di-shared-hosting)
4. [Konfigurasi Cloudflare R2 (Object Storage S3-Compatible)](#4-konfigurasi-cloudflare-r2-object-storage-s3-compatible)
   - [A. Mengapa Cloudflare R2?](#a-mengapa-cloudflare-r2)
   - [B. Langkah Setup di Cloudflare Dashboard](#b-langkah-setup-di-cloudflare-dashboard)
   - [C. Konfigurasi di Eegyo SIMS](#c-konfigurasi-di-eegyosims)
5. [Konfigurasi Firebase Cloud Messaging (FCM)](#5-konfigurasi-firebase-cloud-messaging-fcm)
   - [A. Fungsi FCM di Eegyo SIMS](#a-fungsi-fcm-di-eegyosims)
   - [B. Langkah Setup di Firebase Console](#b-langkah-setup-di-firebase-console)
   - [C. Upload Service Account JSON & Pengaturan di Aplikasi](#c-upload-service-account-json--pengaturan-di-aplikasi)
6. [Troubleshooting & Solusi Masalah Populer](#6-troubleshooting--solusi-masalah-populer)

---

## 1. Persyaratan Sistem (System Requirements)

* **PHP:** Versi `>= 8.3` (Sesuai `composer.json`)
* **Ekstensi PHP Wajib:**
  * `php-bcmath`
  * `php-ctype`
  * `php-curl`
  * `php-dom` / `php-xml`
  * `php-fileinfo`
  * `php-gd` atau `php-imagick` (untuk resize gambar & banner)
  * `php-intl`
  * `php-json`
  * `php-mbstring`
  * `php-openssl`
  * `php-pdo_mysql`
  * `php-tokenizer`
  * `php-zip`
  * `php-pcntl` (wajib di VPS untuk menjalankan process worker seperti Supervisor/Reverb)
* **Database:** MySQL `>= 8.0` atau MariaDB `>= 10.4`
* **Web Server:** Nginx (Sangat direkomendasikan untuk Reverse Proxy WebSocket) atau Apache
* **Opsional (Direkomendasikan untuk traffic tinggi):** Redis Server

---

## 2. Deployment di VPS (Ubuntu / Debian + Nginx) - Rekomendasi

### A. Upload & Persiapan Direktori

1. Tempatkan source code aplikasi di direktori server, misalnya:
   ```bash
   /var/www/eegyosims
   ```
2. Berikan hak akses kepemilikan folder ke user web server (`www-data`):
   ```bash
   sudo chown -R www-data:www-data /var/www/eegyosims
   sudo chmod -R 775 /var/www/eegyosims/storage /var/www/eegyosims/bootstrap/cache
   ```
3. Salin file `.env`:
   ```bash
   cp .env.example .env
   php artisan key:generate
   ```
4. Buat symlink storage publik:
   ```bash
   php artisan storage:link
   ```

> [!NOTE]
> Semua aset frontend JavaScript & CSS sudah ter-bundle di folder `public/`. **Anda tidak perlu menginstall Node.js ataupun menjalankan `npm run build` di server VPS.**

---

### B. Konfigurasi Web Server (Nginx Virtual Host)

Buat file konfigurasi Nginx baru, misalnya di `/etc/nginx/sites-available/eegyosims`:

```nginx
server {
    listen 80;
    server_name sims.sekolah.sch.id;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sims.sekolah.sch.id;

    root /var/www/eegyosims/public;
    index index.php index.html;

    # SSL Certificates (Let's Encrypt / Certbot)
    ssl_certificate /etc/letsencrypt/live/sims.sekolah.sch.id/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sims.sekolah.sch.id/privkey.pem;

    client_max_body_size 50M;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # REVERSE PROXY LARAVEL REVERB (WEBSOCKET REALTIME)
    location /app {
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";

        proxy_pass http://127.0.0.1:8080;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Aktifkan konfigurasi dan reload Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/eegyosims /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### B.2. Jika Menggunakan OpenLiteSpeed (OLS / CyberPanel / aaPanel)

Konfigurasi Nginx **tidak bisa langsung dipakai di OLS** karena OLS menggunakan struktur file konfigurasi dan engine yang berbeda. Namun, OLS **memiliki fitur native WebSocket Proxy** yang sangat mudah dikonfigurasi.

#### Cara 1: Mengedit File Virtual Host OLS (CyberPanel / Manual)
Buka konfigurasi Virtual Host domain Anda (misal di CyberPanel: **Websites > Modify Website / Vhost Conf**, atau di `/usr/local/lsws/conf/vhosts/sims.sekolah.sch.id/vhost.conf`):

Tambahkan blok `websocket /app` di dalam konfigurasi vhost:
```conf
websocket /app {
    address                 127.0.0.1:8080
}
```

Pastikan juga **Document Root** sudah mengarah ke folder `public`:
```conf
docRoot                   $VH_ROOT/public
```

#### Cara 2: Lewat OLS WebAdmin Console (Port 7080)
1. Buka OLS WebAdmin (`https://IP-SERVER:7080`).
2. Masuk ke menu **Virtual Hosts** > Pilih Virtual Host domain aplikasi Anda.
3. Buka tab **Web Socket Proxy**.
4. Klik tombol **+ Add**:
   * **URI:** `/app`
   * **Address:** `127.0.0.1:8080`
5. Simpan (ikon disket), lalu klik **Graceful Restart** OpenLiteSpeed (ikon panah hijau di pojok kanan atas).

> [!NOTE]
> Setelah WebSocket Proxy di OLS aktif, koneksi `wss://sims.sekolah.sch.id/app/...` akan secara otomatis di-forward oleh OLS ke background process Laravel Reverb di `127.0.0.1:8080`.

---

### C. Menjalankan Wizard Instalasi

Buka browser dan akses domain Anda:
```text
https://sims.sekolah.sch.id
```
Aplikasi secara otomatis mendeteksi status instalasi dan mengarahkan Anda ke **Wizard Instalasi**:
1. **Step 1 - Sistem**: Konfigurasi koneksi Database MySQL dan APP_URL.
2. **Step 2 - Profil Sekolah**: Nama sekolah, jenjang, NPSN, logo, dan alamat.
3. **Step 3 - Pengaturan Aplikasi**: Nama branding aplikasi dan kontak.
4. **Step 4 - Akun Administrator**: Pembuatan akun Super Admin pertama.

Setelah selesai, database otomatis dimigrasi beserta seeder master (saldo sistem, angkatan default, roles & permissions).

---

### D. Konfigurasi Background Worker (Supervisor)

Supervisor digunakan untuk menjaga agar **Laravel Reverb (WebSocket)** dan **Queue Worker** tetap berjalan terus di background secara otomatis.

1. Install supervisor:
   ```bash
   sudo apt update && sudo apt install supervisor -y
   ```
2. Buat file `/etc/supervisor/conf.d/eegyosims.conf`:
   ```ini
   [program:eegyosims-reverb]
   process_name=%(program_name)s
   command=php /var/www/eegyosims/artisan reverb:start --host=127.0.0.1 --port=8080
   autostart=true
   autorestart=true
   stopasgroup=true
   killasgroup=true
   user=www-data
   numprocs=1
   redirect_stderr=true
   stdout_logfile=/var/www/eegyosims/storage/logs/reverb.log
   stopwaitsecs=3600

   [program:eegyosims-worker]
   process_name=%(program_name)s_%(process_num)02d
   command=php /var/www/eegyosims/artisan queue:work --sleep=3 --tries=3 --max-time=3600
   autostart=true
   autorestart=true
   stopasgroup=true
   killasgroup=true
   user=www-data
   numprocs=2
   redirect_stderr=true
   stdout_logfile=/var/www/eegyosims/storage/logs/worker.log
   stopwaitsecs=3600
   ```
3. Muat ulang dan jalankan proses:
   ```bash
   sudo supervisorctl reread
   sudo supervisorctl update
   sudo supervisorctl start all
   ```
4. Cek status worker kapan saja:
   ```bash
   sudo supervisorctl status
   ```

---

### E. Konfigurasi Cron Job (Scheduler)

Laravel membutuhkan satu perintah cron job setiap menit untuk memproses penjadwalan otomatis (seperti penagihan SPP, sinkronisasi notifikasi, dan auto-rekap).

Buka crontab user www-data:
```bash
sudo crontab -u www-data -e
```
Tambahkan baris berikut di bagian paling bawah:
```cron
* * * * * cd /var/www/eegyosims && php artisan schedule:run >> /dev/null 2>&1
```

---

## 3. Deployment di Shared Hosting (cPanel / DirectAdmin)

Shared hosting memiliki karakteristik lingkungan terbatas (tidak memiliki akses `root` atau `supervisor`). Berikut cara melakukan instalasi dengan benar dan aman:

### A. Pengaturan Struktur File yang Aman

**Jangan menaruh seluruh isi project Laravel langsung di dalam `public_html`.** Lakukan pemisahan direktori:

1. Buat folder baru di luar `public_html`, misalnya `/home/username/eegyosims/`.
2. Upload semua file projek ke `/home/username/eegyosims/` **kecuali** isi dari folder `public/`.
3. Pindahkan seluruh isi dari folder `public/` ke dalam `public_html/`.
4. Buka file `/home/username/public_html/index.php`, lalu sesuaikan path autoload-nya:
   ```php
   // Ganti baris ini:
   require __DIR__.'/../vendor/autoload.php';
   $app = require_once __DIR__.'/../bootstrap/app.php';

   // Menjadi path absolut atau relative ke folder core:
   require __DIR__.'/../eegyosims/vendor/autoload.php';
   $app = require_once __DIR__.'/../eegyosims/bootstrap/app.php';
   ```

---

### B. Setup Database & Wizard

1. Buka cPanel > **MySQL Database Wizard**.
2. Buat database baru (misal: `user_eegyosims`) dan user database beserta password.
3. Berikan permission **ALL PRIVILEGES** pada user tersebut.
4. Akses domain sekolah Anda di browser, lalu ikuti langkah **Wizard Instalasi** web.

---

### C. Pembuatan Symlink Storage

Agar file gambar, kop surat, dan bukti pembayaran yang diupload lokal bisa diakses publik:
* **Jika cPanel memiliki fitur Terminal:**
  ```bash
  cd /home/username/eegyosims
  php artisan storage:link
  ```
* **Jika tidak ada Terminal:**
  Buat file script sementara di `public_html/symlink.php`:
  ```php
  <?php
  symlink('/home/username/eegyosims/storage/app/public', '/home/username/public_html/storage');
  echo "Symlink berhasil dibuat!";
  ```
  Akses `https://domainanda.com/symlink.php` sekali di browser, kemudian hapus file `symlink.php` tersebut.

---

### D. Cron Job di cPanel

1. Buka cPanel > menu **Cron Jobs**.
2. Pada bagian Common Settings, pilih **Once Per Minute (* * * * *)**.
3. Di kolom Command, masukkan:
   ```bash
   /usr/local/bin/php /home/username/eegyosims/artisan schedule:run >> /dev/null 2>&1
   ```
   *(Catatan: Path `/usr/local/bin/php` bisa disesuaikan dengan versi PHP cPanel Anda, misalnya `/usr/local/bin/ea-php83`).*

---

### E. Catatan Khusus Reverb di Shared Hosting

> [!WARNING]
> Layanan shared hosting standar umumnya **tidak mengizinkan** proses background terus menerus (*persistent background daemon*) dan memblokir port kustom / Nginx reverse proxy internal.

**Opsi Solusi untuk Shared Hosting:**
1. **Opsi Rekomendasi (VPS):** Untuk fitur realtime WebSocket (Laravel Reverb), sangat disarankan menggunakan VPS (seperti DigitalOcean, Linode, AWS Lightsail, IDCloudHost, BiznetGio) yang mendukung supervisor.
2. **Opsi Cloud Pusher:** Jika tetap di shared hosting, Anda dapat beralih menggunakan akun gratis **Pusher.com** dengan mengubah konfigurasi di menu **Admin > Sistem**:
   * Broadcast Driver: `pusher`
   * Masukkan App Key, Secret, dan Cluster dari Pusher.com.
3. **Mode Polling Otomatis:** Jika Reverb tidak aktif, aplikasi tetap berfungsi normal 100%. User hanya perlu me-refresh halaman atau memanfaatkan fitur notifikasi email & FCM.

---

## 4. Konfigurasi Cloudflare R2 (Object Storage S3-Compatible)

### A. Mengapa Cloudflare R2?
* **Zero Egress Fee:** Bebas biaya bandwidth unduhan data (berbeda dengan AWS S3 yang mengenakan tarif egress).
* **Hemat Disk Server:** Foto siswa, dokumen pendaftaran PPDB, struk kantin, dan materi pelajaran tersimpan di cloud storage yang aman, cepat, dan tidak memenuhi kapasitas SSD server.
* **Kompatibel S3:** Menggunakan protokol standar Amazon S3 SDK.

---

### B. Langkah Setup di Cloudflare Dashboard

1. Login ke [Cloudflare Dashboard](https://dash.cloudflare.com/).
2. Di menu sidebar kiri, pilih **R2 Object Storage**.
3. Klik **Create bucket**:
   * Beri nama bucket, misalnya: `eegyosims`
   * Location: Pilih **Automatic** (atau region Asia Pasifik)
   * Klik **Create Bucket**.
4. Hubungkan Domain Publik (Custom Domain):
   * Masuk ke tab **Settings** pada bucket yang baru dibuat.
   * Pada bagian **Public Access**, klik **Connect Domain**.
   * Masukkan subdomain khusus, misalnya `cdn.sekolah.sch.id` atau gunakan default sub-domain Cloudflare `*.r2.dev`.
5. Buat API Token (Credentials):
   * Kembali ke menu utama R2 > Klik **Manage R2 API Tokens** di sebelah kanan.
   * Klik **Create API Token**.
   * Berikan nama token (misal: `token-eegyosims`), atur permissions ke **Object Read & Write**.
   * Klik **Create API Token**.
   * Catat nilai yang muncul:
     * **Access Key ID**
     * **Secret Access Key**
     * **Endpoint URL** (format: `https://<account-id>.r2.cloudflarestorage.com`)

---

### C. Konfigurasi di Eegyo SIMS

Ada 2 cara konfigurasi yang dapat Anda pilih:

#### Cara 1: Lewat Menu Pengaturan Sistem di Dashboard Admin (Sangat Mudah)
1. Login sebagai Admin di aplikasi Eegyo SIMS.
2. Buka menu **Sistem > Pengaturan Sistem**.
3. Di bagian **Penyimpanan Berkas (Filesystem)**:
   * Pilih Driver: `s3`
   * Masukkan **AWS / R2 Access Key ID**
   * Masukkan **AWS / R2 Secret Access Key**
   * Default Region: `auto`
   * Bucket Name: `eegyosims`
   * Endpoint: `https://<account-id>.r2.cloudflarestorage.com`
   * URL Publik (CDN): `https://cdn.sekolah.sch.id`
   * Path Style Endpoint: `true`
4. Klik tombol **Simpan Pengaturan**.

#### Cara 2: Lewat File `.env`
Buka file `.env` di server dan sesuaikan variabel berikut:
```env
FILESYSTEM_DISK=s3

AWS_ACCESS_KEY_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_DEFAULT_REGION=auto
AWS_BUCKET=eegyosims
AWS_USE_PATH_STYLE_ENDPOINT=true
AWS_ENDPOINT=https://<account-id>.r2.cloudflarestorage.com
AWS_URL=https://cdn.sekolah.sch.id
```

---

## 5. Konfigurasi Firebase Cloud Messaging (FCM)

### A. Fungsi FCM di Eegyo SIMS
FCM digunakan untuk mengirim **Web Push Notification** ke perangkat HP/Laptop pengguna (Guru, Wali Murid, Siswa, Admin) saat browser tertutup atau aplikasi terpasang sebagai **PWA (Progressive Web App)**.

Contoh notifikasi: Tagihan SPP baru, pengumuman sekolah, rekap presensi harian, dan notifikasi transaksi kantin/koperasi.

---

### B. Langkah Setup di Firebase Console

1. Buka [Firebase Console](https://console.firebase.google.com/) dan buat project baru (misal: `Eegyo SIMS`).
2. **Daftarkan Web App:**
   * Di dashboard project, klik ikon Web (`</>`).
   * Beri nama aplikasi (misal: `Eegyo SIMS Web`), lalu klik **Register app**.
   * Anda akan melihat konfigurasi SDK berisi:
     * `apiKey`
     * `authDomain`
     * `projectId`
     * `messagingSenderId`
     * `appId`
3. **Generate VAPID Key (Web Push Certificate):**
   * Masuk ke **Project Settings** (ikon gerigi) > Tab **Cloud Messaging**.
   * Gulir ke bawah ke bagian **Web configuration** > **Web Push certificates**.
   * Klik tombol **Generate key pair**.
   * Salin string **Key pair** yang muncul (ini adalah `FIREBASE_VAPID_KEY`).
4. **Unduh Service Account JSON (Google OAuth2 v1):**
   * Di Project Settings, masuk ke tab **Service accounts**.
   * Pastikan opsi **Firebase Admin SDK** terpilih.
   * Klik tombol **Generate new private key** > Konfirmasi.
   * Browser akan mengunduh sebuah file berformat `.json` (misal: `eegyosims-firebase-adminsdk-xxxxx.json`).

---

### C. Upload Service Account JSON & Pengaturan di Aplikasi

Eegyo SIMS telah dilengkapi antarmuka bawaan untuk mengupload kredensial Firebase tanpa perlu coding:

1. Buka aplikasi Eegyo SIMS > Login sebagai **Admin**.
2. Masuk ke menu **Sistem > Pengaturan Sistem** > Pilih Tab **Firebase / FCM**.
3. Isi parameter konfigurasi web:
   * **Firebase API Key:** Isikan nilai `apiKey`
   * **Firebase Project ID:** Isikan nilai `projectId`
   * **Firebase Auth Domain:** Isikan nilai `authDomain`
   * **Firebase Messaging Sender ID:** Isikan nilai `messagingSenderId`
   * **Firebase App ID:** Isikan nilai `appId`
   * **Firebase VAPID Key:** Isikan Key pair dari Web Push Certificate
4. **Upload File Kredensial:**
   * Pada kolom **Upload Service Account (JSON)**, pilih file `.json` yang Anda unduh dari Firebase Console.
   * Sistem akan otomatis memvalidasi struktur JSON dan menempatkannya secara aman di `storage/app/firebase/firebase_credentials.json`.
5. Klik **Simpan Pengaturan**.
6. Gunakan tombol **Uji Coba Otentikasi FCM** untuk memverifikasi bahwa server berhasil terkoneksi ke Google Firebase API via OAuth2.

---

## 6. Troubleshooting & Solusi Masalah Populer

### 1. Pesan Error: *WebSocket connection to 'wss://.../app/...' failed*
* **Penyebab:** Konfigurasi Nginx Reverse Proxy belum ditambahkan atau port 8080 terblokir firewall.
* **Solusi:**
  1. Pastikan blok `location /app { ... }` sudah ada di Nginx server block HTTPS (port 443).
  2. Pastikan Reverb sedang berjalan di supervisor: `sudo supervisorctl status eegyosims-reverb`.
  3. Pastikan `REVERB_SCHEME=https` dan `REVERB_PORT=443` di `.env`.

---

### 2. File Upload / Logo / Avatar Gambar Broken (Tidak Muncul)
* **Jika menggunakan Local Storage:**
  Jalankan perintah symlink:
  ```bash
  php artisan storage:link
  ```
* **Jika menggunakan Cloudflare R2:**
  Pastikan URL publik (`AWS_URL`) sudah bisa diakses langsung via browser dan bucket memiliki public read permission.

---

### 3. Error Permission Denied (500 Internal Server Error)
* Pastikan direktori `storage` dan `bootstrap/cache` memiliki permission tulis bagi web server:
  ```bash
  sudo chown -R www-data:www-data /var/www/eegyosims
  sudo chmod -R 775 /var/www/eegyosims/storage /var/www/eegyosims/bootstrap/cache
  ```

---

### 4. Notifikasi Push PWA Tidak Masuk ke HP / Browser Siswa
1. Pastikan website diakses menggunakan protokol **HTTPS (SSL)**. Service Worker dan Web Push Notification di browser modern **wajib** menggunakan HTTPS.
2. Pastikan pengguna telah mengklik tombol **"Izinkan Notifikasi"** pada prompt izin browser.
3. Pastikan status kredensial Firebase di menu **Admin > Sistem > Firebase** berstatus **Valid / Hijau**.

---

*Eegyo SIMS - School Integrated Management System*
