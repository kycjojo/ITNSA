# Panduan Teknis Kisi-Kisi Atswa NeSA Skill Competition

Dokumen ini disusun sebagai panduan teknis resmi dan komprehensif untuk persiapan menghadapi **Atswa NeSA (Network and System Administration) Skill Competition** yang diselenggarakan oleh CV. Atswa Indonesia ([nesa.atswa.co.id](https://nesa.atswa.co.id)).

Materi dalam panduan ini mencakup analisis fungsi perlengkapan peserta, penjabaran konsep dan fungsi materi uji dalam arsitektur sistem, serta prediksi skenario pengujian oleh tim juri.

---

## Daftar Isi
1. [Analisis Kebutuhan Perlengkapan Peserta](#1-analisis-kebutuhan-perlengkapan-peserta)
2. [Penjelasan Materi Uji Kompetensi](#2-penjelasan-materi-uji-kompetensi)
   - [2.1 Linux Dasar](#21-linux-dasar)
   - [2.2 Ubuntu 24.04 LTS](#22-ubuntu-2404-lts)
   - [2.3 Manajemen Domain dan DNS](#23-manajemen-domain-dan-dns)
   - [2.4 Git Dasar](#24-git-dasar)
   - [2.5 Web Server](#25-web-server)
   - [2.6 Reverse Proxy](#26-reverse-proxy)
   - [2.7 SSL / TLS Let's Encrypt](#27-ssl--tls-lets-encrypt)
   - [2.8 Proxmox VE (LXC, Firewall, User Management, Backup, VLAN)](#28-proxmox-ve)
3. [Arsitektur Sistem dan Integrasi Layanan](#3-arsitektur-sistem-dan-integrasi-layanan)
4. [Prediksi Skenario Soal dan Prosedur Penilaian](#4-prediksi-skenario-soal-dan-prosedur-penilaian)
5. [Prosedur Operasional Standar (SOP) Pengerjaan](#5-prosedur-operasional-standar-sop-pengerjaan)

---

## 1. Analisis Kebutuhan Perlengkapan Peserta

Panitia NeSA Atswa menetapkan persyaratan perangkat keras dan perangkat lunak tertentu yang memiliki fungsi operasional langsung terhadap skema penilaian lomba:

### A. Laptop (Minimal 2 Core, 2 GB RAM, 128 GB Storage, Ethernet, Wireless)
- **Fungsi Operasional**: Laptop berfungsi sebagai *Management Workstation* (konsol kerja peserta) untuk mengakses Proxmox VE Web GUI, terminal SSH, pengelola DNS, dan menjalankan aplikasi perekam layar.
- **Dua Antarmuka Jaringan (Ethernet dan Wireless)**:
  - **Antarmuka Ethernet**: Digunakan untuk koneksi berkecepatan tinggi dan stabil menuju jaringan lokal pengujian atau server Proxmox.
  - **Antarmuka Wireless**: Digunakan untuk koneksi internet eksternal yang diperlukan dalam pengelolaan DNS publik dan validasi sertifikat SSL secara langsung (*live*).
- **Rekomendasi Manajemen Sumber Daya**: Mengingat batas spesifikasi minimum prosesor 2 core dan RAM 2 GB, peserta disarankan membatasi aplikasi yang berjalan di latar belakang guna mencegah beban komputasi berlebih (*overloading*).

### B. OBS Studio 32.1.2
- **Tujuan Pemasangan**:
  1. **Pengawasan Integritas Kompetisi (*Proctoring*)**: Merekam seluruh aktivitas layar monitor peserta secara berkelanjutan sejak awal pengerjaan hingga waktu pengujian berakhir guna memastikan tidak terdapat tindakan yang melanggar ketentuan.
  2. **Validasi Bukti Masa Sanggah**: Apabila terjadi perbedaan hasil antara pengujian otomatis oleh juri dan konfigurasi peserta, rekaman video OBS menjadi alat bukti resmi untuk proses peninjauan kembali.
- **Rekomendasi Konfigurasi Video Encoding untuk Efisiensi Sistem**:
  - *Base & Output Resolution*: 1280 x 720 (720p).
  - *Framerate*: 30 FPS.
  - *Bitrate*: 1500 – 2000 kbps (menghasilkan keterbacaan teks terminal yang jelas dengan ukuran file yang terkontrol).
  - *Encoder*: Gunakan akselerasi perangkat keras (Intel QuickSync / AMD AMF / Nvidia NVENC) jika tersedia, atau x264 dengan profil CPU *ultrafast*.

### C. Domain `nama_peserta.my.id` (Berlangganan Mandiri)
- **Fungsi Utama**: NeSA Atswa mengadopsi standar industri riil dengan mewajibkan penggunaan *Top-Level Domain* (TLD) publik aktif di internet, bukan sekadar domain simulasi lokal (`.lab` atau `.local`).
- **Implikasi Teknis**:
  - Peserta harus mampu mengonfigurasi DNS Record (tipe A, CNAME) pada dashboard DNS Management publik (seperti Cloudflare atau portal registrar).
  - Proses penerbitan sertifikat SSL Let's Encrypt dilakukan secara langsung (*live verification*) melalui jaringan internet menggunakan tantangan validasi ACME HTTP-01.
  - Tim juri dapat memverifikasi ketersediaan layanan web secara langsung dari internet melalui URL `https://subdomain.nama_peserta.my.id`.

---

## 2. Penjelasan Materi Uji Kompetensi

### 2.1 Linux Dasar
- **Definisi dan Konsep**: Penguasaan antarmuka baris perintah (*Command Line Interface* / CLI) Linux, struktur direktori standar (FHS), serta pengelolaan hak akses file dan proses sistem.
- **Fungsi dalam Lomba**:
  - Pengoperasian sistem tanpa antarmuka grafis (headless).
  - Pengaturan kepemilikan file (*file ownership*) menggunakan `chown` dan izin akses (*permissions*) menggunakan `chmod` (misalnya izin direktori web `755` dan izin file konfigurasi sensitif `600`).
  - Pengelolaan service menggunakan `systemd` (`systemctl status`, `start`, `enable`, `reload`).
- **Parameter Penilaian**: Persistensi service setelah komputer di-reboot. Perintah `systemctl enable` wajib dijalankan agar layanan aktif otomatis saat sistem menyala.

### 2.2 Ubuntu 24.04 LTS
- **Definisi dan Konsep**: Distribusi Linux enterprise modern yang menjadi sistem operasi dasar untuk seluruh guest container (LXC).
- **Fungsi dalam Lomba**:
  - **Netplan v2**: Konfigurasi antarmuka jaringan statis melalui file deklaratif YAML di `/etc/netplan/` dengan renderer `systemd-networkd`.
  - **Manajemen Port Resolver**: Penanganan konflik port 53 antara *stub resolver* internal (`systemd-resolved`) dengan DNS server BIND9.
- **Parameter Penilaian**: Keberhasilan konektivitas jaringan pasca-reboot tanpa adanya kesalahan sintaks indentasi pada file Netplan.

### 2.3 Manajemen Domain dan DNS
- **Definisi dan Konsep**: Sistem penerjemahan alamat logika berbentuk *Fully Qualified Domain Name* (FQDN) menjadi alamat Internet Protocol (IP), serta sebaliknya.
- **Fungsi dalam Lomba**:
  - **DNS Publik**: Pemetaan record A pada domain `nama_peserta.my.id` menuju alamat IP publik server atau antarmuka DMZ.
  - **DNS Lokal (BIND9)**: Penyediaan resolusi nama internal untuk komunikasi antar-node di dalam jaringan laboratorium, mencakup konfigurasi *Forward Zone*, *Reverse Zone*, dan *Forwarders*.
- **Parameter Penilaian**: Kecepatan dan ketepatan resolusi nama domain saat diuji menggunakan perintah `dig` atau `nslookup`, serta keabsahan sintaks zona yang diverifikasi melalui `named-checkzone`.

### 2.4 Git Dasar
- **Definisi dan Konsep**: Sistem kontrol versi terdistribusi untuk melacak perubahan kode sumber dan mengelola distribusi aplikasi.
- **Fungsi dalam Lomba**:
  - Penarikan (*cloning*) kode sumber website lomba dari repositori yang disediakan panitia.
  - Berpindah ke cabang (*branch*) rilis tertentu (misalnya cabang `main` atau `production`) menggunakan `git checkout`.
  - Otomatisasi pembaruan kode aplikasi ke direktori web melalui *git hook* (`post-receive`).
- **Parameter Penilaian**: Kelengkapan kode pada direktori tujuan (`/var/www/`), kesesuaian branch kerja, dan pencegahan terbukanya folder `.git` ke akses publik demi standar keamanan.

### 2.5 Web Server
- **Definisi dan Konsep**: Layanan perangkat lunak (umumnya Nginx atau Apache HTTP Server) yang bertugas memproses permintaan protokol HTTP/HTTPS dan mengirimkan respons berupa halaman web atau data API.
- **Fungsi dalam Lomba**:
  - Konfigurasi Virtual Host (*Server Block*) untuk memetakan domain ke direktori dokumen tertentu (*Document Root*).
  - Integrasi dengan pemroses skrip dinamis (PHP-FPM) melalui Unix Domain Socket atau TCP Socket.
  - Kustomisasi halaman kesalahan sistem (seperti 404 Not Found dan 502 Bad Gateway) serta pengamanan direktori (*autoindex off*).
- **Parameter Penilaian**: Ketersediaan respons HTTP 200 OK saat diakses melalui nama domain yang ditentukan, eksekusi skrip PHP yang valid, dan kesesuaian konfigurasi hak akses user `www-data`.

### 2.6 Reverse Proxy
- **Definisi dan Konsep**: Server perantara yang menerima permintaan dari klien luar di zona demiliterisasi (DMZ), kemudian meneruskannya ke satu atau beberapa backend server di jaringan privat internal.
- **Fungsi dalam Lomba**:
  - **Isolasi Arsitektur Multi-Tier**: Melindungi backend server dan database agar tidak terekspos langsung ke internet.
  - **SSL Termination**: Memusatkan proses dekripsi HTTPS di layer reverse proxy sehingga backend server hanya perlu menangani traffic HTTP biasa.
  - **Header Passing**: Meneruskan informasi identitas klien asli ke backend server melalui penyisipan header HTTP (`X-Real-IP`, `X-Forwarded-For`, dan `X-Forwarded-Proto`).
- **Parameter Penilaian**: Keberhasilan akses aplikasi internal melalui reverse proxy, pencatatan IP klien asli pada log backend, serta ketiadaan galat *Bad Gateway* (502).

### 2.7 SSL / TLS Let's Encrypt
- **Definisi dan Konsep**: Implementasi protokol kriptografi TLS untuk mengamankan pertukaran data melalui HTTPS menggunakan sertifikat digital yang diterbitkan oleh Certificate Authority (CA) Let's Encrypt.
- **Fungsi dalam Lomba**:
  - Pemasangan dan eksekusi utilitas **Certbot** untuk menerbitkan sertifikat resmi pada domain `nama_peserta.my.id`.
  - Penerapan pengalihan otomatis (*HTTP to HTTPS redirect*) dengan kode status 301 Permanent Redirect.
  - Penerapan parameter keamanan modern, seperti TLSv1.2/TLSv1.3 dan header *Strict-Transport-Security* (HSTS).
- **Parameter Penilaian**: Validitas sertifikat digital tanpa adanya peringatan ketidakamanan (*untrusted certificate warning*) pada browser juri dan keberhasilan enkripsi pada port 443.

### 2.8 Proxmox VE
Platform virtualisasi tipe-1 (*bare-metal hypervisor*) berbasis kernel Linux Debian yang mengelola seluruh infrastruktur virtual kompetisi:

1. **LXC (Linux Containers)**:
   - *Konsep*: Virtualisasi tingkat sistem operasi yang memanfaatkan fitur *kernel namespaces* dan *cgroups*.
   - *Fungsi*: Penyediaan node server independen untuk DNS, Reverse Proxy, dan Web Server secara cepat dan efisien terhadap konsumsi RAM.
2. **VLAN (Virtual Local Area Network)**:
   - *Konsep*: Segmentasi jaringan logis berbasis standar IEEE 802.1Q pada Linux Bridge (`vmbr0`).
   - *Fungsi*: Memisahkan zona jaringan publik/DMZ (VLAN 10) dengan zona aplikasi/internal (VLAN 20).
3. **Firewall**:
   - *Konsep*: Sistem penyaringan paket jaringan bertingkat (Datacenter, Node, dan VM/CT) berbasis Netfilter.
   - *Fungsi*: Penerapan kebijakan *default drop*, membuka hanya port layanan publik yang diizinkan (port 80 dan 443), serta membatasi akses administratif (port 22 dan 8006).
4. **User Management & RBAC**:
   - *Konsep*: Pengaturan identitas pada realm autentikasi (`pve` / `pam`) dan pembagian hak akses berbasis peran (*Role-Based Access Control*).
   - *Fungsi*: Pembuatan akun pengawas bagi tim penilai (misalnya user `juri@pve` dengan peran `PVEAuditor` pada path `/vms`).
5. **Backup (VZDump)**:
   - *Konsep*: Utilitas pembuatan arsip snapshot dari container atau mesin virtual secara terjadwal.
   - *Fungsi*: Penyusunan skema cadangan data otomatis dan kebijakan retensi pembersihan (*prune retention*) untuk menjamin pemulihan sistem (*disaster recovery*).

---

## 3. Arsitektur Sistem dan Integrasi Layanan

Bagan di bawah ini menggambarkan alur komunikasi terpadu antar-komponen saat pengujian berlangsung:

```
[Laptop Juri / Penguji]
       │
       ▼ (1. Resolusi DNS Publik: https://app.nama_peserta.my.id)
[DNS Registrar / Cloudflare]
       │
       ▼ (2. Request HTTPS Port 443)
[PROXMOX VE HYPERVISOR]
   ├── [FIREWALL]: Verifikasi port 443 -> Diizinkan
   │
   ├── [VLAN 10 - DMZ]:
   │     LXC 101: Nginx Reverse Proxy & SSL Let's Encrypt
   │     - Menangani dekripsi HTTPS
   │     - Menyisipkan header X-Forwarded-For
   │     - Mengarahkan traffic ke backend via jaringan internal
   │
   └── [VLAN 20 - INTERNAL]:
         LXC 102: Web Server (Nginx + PHP-FPM)
         - Kode sumber ditarik dari Git
         - Melayani respons aplikasi web
```

---

## 4. Prediksi Skenario Soal dan Prosedur Penilaian

### Deskripsi Tugas Terpadu
Peserta diinstruksikan membangun infrastruktur layanan enterprise yang aman dan terisolasi dengan tahapan sebagai berikut:

1. **Konfigurasi Lapisan Virtualisasi & Jaringan**:
   - Menyiapkan bridge `vmbr0` pada Proxmox dengan opsi `bridge-vlan-aware yes`.
   - Membuat LXC 101 (Ubuntu 24.04 LTS) pada **VLAN 10** sebagai Reverse Proxy.
   - Membuat LXC 102 (Ubuntu 24.04 LTS) pada **VLAN 20** sebagai Backend Web Server.
2. **Penyebaran Kode Aplikasi via Git**:
   - Menarik kode sumber website resmi dari repositori panitia ke dalam LXC 102 pada direktori `/var/www/`.
   - Mengonfigurasi kepemilikan file agar dapat dieksekusi secara benar oleh proses web server.
3. **Konfigurasi Reverse Proxy & Domain Publik**:
   - Mengarahkan subdomain `app.nama_peserta.my.id` pada panel DNS publik ke IP publik pengujian.
   - Mengonfigurasi Nginx di LXC 101 untuk meneruskan permintaan ke IP privat LXC 102 di VLAN 20.
4. **Penerapan Keamanan SSL Let's Encrypt**:
   - Menjalankan Certbot pada LXC 101 untuk mendapatkan sertifikat digital resmi Let's Encrypt.
   - Mengaktifkan pengalihan otomatis seluruh traffic HTTP (port 80) menuju HTTPS (port 443).
5. **Pengamanan Firewall & Manajemen Pengguna**:
   - Menerapkan kebijakan firewall incoming DROP, dengan pengecualian hanya pada port layanan publik dan manajemen.
   - Membuat akun penilai `juri@pve` dengan hak akses audit (`PVEAuditor`).
   - Mengonfigurasi jadwal pencadangan container menggunakan utilitas `vzdump`.
6. **Uji Persistensi Sistem**:
   - Seluruh server fisik dan container akan di-reboot sebelum penilaian akhir dilakukan. Seluruh layanan wajib aktif kembali secara otomatis.

---

## 5. Prosedur Operasional Standar (SOP) Pengerjaan

Guna memaksimalkan perolehan nilai dan meminimalkan galat teknis, ikuti urutan kerja sistematis berikut:

1. **Tahap 1: Pengujian Awal & Perekaman OBS**:
   - Pastikan OBS Studio telah mulai merekam layar penuh sebelum membuka lembar soal atau terminal.
2. **Tahap 2: Infrastruktur Dasar Proxmox**:
   - Konfigurasi VLAN Bridge $\rightarrow$ Pembuatan LXC Container $\rightarrow$ Pengaturan IP statis Netplan.
3. **Tahap 3: Resolusi DNS Publik**:
   - Buat A Record subdomain di DNS Management publik $\rightarrow$ Verifikasi menggunakan perintah `ping` atau `dig`.
4. **Tahap 4: Layanan Web & Git**:
   - Deploy backend web di LXC internal $\rightarrow$ Uji respon HTTP lokal $\rightarrow$ Pastikan kode berjalan normal.
5. **Tahap 5: Reverse Proxy & Enkripsi SSL**:
   - Konfigurasi Nginx proxy $\rightarrow$ Terbitkan sertifikat Let's Encrypt via Certbot $\rightarrow$ Verifikasi HTTPS pada peramban.
6. **Tahap 6: Pengamanan Firewall & Hak Akses**:
   - Aktifkan firewall Proxmox setelah memastikan port manajemen telah masuk dalam daftar izin (*whitelist*) $\rightarrow$ Konfigurasi user juri $\rightarrow$ Atur jadwal backup.
7. **Tahap 7: Uji Ketahanan Pasca-Reboot (20 Menit Sebelum Selesai)**:
   - Lakukan restart pada seluruh node server. Verifikasi kembali ketersediaan layanan web melalui tautan `https://app.nama_peserta.my.id`.
