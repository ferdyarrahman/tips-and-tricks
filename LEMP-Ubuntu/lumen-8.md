# Cara Install dan Konfigurasi Lumen dengan Nginx di Unbuntu (LEMP)

## Jalankan pembaruan sistem

Sistem harus dalam kondisi terbaru untuk menghindari konflik paket. Oleh karena itu, sebelum melakukan installasi Anda jalankan perintah pembaruan sistem terlebih dahulu untuk memastikan semua pembaruan keamanan yang tersedia ada di Ubuntu.

```bash
sudo apt update && sudo apt upgrade
```

## Menginstal Nginx

Karena Nginx tersedia di repositori default Ubuntu, mungkinkan untuk menginstalnya dari repositori ini menggunakan apt sistem.

Anda dapat menginstal nginx:

```bash
sudo apt install nginx
```

## Menyesuaikan Firewall

Sebelum menguji Nginx, perangkat lunak firewall perlu disesuaikan untuk memungkinkan akses ke layanan. Nginx mendaftarkan dirinya sebagai layanan pada ufw saat instalasi, membuatnya mudah untuk mengizinkan akses Nginx.

Buat daftar konfigurasi aplikasi yang ufwmengetahui cara kerjanya dengan mengetik:

```bash
sudo ufw app list
```

Anda harus mendapatkan daftar profil aplikasi:

```bash
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

Seperti yang ditunjukkan oleh output-nya, ada tiga profil yang tersedia untuk Nginx:

Nginx Full : Profil ini membuka port 80 (lalu lintas web normal dan tidak terenkripsi) dan port 443 (lalu lintas terenkripsi TLS/SSL)
Nginx HTTP : Profil ini hanya membuka port 80 (lalu lintas web normal dan tidak terenkripsi)
Nginx HTTPS : Profil ini hanya membuka port 443 (lalu lintas terenkripsi TLS/SSL)
Disarankan agar Anda mengaktifkan profil paling ketat yang masih mengizinkan lalu lintas yang telah Anda konfigurasi. Saat ini, kita hanya perlu mengizinkan lalu lintas di port 80.

Anda dapat mengaktifkannya dengan mengetik:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow 'OpenSSH'
```

ouptunya akan seperti berikut:

```bash
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
Nginx HTTPS                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
Nginx HTTPS (v6)            ALLOW       Anywhere (v6)
```

Di akhir proses instalasi Nginx. Server web seharusnya sudah aktif dan berjalan.

Kita dapat memeriksa systemdsistem init untuk memastikan layanan berjalan dengan mengetik:

```bash
systemctl status nginx
```

## Tambahkan repository PPA Ondrej di Ubuntu 22.04

Kami tidak dapat menginstal paket PHP7.4 menggunakan repositori sistem default Ubuntu 22.04 karena versi default PHP yang ada di versi Ubuntu ini adalah PHP 8.1. Oleh karena itu, untuk mendapatkan versi yang lebih lama, tambahkan repositori PPA bernama Ondrej .

```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
```

## Instal PHP 7.4 di Ubuntu 22.04

Sekarang, Anda dapat menginstal PHP7.4 di Linux Ubuntu 22.04 Anda, namun Anda perlu menyebutkan nomor versi dengan perintah jika tidak, sistem akan menginstal php8.1 di sistem Anda. Berikut adalah perintah yang harus diikuti:

```bash
sudo apt install php7.4
```

Untuk Ekstensi Umum Anda dapat menggunakan:

```bash
sudo apt install php7.4-{cli,common,curl,zip,gd,mysql,xml,mbstring,json,intl}
```

Apabila proses instalasi selesai Anda dapat mengeceknya menggunakan perintah berikut:

```bash
php -v
```

## Mengunduh dan menginstal Composer

Composer menyediakan skrip installer yang ditulis dalam PHP. Kami akan mengunduhnya, memverifikasi bahwa itu tidak rusak, dan kemudian menggunakannya untuk menginstal Komposer.

Pastikan Anda berada di direktori home, lalu ambil penginstal menggunakan curl:

```bash
cd ~
curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
```

Selanjutnya, Anda akan memverifikasi bahwa penginstal yang diunduh cocok dengan hash SHA-384 untuk penginstal terbaru yang ditemukan di halaman Composer Public Keys / Signatures. Untuk memudahkan langkah verifikasi, Anda dapat menggunakan perintah berikut untuk mendapatkan hash terbaru secara terprogram dari halaman Composer dan menyimpannya dalam variabel shell:

```bash
HASH=`curl -sS https://composer.github.io/installer.sig`
```

Jika Anda ingin memverifikasi nilai yang diperoleh, Anda dapat menjalankan:

```bash
echo $HASH
```

Sekarang jalankan kode PHP berikut, seperti yang disediakan di halaman unduh Composer, untuk memverifikasi bahwa skrip instalasi aman untuk dijalankan:

```bash
php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

output:

```bash
Installer verified
```

Jika output berbunyi Installer corrupt, Anda perlu mengunduh skrip instalasi lagi dan memeriksa ulang apakah Anda menggunakan hash yang benar. Kemudian, ulangi proses verifikasi. Jika Anda memiliki penginstal terverifikasi, Anda dapat melanjutkan.

Untuk menginstal composersecara global, gunakan perintah berikut yang akan mengunduh dan menginstal Komposer sebagai perintah seluruh sistem bernama composer, di bawah /usr/local/bin:

```bash
sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

Anda akan melihat output berikut:

```bash
Output
All settings correct for using Composer
Downloading...

Composer (version 2.2.9) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer
```

Untuk menguji instalasi Anda, jalankan:

```bash
composer
```

```bash
Output
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version Composer version 2.2.9 2022-03-15 22:13:37
Usage:
  command [options] [arguments]

Options:
  -h, --help                     Display this help message
  -q, --quiet                    Do not output any message
  -V, --version                  Display this application version
      --ansi                     Force ANSI output
      --no-ansi                  Disable ANSI output
  -n, --no-interaction           Do not ask any interactive question
      --profile                  Display timing and memory usage information
      --no-plugins               Whether to disable plugins.
  -d, --working-dir=WORKING-DIR  If specified, use the given directory as working directory.
      --no-cache                 Prevent use of the cache
  -v|vv|vvv, --verbose           Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
...
```

## Menginstal MariaDB Server

Jalankan perintah berikut ini untuk menginstall MySQL/MariaDB

```sh
apt install mariadb-server -y
```

Secara default, pengguna root MariaDB tidak memiliki kata sandi, jadi Anda perlu menyetel kata sandi untuk pengguna root MariaDB.

Anda dapat mengaturnya dengan perintah berikut:

```sh
mysql_secure_installation
```

Jawab semua pertanyaan, seperti yang ditunjukkan di bawah ini:

```sh
Enter current password for root (enter for none): # Berikan kata sandi pengguna root Anda 
Switch to unix_socket authentication [Y/n] # n
Change the root password? [Y/n] # Y
New password:
Re-enter new password:
Remove anonymous users? [Y/n] # Y
Disallow root login remotely? [Y/n] # Y
Remove test database and access to it? [Y/n] # Y
Reload privilege tables now? [Y/n] # Y
```

Untuk memulai, masuk ke konsol MariaDB/MySQL:

```bash
mysql -u root -p
```
