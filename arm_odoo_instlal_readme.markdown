# Panduan Instalasi Odoo Community Edition di Orange Pi (Ubuntu 22.04, ARM Architecture)

Panduan ini menjelaskan cara menginstal **Odoo 17.0 Community Edition** menggunakan skrip otomatis pada **Orange Pi 3 LTS** (ARM64) atau **Orange Pi Zero 2** (ARMv8-A) dengan Ubuntu 22.04 (Jammy) berbasis Armbian. Panduan ini dioptimalkan untuk kebutuhan sistem inventory dan mencakup penyesuaian untuk arsitektur ARM.

## Persyaratan
- **Perangkat**: Orange Pi 3 LTS (2GB RAM, ARM64) atau Orange Pi Zero 2 (1GB RAM, ARMv8-A).
- **Sistem Operasi**: Ubuntu 22.04 (Jammy) berbasis Armbian.
- **MicroSD**: Minimal 16GB (Class 10 direkomendasikan).
- **Koneksi Internet**: Stabil untuk mengunduh paket.
- **Akses Root/Sudo**: Diperlukan untuk instalasi.
- **Ruang Disk Kosong**: Minimal 5GB untuk Odoo dan dependensi.
- **Sistem Bersih**: Pastikan instalasi Odoo sebelumnya sudah dihapus (lihat `uninstall_odoo_README.markdown`).

## Langkah-Langkah Instalasi

### 1. Verifikasi Sistem Bersih
Pastikan tidak ada sisa instalasi Odoo sebelumnya:
```bash
sudo ls /opt/odoo
sudo ls /odoo/odoo-server
sudo ls /odoo/custom
sudo ls /etc/odoo*
sudo ls /var/log/odoo
sudo systemctl status odoo
dpkg -l | grep postgresql
sudo ls /var/lib/postgresql
sudo ls /usr/local/lib/wkhtmltox*
sudo ls /usr/local/share/wkhtmltox*
sudo ls /usr/local/bin/wkhtmltopdf
```
Jika ada file/direktori/layanan yang ditemukan, ikuti panduan pembersihan di `uninstall_odoo_README.markdown` (ID: `c74410a1-7ffa-4e7e-a921-3309e8dc9a87`).

### 2. Perbarui Sistem
Perbarui semua paket di sistem Anda:
```bash
sudo apt update && sudo apt upgrade -y
```

### 3. Tambah Swap Space (untuk Orange Pi Zero 2)
Orange Pi Zero 2 memiliki RAM 1GB, yang terbatas untuk Odoo 17.0. Tambahkan swap space untuk mencegah crash:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Catatan**: Langkah ini opsional untuk Orange Pi 3 LTS (2GB RAM) tetapi sangat disarankan untuk Zero 2.

### 4. Unduh Skrip Instalasi
Unduh skrip instalasi Odoo 17.0 dari Yenthe666:
```bash
wget https://raw.githubusercontent.com/Yenthe666/InstallScript/17.0/odoo_install.sh
```

**Catatan**: Untuk versi lain (misalnya, Odoo 16.0), ganti `17.0` dengan versi yang diinginkan (misalnya, `16.0`) pada URL.

### 5. Jadikan Skrip Dapat Dieksekusi
Beri izin eksekusi pada skrip:
```bash
sudo chmod +x odoo_install.sh
```

### 6. Modifikasi Skrip untuk ARM (wkhtmltopdf)
Skrip otomatis mungkin menginstal `wkhtmltopdf` versi x86_64, yang tidak kompatibel dengan ARM. Modifikasi skrip untuk menggunakan versi ARM64:
```bash
nano odoo_install.sh
```
Cari baris yang menginstal `wkhtmltopdf` (biasanya berisi `apt-get install wkhtmltopdf`). Ganti dengan:
```bash
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_arm64.deb
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_arm64.deb
sudo apt install -f
```
Simpan dengan `Ctrl+O`, lalu keluar dengan `Ctrl+X`.

### 7. Jalankan Skrip Instalasi
Jalankan skrip untuk menginstal Odoo dan dependensinya (proses ini memakan waktu 10-20 menit):
```bash
sudo ./odoo_install.sh
```

**Catatan**: Jika terjadi error terkait `wkhtmltopdf` atau dependensi lain, periksa log di Terminal dan lihat bagian **Troubleshooting**.

### 8. Optimasi Konfigurasi untuk RAM Terbatas
Untuk Orange Pi Zero 2, edit file konfigurasi Odoo untuk mengurangi penggunaan memori:
```bash
sudo nano /etc/odoo-server.conf
```
Tambahkan atau ubah baris berikut:
```ini
[options]
workers = 0
limit_memory_hard = 268435456
limit_memory_soft = 214748364
```
Simpan dengan `Ctrl+O`, lalu keluar dengan `Ctrl+X`. Restart layanan Odoo:
```bash
sudo systemctl restart odoo
```

### 9. Verifikasi Instalasi
Periksa status layanan Odoo:
```bash
sudo systemctl status odoo
```
Pastikan statusnya `active (running)`.

Verifikasi `wkhtmltopdf`:
```bash
wkhtmltopdf --version
```
Output yang diharapkan: `wkhtmltopdf 0.12.6.1 (with patched qt)`.

### 10. Akses Odoo di Browser
Buka browser dan akses:
```
http://<IP_ORANGE_PI>:8069
```
Ikuti langkah-langkah untuk membuat database baru:
- Masukkan nama database (misalnya, `inventory_db`).
- Atur kata sandi admin.
- Pilih bahasa dan negara.

### 11. Konfigurasi Awal untuk Sistem Inventory
1. Masuk ke Odoo melalui browser.
2. Buka menu **Apps**, cari dan instal modul **Inventory**.
3. Buka **Inventory** > **Configuration** > **Settings** untuk mengaktifkan fitur seperti **Multi-Warehouse** atau **Barcode** sesuai kebutuhan.
4. Tambahkan informasi perusahaan di **Settings** > **General Settings**.

## Informasi Penting
- **Lokasi Instalasi**: Odoo terinstal di `/odoo/odoo-server/`.
- **File Konfigurasi**: File konfigurasi berada di `/etc/odoo-server.conf`. Periksa file ini untuk *Master Password* yang dibuat oleh skrip.
- **Log**: Log Odoo berada di `/var/log/odoo/odoo-server.log`.
- **Pendingin (Orange Pi 3 LTS)**: Pasang heatsink atau kipas untuk mencegah overheating saat memproses data besar.
- **Odoo 17.0 vs 16.0**: Odoo 17.0 lebih berat. Untuk Orange Pi Zero 2, pertimbangkan Odoo 16.0 (lihat panduan manual ID: `2a95dcb6-751e-4ca7-9f93-74aac47abdc8`) jika performa lambat.

## Troubleshooting
- **Error wkhtmltopdf**:
  Jika PDF tidak dihasilkan, pastikan versi ARM64 terinstal:
  ```bash
  wkhtmltopdf --version
  ```
  Jika tidak menunjukkan `(with patched qt)`, instal ulang:
  ```bash
  sudo apt purge -y wkhtmltopdf wkhtmltox
  wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_arm64.deb
  sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_arm64.deb
  sudo apt install -f
  ```

- **Odoo Tidak Berjalan**:
  Periksa log:
  ```bash
  sudo cat /var/log/odoo/odoo-server.log
  ```
  Pastikan PostgreSQL berjalan:
  ```bash
  sudo systemctl status postgresql
  ```

- **Memori Rendah (Zero 2)**:
  Jika Odoo crash, periksa swap space:
  ```bash
  swapon -s
  ```
  Tambahkan lebih banyak swap jika perlu:
  ```bash
  sudo fallocate -l 4G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  ```

- **Konflik File Konfigurasi**:
  Jika ada error terkait `/etc/odoo-server.conf`, pastikan tidak ada file duplikat:
  ```bash
  sudo rm -f /etc/odoo.conf /etc/odoo-server.conf
  sudo ./odoo_install.sh
  ```
