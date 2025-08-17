# Panduan Instalasi Odoo Community Edition di Orange Pi (ARM Architecture)

Panduan ini menjelaskan cara menginstal **Odoo 16.0 Community Edition** pada perangkat **Orange Pi** (seperti Orange Pi 3 LTS atau Orange Pi Zero 2) dengan prosesor berarsitektur ARM (ARM64 atau ARMv8-A) menggunakan **Ubuntu 22.04 (Jammy)** berbasis Armbian. Panduan ini dioptimalkan untuk kebutuhan sistem inventory dengan sumber daya terbatas.

## Persyaratan
- **Perangkat**: Orange Pi 3 LTS (2GB RAM, ARM64) atau Orange Pi Zero 2 (1GB RAM, ARMv8-A).
- **Sistem Operasi**: Ubuntu 22.04 (Jammy) berbasis Armbian.
- **MicroSD**: Minimal 16GB (Class 10 direkomendasikan).
- **Koneksi Internet**: Stabil untuk mengunduh paket.
- **Akses Root/Sudo**: Diperlukan untuk instalasi.
- **Ruang Disk Kosong**: Minimal 5GB untuk Odoo dan dependensi.

**Catatan**: Orange Pi Zero 2 memiliki RAM terbatas (1GB), jadi gunakan hanya modul yang diperlukan (misalnya, Inventory) dan tambahkan swap space untuk performa optimal.

## Langkah-Langkah Instalasi

### 1. Persiapan Sistem
1. **Unduh dan Flash Armbian**:
   - Unduh image Armbian Ubuntu 22.04 (Jammy) untuk perangkat Anda dari [Armbian Downloads](https://www.armbian.com/).
   - Flash image ke microSD menggunakan alat seperti **Balena Etcher**.
   - Masukkan microSD ke Orange Pi, nyalakan, dan ikuti konfigurasi awal Armbian (buat pengguna, kata sandi, dll.).

2. **Perbarui Sistem**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Instal Alat Dasar**:
   ```bash
   sudo apt install -y nano git wget curl
   ```

### 2. Instal Dependensi Odoo
Odoo memerlukan PostgreSQL, Python, dan beberapa pustaka tambahan.

1. **Instal PostgreSQL**:
   ```bash
   sudo apt install -y postgresql
   sudo -u postgres createuser -s odoo
   ```

2. **Instal Dependensi Sistem**:
   ```bash
   sudo apt install -y python3-pip python3-dev libxml2-dev libxslt1-dev zlib1g-dev \
   libldap2-dev libsasl2-dev libjpeg-dev libpq-dev build-essential
   ```

3. **Instal wkhtmltopdf (untuk laporan PDF)**:
   ```bash
   sudo apt install -y xfonts-75dpi xfonts-base libxrender1 libfontconfig1
   sudo wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_arm64.deb
   sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_arm64.deb
   sudo apt install -f
   ```
   Verifikasi instalasi:
   ```bash
   wkhtmltopdf --version
   ```
   Output yang diharapkan: `wkhtmltopdf 0.12.6.1 (with patched qt)`.

### 3. Unduh dan Instal Odoo
1. **Unduh Odoo 16.0 dari GitHub**:
   ```bash
   sudo git clone --depth 1 --branch 16.0 https://www.github.com/odoo/odoo /opt/odoo
   ```

2. **Instal Dependensi Python**:
   ```bash
   cd /opt/odoo
   sudo pip3 install -r requirements.txt
   ```

   **Catatan**: Jika ada masalah dengan `pip3`, perbarui pip:
   ```bash
   sudo pip3 install --upgrade pip
   ```

### 4. Konfigurasi Odoo
1. **Buat File Konfigurasi**:
   ```bash
   sudo nano /etc/odoo.conf
   ```
   Tambahkan isi berikut:
   ```ini
   [options]
   admin_passwd = your_secure_password
   http_port = 8069
   logfile = /var/log/odoo/odoo.log
   addons_path = /opt/odoo/addons
   db_host = False
   db_port = False
   db_user = odoo
   db_password = False
   ```
   Ganti `your_secure_password` dengan kata sandi yang kuat. Simpan dengan `Ctrl+O`, lalu keluar dengan `Ctrl+X`.

2. **Buat Direktori Log**:
   ```bash
   sudo mkdir /var/log/odoo
   sudo chown $USER:$USER /var/log/odoo
   ```

3. **Uji Coba Jalankan Odoo**:
   ```bash
   python3 /opt/odoo/odoo-bin -c /etc/odoo.conf
   ```
   Jika berhasil, Odoo akan berjalan di `http://localhost:8069` atau `http://<IP_ORANGE_PI>:8069`.

### 5. Jalankan Odoo sebagai Layanan
1. **Buat File Layanan**:
   ```bash
   sudo nano /etc/systemd/system/odoo.service
   ```
   Tambahkan isi berikut:
   ```ini
   [Unit]
   Description=Odoo ERP
   After=network.target postgresql.service

   [Service]
   Type=simple
   User=$USER
   ExecStart=/usr/bin/python3 /opt/odoo/odoo-bin -c /etc/odoo.conf
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

2. **Aktifkan dan Jalankan Layanan**:
   ```bash
   sudo systemctl enable odoo
   sudo systemctl start odoo
   ```

3. **Periksa Status Layanan**:
   ```bash
   sudo systemctl status odoo
   ```
   Pastikan statusnya `active (running)`.

### 6. Konfigurasi Awal Odoo untuk Sistem Inventory
1. **Akses Odoo**:
   Buka browser dan kunjungi `http://<IP_ORANGE_PI>:8069`.
   Buat database baru:
   - Masukkan nama database (misalnya, `inventory_db`).
   - Atur kata sandi admin.
   - Pilih bahasa dan negara.

2. **Instal Modul Inventory**:
   - Masuk ke menu **Apps**.
   - Cari dan instal modul **Inventory**.
   - (Opsional) Instal modul tambahan seperti **Sales** atau **Purchase** jika diperlukan.

3. **Konfigurasi Inventory**:
   - Buka **Inventory** > **Configuration** > **Settings**.
   - Aktifkan fitur seperti **Multi-Warehouse** atau **Barcode** sesuai kebutuhan.
   - Tambahkan informasi perusahaan di **Settings** > **General Settings**.

### 7. Optimasi untuk Orange Pi
- **Tambah Swap Space** (khususnya untuk Orange Pi Zero 2):
  ```bash
  sudo fallocate -l 2G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  ```
- **Batasi Modul**: Hanya instal modul yang diperlukan (misalnya, Inventory) untuk menghemat RAM.
- **Pendingin**: Pasang heatsink atau kipas pada Orange Pi 3 LTS untuk mencegah overheating saat Odoo memproses data besar.

### 8. Troubleshooting Umum
- **Error wkhtmltopdf**:
  Jika PDF tidak dihasilkan, pastikan `wkhtmltopdf` versi 0.12.6.1 dengan Qt patch terinstal:
  ```bash
  wkhtmltopdf --version
  ```
  Jika tidak menunjukkan `(with patched qt)`, instal ulang seperti pada Langkah 2.3.

- **Odoo Tidak Berjalan**:
  Periksa log di `/var/log/odoo/odoo.log`:
  ```bash
  sudo cat /var/log/odoo/odoo.log
  ```

- **Database Error**:
  Pastikan PostgreSQL berjalan:
  ```bash
  sudo systemctl status postgresql
  ```

- **Memori Rendah**:
  Untuk Orange Pi Zero 2, nonaktifkan fitur Odoo yang tidak diperlukan di `/etc/odoo.conf`:
  ```ini
  workers = 0
  limit_memory_hard = 268435456
  limit_memory_soft = 214748364
  ```

### 9. Backup dan Pemeliharaan
- **Backup Database**:
  ```bash
  sudo -u postgres pg_dump inventory_db > inventory_db_backup.sql
  ```
- **Perbarui Odoo**:
  ```bash
  cd /opt/odoo
  sudo git pull origin 16.0
  sudo pip3 install -r requirements.txt
  sudo systemctl restart odoo
  ```

## Catatan
- **Versi Odoo**: Panduan ini menggunakan Odoo 16.0. Untuk versi lain (misalnya, 17.0), ubah branch pada perintah `git clone` (Langkah 3.1).
- **Sumber Daya**: Orange Pi 3 LTS lebih cocok untuk menjalankan beberapa modul, sedangkan Orange Pi Zero 2 lebih terbatas dan memerlukan optimasi.
- **Dokumentasi Resmi**: Lihat [Odoo Documentation](https://www.odoo.com/documentation/16.0/) untuk panduan lanjutan tentang modul Inventory.
