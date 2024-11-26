# 🌐 Panduan Menyiapkan VPS

Panduan ini membantu Anda menyiapkan VPS hingga **siap produksi** dengan langkah-langkah sederhana. Cocok untuk pemula yang ingin mempelajari dasar-dasar pengelolaan VPS.

---

## 🛠️ Fitur VPS Siap Produksi

- 🌍 **DNS Konfigurasi**: Menghubungkan nama domain ke VPS.
- 🔒 **Keamanan**: Pengamanan koneksi dengan TLS otomatis, SSH aman, dan firewall.
- 📈 **Pemantauan**: Notifikasi jika aplikasi Anda mengalami downtime.
- ⚡ **Ketersediaan Tinggi**: Load balancing untuk menjaga reliabilitas.
- 🔄 **Penerapan Otomatis**: Mempermudah update aplikasi tanpa downtime.

---

## 📋 Persyaratan

1. VPS dengan sistem operasi, disarankan **Ubuntu 20.04 LTS**.
2. Nama domain untuk aplikasi atau layanan Anda.
3. Dasar-dasar penggunaan terminal dan SSH.

---

## 🚀 Langkah-Langkah Setup

### 1️⃣ Login ke VPS

Masuk ke VPS Anda menggunakan SSH:
```bash
ssh root@IP_SERVER
```

*Penjelasan*: Perintah ini digunakan untuk masuk ke server VPS menggunakan akun root dengan alamat IP yang disediakan.

---

### 2️⃣ Membuat Pengguna Baru

Demi keamanan, hindari menggunakan root. Buat pengguna baru:
```bash
adduser nama_pengguna  # Membuat pengguna baru dengan nama yang diinginkan
usermod -aG sudo nama_pengguna  # Menambahkan pengguna baru ke grup sudo agar memiliki izin administratif
```

Masuk sebagai pengguna baru:
```bash
su - nama_pengguna  # Beralih ke akun pengguna baru yang telah dibuat
```

*Penjelasan*: Hal ini diperlukan untuk meningkatkan keamanan server. Menggunakan akun root secara langsung dapat berisiko tinggi.

---

### 3️⃣ Mengatur DNS untuk Nama Domain

1. **Beli domain** dari penyedia layanan seperti Hostinger.
2. Tambahkan **A Record** di pengaturan DNS domain Anda, arahkan ke IP VPS.

Contoh:
| Tipe | Nama  | Alamat IP     |
|------|-------|---------------|
| A    | @     | 192.168.1.1   |

*Penjelasan*: Menambahkan A Record mengarahkan domain ke alamat IP VPS sehingga pengguna dapat mengakses aplikasi melalui nama domain.

---

### 4️⃣ Mengamankan SSH

1. Salin kunci SSH dari komputer lokal Anda ke VPS:
   ```bash
   ssh-copy-id nama_pengguna@IP_SERVER  # Mengirim kunci publik SSH ke server agar dapat login tanpa password
   ```

2. Edit konfigurasi SSH untuk menonaktifkan login dengan password:
   ```bash
   sudo nano /etc/ssh/sshd_config  # Membuka file konfigurasi SSH untuk diedit
   ```
   Ubah:
   ```
   UsePAM no  # Menonaktifkan otentikasi PAM
   PasswordAuthentication no  # Menonaktifkan login dengan password
   PermitRootLogin no  # Melarang login sebagai root
   ```
3. Restart layanan SSH:
   ```bash
   sudo systemctl restart ssh  # Merestart layanan SSH agar perubahan diterapkan
   ```

*Penjelasan*: Menonaktifkan login password meningkatkan keamanan server dengan memaksa pengguna menggunakan kunci SSH.

---

### 5️⃣ Menginstal Firewall

Gunakan **UFW** (Uncomplicated Firewall) untuk melindungi VPS Anda:
```bash
sudo apt install ufw  # Menginstal UFW sebagai firewall
sudo ufw default deny incoming  # Menolak semua koneksi masuk secara default
sudo ufw default allow outgoing  # Mengizinkan semua koneksi keluar secara default
sudo ufw allow ssh  # Mengizinkan koneksi SSH
sudo ufw allow http  # Mengizinkan koneksi HTTP
sudo ufw allow https  # Mengizinkan koneksi HTTPS
sudo ufw enable  # Mengaktifkan firewall
```

*Penjelasan*: Firewall digunakan untuk mengontrol lalu lintas jaringan dan melindungi server dari serangan berbahaya.

---

### 6️⃣ Menginstal Docker dan Docker Compose

1. Instal Docker:
   ```bash
   sudo apt update  # Memperbarui daftar paket
   sudo apt install docker.io  # Menginstal Docker
   ```
2. Tambahkan pengguna ke grup Docker:
   ```bash
   sudo usermod -aG docker nama_pengguna  # Memberi akses Docker ke pengguna baru tanpa sudo
   ```
3. Instal Docker Compose:
   ```bash
   sudo apt install docker-compose  # Menginstal Docker Compose untuk manajemen layanan
   ```

*Penjelasan*: Docker digunakan untuk containerisasi aplikasi, sedangkan Docker Compose memudahkan pengelolaan beberapa kontainer sekaligus.

---

### 7️⃣ Menambahkan Proxy Terbalik dengan TLS

Gunakan **Traefik** untuk proxy terbalik dengan TLS otomatis. Tambahkan konfigurasi berikut ke `docker-compose.yml`:
```yaml
proxy:
  image: traefik:v3.1  # Menggunakan image Traefik versi 3.1 sebagai proxy terbalik
  ports:
    - "80:80"  # Memetakan port 80 untuk HTTP
    - "443:443"  # Memetakan port 443 untuk HTTPS
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock  # Menghubungkan Docker socket agar Traefik dapat mendeteksi layanan
    - ./letsencrypt:/letsencrypt  # Direktori untuk menyimpan sertifikat TLS
  command:
    - "--entrypoints.web.address=:80"  # Menentukan entry point untuk HTTP
    - "--entrypoints.websecure.address=:443"  # Menentukan entry point untuk HTTPS
    - "--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web"  # Menggunakan HTTP challenge untuk ACME
    - "--certificatesresolvers.myresolver.acme.email=email@example.com"  # Email untuk pendaftaran sertifikat ACME
    - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"  # Menyimpan data sertifikat
```

Jalankan ulang Docker Compose:
```bash
docker-compose down  # Menghentikan layanan yang berjalan
docker-compose up -d  # Menjalankan layanan dalam mode background
```

*Penjelasan*: Traefik bertindak sebagai proxy terbalik yang mengatur lalu lintas masuk dan menyediakan sertifikat SSL secara otomatis.

---

### 8️⃣ Menyiapkan Pemantauan

Daftarkan domain Anda ke layanan seperti [UptimeRobot](https://uptimerobot.com/) untuk mendapatkan notifikasi jika terjadi downtime.

*Penjelasan*: Pemantauan uptime sangat penting untuk memastikan bahwa aplikasi Anda selalu tersedia dan Anda akan diberi tahu jika ada masalah.

---

### lainnya:  Memberikan Hak Akses PPK ke Pengguna `nama_pengguna` (jika tidak bisa no 1 di mengamakan ssh)

Agar pengguna `nama_pengguna` dapat memiliki akses PPK (Public Private Key), ikuti langkah-langkah berikut:

1. Login ke VPS menggunakan akun `root` atau akun dengan hak sudo:
   ```bash
   ssh root@IP_SERVER  # Masuk ke VPS menggunakan akun root atau sudo
   ```

2. Buat direktori `.ssh` untuk pengguna `nama_pengguna` (jika belum ada):
   ```bash
   sudo mkdir -p /home/nama_pengguna/.ssh  # Membuat direktori .ssh untuk menyimpan kunci
   ```

3. Setel izin direktori `.ssh`:
   ```bash
   sudo chmod 700 /home/nama_pengguna/.ssh  # Mengatur izin agar hanya pemilik yang dapat mengakses
   ```

4. Buat atau tambahkan kunci publik ke file `authorized_keys`:
   ```bash
   sudo nano /home/nama_pengguna/.ssh/authorized_keys  # Membuka file authorized_keys untuk menambahkan kunci publik
   ```
   Tempelkan kunci publik yang ingin Anda gunakan untuk mengakses akun `nama_pengguna`.

5. Setel izin file `authorized_keys`:
   ```bash
   sudo chmod 600 /home/nama_pengguna/.ssh/authorized_keys  # Mengatur izin agar hanya pemilik yang dapat membaca dan menulis
   ```

6. Ubah kepemilikan direktori `.ssh` dan file `authorized_keys` ke pengguna `nama_pengguna`:
   ```bash
   sudo chown -R nama_pengguna:nama_pengguna /home/nama_pengguna/.ssh  # Mengubah kepemilikan agar sesuai dengan pengguna
   ```

*Penjelasan*: Langkah-langkah ini memastikan bahwa hanya pengguna `nama_pengguna` yang dapat mengakses VPS menggunakan kunci SSH.

---
### Rangkuman dari sumber-sumber:
- [Setting up a production ready VPS is a lot easier than I thought.](https://www.youtube.com/watch?v=F-9KWQByeU0)
- bersambung
