# Langkah-langkah Menambah Swap File di Linux (Debian/Ubuntu)

Dokumentasi ini berisi langkah-langkah untuk membuat dan mengaktifkan swap file baru sebesar 4GB.

## 1. Buat Swap File

Gunakan `fallocate` untuk membuat file kosong dengan ukuran yang diinginkan secara instan. Di sini, kita membuat file 4GB bernama `/swapfile`.

```bash
sudo fallocate -l 4G /swapfile
```

## 2. Atur Izin Akses (Permissions)

Untuk alasan keamanan, hanya user `root` yang boleh membaca dan menulis ke swap file. Atur izinnya ke `600`.

```bash
sudo chmod 600 /swapfile
```

## 3. Format dan Aktifkan Swap

Format file tersebut agar dikenali sebagai area swap, lalu langsung aktifkan.

```bash
# Format file
sudo mkswap /swapfile

# Aktifkan swap file
sudo swapon /swapfile
```

## 4. Verifikasi

Periksa apakah swap file sudah aktif dan total swap sudah bertambah.

```bash
# Tampilkan semua area swap yang aktif beserta prioritasnya
sudo swapon --show

# Tampilkan penggunaan memori dan swap secara keseluruhan
free -h
```

## 5. Jadikan Permanen

Agar swap file otomatis aktif setiap kali sistem dinyalakan, tambahkan entri ke file `/etc/fstab`.

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Analisis Hasil Perintah

Semua hasil yang Mas Dion tunjukkan sudah terlihat benar dan sempurna.

- **`free -h`**: Outputnya menunjukkan `Swap: 5.0Gi`, yang mengonfirmasi bahwa total ruang swap adalah 4GB dari `/swapfile` ditambah ~1GB dari `zram`.
- **`swapon --show`**: Menunjukkan dua area swap aktif:
  - `/dev/zram0` dengan prioritas (`PRIO`) **5**.
  - `/swapfile` dengan prioritas **-2**.
  Karena prioritas `zram0` lebih tinggi, sistem akan menggunakan ZRAM terlebih dahulu sebelum `/swapfile`. Ini adalah konfigurasi ideal.
- **`echo ... | sudo tee ...`**: Berhasil menambahkan konfigurasi ke `/etc/fstab`, memastikan `/swapfile` aktif setelah reboot.

Konfigurasi ini sudah optimal, Mas Dion.
