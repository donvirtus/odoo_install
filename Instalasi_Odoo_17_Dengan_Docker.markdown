# Instalasi Odoo 17 dengan Docker di Orange Pi (ARM64)

Panduan ini menjelaskan cara menginstal Odoo Community Edition versi 17 di perangkat berbasis ARM64 seperti Orange Pi 3 LTS atau Orange Pi Zero 2 menggunakan Docker. Metode ini jauh lebih unggul daripada instalasi manual karena mengisolasi semua dependensi ke dalam kontainer, menjaga sistem utama tetap bersih, dan menghindari masalah kompatibilitas pustaka Python.

## Prasyarat
- Perangkat Orange Pi dengan sistem operasi berbasis Debian (misalnya, Armbian atau Ubuntu).
- Akses ke terminal dengan hak `sudo`.

## Langkah 1: Instalasi Docker & Docker Compose
Pertama, kita perlu menginstal Docker Engine dan plugin Docker Compose modern.

### Perbarui Sistem Anda
```bash
sudo apt update && sudo apt upgrade -y
```

### Instal Docker dan Plugin Compose
Nama paket untuk plugin Compose bisa berbeda. Coba `docker-compose-plugin` terlebih dahulu. Jika gagal, gunakan `docker-compose-v2`.

```bash
# Coba instal ini terlebih dahulu
sudo apt install docker.io docker-compose-plugin -y

# Jika gagal, gunakan nama paket ini
sudo apt install docker.io docker-compose-v2 -y
```

### (Sangat Direkomendasikan) Tambahkan User ke Grup Docker
Untuk menjalankan perintah Docker tanpa `sudo`:

```bash
sudo usermod -aG docker $USER
```

**PENTING**: Logout dan login kembali agar perubahan diterapkan.

## Langkah 2: Siapkan Lingkungan Proyek Odoo
Kita akan membuat direktori yang berisi konfigurasi dan data untuk Odoo.

### Buat Direktori Proyek
```bash
# Buat folder di home directory Anda dan langsung masuk ke dalamnya
mkdir ~/odoo-docker && cd ~/odoo-docker
```

### Buat File Konfigurasi `docker-compose.yml`
Buat file bernama `docker-compose.yml` menggunakan editor teks seperti `nano`:

```bash
nano docker-compose.yml
```

Salin dan tempel seluruh konten di bawah ini ke dalam editor. Ganti `password_aman_anda` dengan password yang kuat untuk database.

```yaml
version: '3.1'
services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD=password_aman_anda
      - POSTGRES_USER=odoo
    volumes:
      - ./postgresql:/var/lib/postgresql/data
    restart: always

  odoo:
    image: odoo:17.0
    depends_on:
      - db
    ports:
      - "8069:8069"
    volumes:
      - ./addons:/mnt/extra-addons
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=password_aman_anda
    restart: always
```

Simpan file dan keluar (di `nano`: `Ctrl+X`, `Y`, `Enter`).

### Buat Folder Pendukung
Docker Compose membutuhkan folder ini untuk menyimpan data persisten:

```bash
# Folder untuk data database PostgreSQL
mkdir postgresql

# Folder untuk custom addons Odoo Anda di masa depan
mkdir addons
```

## Langkah 3: Jalankan Odoo!
Jalankan perintah berikut dari dalam direktori `~/odoo-docker`:

```bash
# Gunakan perintah modern tanpa tanda hubung (-)
docker compose up -d
```

Docker akan mengunduh image Odoo 17 dan PostgreSQL 15 yang kompatibel dengan ARM64, lalu menjalankannya di latar belakang. Proses unduh mungkin memakan waktu beberapa menit saat pertama kali dijalankan.

## Langkah 4: Verifikasi dan Akses

### Periksa Status Kontainer
Pastikan kedua layanan berjalan dengan baik:

```bash
docker ps
```

Anda akan melihat dua kontainer dengan status `Up`.

### Akses Odoo di Browser
Buka browser dan kunjungi alamat IP Orange Pi Anda di port 8069:

```
http://<ALAMAT_IP_ORANGE_PI>:8069
```

Anda akan disambut oleh halaman setup Odoo!

## Manajemen Sehari-hari

### Melihat Log Odoo secara Real-time
```bash
docker compose logs -f odoo
```

### Menghentikan Semua Layanan Odoo
```bash
docker compose down
```

### Memulai Kembali Semua Layanan
```bash
docker compose up -d
```
