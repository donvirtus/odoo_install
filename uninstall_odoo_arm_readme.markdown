# Panduan Menghapus Instalasi Odoo di Orange Pi (Ubuntu 22.04, ARM Architecture)

Panduan ini menjelaskan cara menghapus semua jejak instalasi **Odoo Community Edition** (baik dari instalasi manual maupun otomatis) pada perangkat **Orange Pi** (seperti Orange Pi 3 LTS atau Zero 2) dengan Ubuntu 22.04 (Jammy) berbasis Armbian. Langkah-langkah ini memastikan sistem bersih sebelum instalasi ulang Odoo untuk kebutuhan sistem inventory.

## Persyaratan
- **Perangkat**: Orange Pi 3 LTS (ARM64) atau Orange Pi Zero 2 (ARMv8-A).
- **Sistem Operasi**: Ubuntu 22.04 (Jammy) berbasis Armbian.
- **Akses Root/Sudo**: Diperlukan untuk menghapus file dan layanan.
- **Cadangan Data**: Pastikan Anda mencadangkan database penting sebelum menghapus.

## Langkah-Langkah Pembersihan

### 1. Hentikan Layanan Odoo
Hentikan dan nonaktifkan layanan Odoo untuk mencegah proses berjalan di latar belakang:
```bash
sudo systemctl stop odoo
sudo systemctl disable odoo
```

**Catatan**: Jika layanan tidak ada, Anda mungkin melihat pesan error, yang aman untuk diabaikan.

### 2. Hapus File Layanan Odoo
Hapus file layanan sistem Odoo:
```bash
sudo rm -f /etc/systemd/system/odoo.service
sudo systemctl daemon-reload
sudo systemctl reset-failed
```

**Penjelasan**: Ini menghapus definisi layanan Odoo dan memperbarui daftar layanan sistem.

### 3. Hapus Direktori Instalasi Odoo
Hapus direktori instalasi Odoo dari instalasi manual maupun otomatis:
```bash
sudo rm -rf /opt/odoo
sudo rm -rf /odoo/odoo-server
sudo rm -rf /odoo/custom
sudo rm -rf /odoo
```

**Penjelasan**:
- `/opt/odoo`: Direktori instalasi manual.
- `/odoo/odoo-server`: Direktori instalasi dari skrip otomatis (misalnya, Yenthe666).
- `/odoo/custom`: Direktori untuk addons kustom, jika ada.
- `/odoo`: Direktori utama untuk memastikan tidak ada sisa subdirektori.

### 4. Hapus File Konfigurasi
Hapus file konfigurasi Odoo:
```bash
sudo rm -f /etc/odoo.conf
sudo rm -f /etc/odoo-server.conf
```

**Penjelasan**: Ini menghapus file konfigurasi dari instalasi manual (`/etc/odoo.conf`) dan otomatis (`/etc/odoo-server.conf`).

### 5. Hapus Direktori Log
Hapus direktori log Odoo:
```bash
sudo rm -rf /var/log/odoo
```

**Penjelasan**: Ini menghapus file log yang dibuat oleh Odoo.

### 6. Hapus Pengguna dan Database PostgreSQL
Odoo menggunakan PostgreSQL sebagai database. Hapus pengguna dan database terkait jika PostgreSQL masih terinstal:

1. **Periksa Versi PostgreSQL**:
   ```bash
   dpkg -l | grep postgresql
   ```
   Catat versi PostgreSQL yang terinstal (misalnya, `postgresql-16`).

2. **Periksa Daftar Database**:
   ```bash
   sudo -u postgres psql -l
   ```
   **Catatan**: Jika perintah ini menghasilkan error `Connection refused`, PostgreSQL mungkin tidak aktif atau sudah dihapus. Lewati ke Langkah 6.4 jika error terjadi.

3. **Hapus Database**:
   Untuk setiap database Odoo (misalnya, `inventory_db`):
   ```bash
   sudo -u postgres dropdb inventory_db
   ```
   Ganti `inventory_db` dengan nama database yang sesuai.

4. **Hapus Pengguna PostgreSQL**:
   ```bash
   sudo -u postgres dropuser odoo
   ```

5. **(Opsional) Hapus PostgreSQL Sepenuhnya**:
   Jika Anda tidak memerlukan PostgreSQL untuk aplikasi lain:
   ```bash
   sudo apt purge -y postgresql postgresql-contrib postgresql-16 postgresql-client-16 postgresql-client-common postgresql-common postgresql-common-dev
   sudo apt autoremove -y
   sudo rm -rf /var/lib/postgresql
   sudo rm -rf /var/run/postgresql
   ```

**Penjelasan**: Perintah `apt purge` mencakup paket umum dan versi spesifik (misalnya, `postgresql-16`) untuk memastikan penghapusan lengkap.

### 7. Hapus Dependensi Odoo
Hapus dependensi yang tidak lagi diperlukan:
```bash
sudo apt autoremove -y
sudo apt autoclean
```

**Penjelasan**: Ini menghapus paket yang tidak lagi dibutuhkan dan membersihkan cache apt.

### 8. Hapus wkhtmltopdf/wkhtmltox
Hapus `wkhtmltopdf` dan `wkhtmltox`, yang digunakan untuk laporan PDF:
```bash
sudo apt purge -y wkhtmltopdf wkhtmltox
sudo rm -rf /usr/local/lib/wkhtmltox*
sudo rm -rf /usr/local/share/wkhtmltox*
sudo rm -rf /usr/local/bin/wkhtmltopdf
sudo rm -f wkhtmltox_*.deb
```

**Penjelasan**: Ini menghapus file `wkhtmltopdf` dan sisa file dari instalasi manual.

### 9. Verifikasi Pembersihan
Periksa apakah semua jejak Odoo telah dihapus:
```bash
sudo ls /opt/odoo
sudo ls /odoo/odoo-server
sudo ls /odoo/custom
sudo ls /odoo
sudo ls /etc/odoo*
sudo ls /var/log/odoo
sudo systemctl status odoo
dpkg -l | grep postgresql
sudo ls /var/lib/postgresql
sudo ls /usr/local/lib/wkhtmltox*
sudo ls /usr/local/share/wkhtmltox*
sudo ls /usr/local/bin/wkhtmltopdf
```

**Penjelasan**:
- Gunakan `sudo ls` untuk menghindari error `Permission denied` saat memeriksa direktori seperti `/odoo`.
- Jika perintah `ls` menunjukkan `No such file or directory`, pembersihan berhasil.
- Jika `sudo systemctl status odoo` menunjukkan `Unit odoo.service could not be found`, layanan Odoo sudah dihapus.
- Jika `dpkg -l | grep postgresql` tidak menghasilkan output, PostgreSQL sudah dihapus.
- Jika Anda mendapatkan error `Connection refused` saat menjalankan `sudo -u postgres psql -l`, ini normal jika PostgreSQL sudah dihapus.

### 10. (Opsional) Cadangkan Data Penting
Sebelum menghapus database, cadangkan data penting:
```bash
sudo -u postgres pg_dump inventory_db > inventory_db_backup.sql
```
Simpan file `inventory_db_backup.sql` ke lokasi aman (misalnya, USB atau cloud) untuk pemulihan nanti.

## Catatan
- **Keamanan**: Pastikan Anda benar-benar ingin menghapus semua data Odoo, karena langkah-langkah ini tidak dapat dibatalkan kecuali Anda memiliki cadangan.
- **Sumber Daya**: Pembersihan ini membebaskan ruang disk dan mencegah konflik saat instalasi ulang Odoo.
- **Instalasi Ulang**: Setelah pembersihan, Anda dapat menginstal ulang Odoo menggunakan panduan instalasi seperti di [README.md untuk Instalasi Odoo](#).

## Troubleshooting
- **Direktori Tidak Kosong**: Jika Anda mendapatkan peringatan seperti `directory not empty` saat menghapus `wkhtmltopdf` atau dependensi lain (misalnya, `/usr/share/fonts/X11/encodings/large`), ini biasanya aman untuk diabaikan kecuali direktori tersebut berisi file terkait Odoo atau `wkhtmltopdf`. Periksa isi direktori dengan:
  ```bash
  sudo ls /path/to/directory
  ```
  Hapus file terkait secara manual jika diperlukan:
  ```bash
  sudo rm -rf /path/to/directory/*
  ```

- **Database Tidak Dapat Dihapus**: Jika `dropdb` atau `dropuser` gagal karena error `Connection refused`, pastikan PostgreSQL berjalan:
  ```bash
  sudo systemctl start postgresql
  ```
  Jika masih gagal atau PostgreSQL sudah dihapus, lewati langkah ini dan lanjutkan ke penghapusan PostgreSQL (Langkah 6.5).

- **Layanan Masih Ada**: Jika `systemctl status odoo` masih menunjukkan layanan, ulangi Langkah 2.

- **Error Permission Denied**: Jika verifikasi seperti `ls /odoo/odoo-server` menghasilkan `Permission denied`, gunakan `sudo ls` untuk memeriksa direktori:
  ```bash
  sudo ls /odoo/odoo-server
  ```
