# INSTALL DAN KONFIGURSAI MARIADB GALERA CLUSTER

Cluster database untuk meningkatkan ketersediaan database dengan mendistribusikan beban ke server yang berbeda. Jika ada server yang gagal maka server lain akan segera tersedia untuk melayani.
Membangun cluster MariaDB/MySQL kita bisa menggunakan **Galera Cluster**, **Galera Cluster** sudah termasuk dalam paket instalasi MariaDB di Ubuntu 20.04 jadi tidak perlu menginstall paket tambahan. Dengan **Galera Cluster** perubahan yang dilakukan pada satu node akan direplikasi ke semua node. berikut fitur-fitur yang ada di **Galera Cluster**:

- True Multi-master, Cluster Aktif-Aktif Membaca dan menulis ke node mana pun kapan saja.
- Synchronous Replication No slave lag, tidak ada data yang hilang saat node bermasalah.
- Tightly Coupled All nodes hold the same state. Tidak ada data yang berbeda antar node yang diperbolehkan.
- Multi-threaded Slave For better performance.
- No Master-Slave Failover Operations or Use of VIP.
- Hot Standby No downtime during failover (since there is no failover).
- Penyediaan Node Otomatis Tidak perlu membuat cadangan database secara manual dan menyalinnya ke node baru.
- Supports InnoDB.
- Transparent to Applications Required no (or minimal changes) to the application.
- No Read and Write Splitting Needed.
- Easy to Use and Deploy

server yang digunakan sebagai contoh:

|IP|HOSTNAME|DESCRIPTIONS|
|--|--|--|
|192.168.0.191|DB-CLUSTER-1|Cluster NODE 1 (main)|
|192.168.0.192|DB-CLUSTER-2|Cluster NODE 2|
|192.168.0.193|DB-CLUSTER-3|Cluster NODE 3|

## Update System

Update system Anda ke update paling baru yang tersedia, jalankan di semua server

```bash
apt update; apt upgrade -y
```

## Setting Hostname Server

Agar masing-masing server mudah dikenali, setting hostname masing-masing server

```bash
# server 1
hostnamectl set-hostname --static DB-CLUSTER-1
# server 2
hostnamectl set-hostname --static DB-CLUSTER-2
# server 3
hostnamectl set-hostname --static DB-CLUSTER-3
```

di semua jalankan perintah ini untuk menambah mapping IP tersebut ke hostname

```bash
echo '192.168.0.191  DB-CLUSTER-1
192.168.0.192  DB-CLUSTER-2
192.168.0.193  DB-CLUSTER-3' >> /etc/hosts
```

## Install MariaDB Server

Di ketiga server tersebut jalankan perintah berikut ini untuk menginstall MySQL/MariaDB

```bash
apt install mariadb-server -y
```

Secara default, pengguna root MariaDB tidak memiliki kata sandi, jadi Anda perlu menyetel kata sandi untuk pengguna root MariaDB.

Anda dapat mengaturnya dengan perintah berikut:

```bash
mysql_secure_installation
```

Jawab semua pertanyaan, seperti yang ditunjukkan di bawah ini:

```bash
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

di Ubuntu services MariaDB otomatis dijalankan, karena kita masih butuh untuk konfigurasinya, matikan service nya terlebih dahulu

```bash
systemctl stop mariadb
```

Silakan ulangi langkah di atas pada 3 server database.

## Konfigurasikan Setiap Server di Cluster

Pada titik ini, Anda telah menginstal dan mengkonfigurasi server MariaDB di setiap server. Selanjutnya, Anda perlu mengkonfigurasi cluster Galera untuk berkomunikasi antar server. Untuk melakukannya, Anda perlu membuat file konfigurasi umum di setiap server.

### Server Database 1 : DB-CLUSTER-1

Pertama, login ke server pertama dan buat file konfigurasi Galera dengan perintah berikut:

```bash
nano /etc/mysql/conf.d/galera.cnf
```

Tambahkan baris berikut:

```bash
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://192.168.0.191,192.168.0.192,192.168.0.193"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="192.168.0.191"
wsrep_node_name="DB-CLUSTER-1"
```

Simpan dan tutup file setelah Anda selesai. Selanjutnya, Anda dapat melanjutkan ke server kedua

### Server Database 2 : DB-CLUSTER-2

Selanjutnya login ke server kedua dan buat file konfigurasi Galera dengan perintah berikut:

```bash
nano /etc/mysql/conf.d/galera.cnf
```

Tambahkan baris berikut:

```bash
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://192.168.0.191,192.168.0.192,192.168.0.193"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="192.168.0.192"
wsrep_node_name="DB-CLUSTER-2"
```

Simpan dan tutup file setelah Anda selesai. Selanjutnya, Anda dapat melanjutkan ke server ketiga.

### Server Database 3 : DB-CLUSTER-3

Selanjutnya login ke server ketiga dan buat file konfigurasi Galera dengan perintah berikut:

```bash
nano /etc/mysql/conf.d/galera.cnf
```

Tambahkan baris berikut:

```bash
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://192.168.0.191,192.168.0.192,192.168.0.193"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="192.168.0.193"
wsrep_node_name="DB-CLUSTER-3"
```

Simpan dan tutup file setelah Anda selesai.

Pada titik ini, kami telah mengkonfigurasi ketiga server untuk berkomunikasi satu sama lain.

## Inisialisasi Cluster Galera

Inisialisasi cluster di node pertama dengan perintah berikut:

```bash
galera_new_cluster
```

Perintah di atas akan memulai cluster dan menambahkan server1 ke cluster.

Anda dapat memeriksanya dengan perintah berikut:

```sql
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
Enter password:
```

Berikan kata sandi root Anda dan tekan Enter. Anda akan melihat keluaran berikut:

```sql
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
```

Selanjutnya, buka server kedua dan mulai layanan MariaDB:

```bash
systemctl start mariadb
```

Selanjutnya, verifikasi ukuran cluster Anda dengan perintah berikut:

```sql
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
Enter password:
```

Berikan kata sandi root Anda dan tekan Enter. Anda akan melihat bahwa server kedua telah bergabung dengan cluster.

```sql
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+
```

Selanjutnya, buka server ketiga dan mulai layanan MariaDB:

```bash
systemctl start mariadb
```

Selanjutnya, verifikasi ukuran cluster Anda dengan perintah berikut:

```sql
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
Enter password:
```

Berikan kata sandi root Anda dan tekan Enter. Anda akan melihat bahwa server ketiga telah bergabung dengan cluster.

```sql
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```

## Uji Replikasi Cluster Galera

Cluster Galera Anda sekarang sudah aktif dan berjalan. Saatnya menguji dan melihat apakah replikasi berfungsi.

Untuk melakukannya, buat database di server pertama dan periksa apakah database tersebut telah direplikasi ke server lain.

Di server1, masuk ke shell MySQL dengan perintah berikut:

```bash
mysql -u root -p
```

Berikan kata sandi root Anda saat diminta, lalu buat database dengan perintah berikut:

```sql
create database testDB1;
create database testDB2;
```

Selanjutnya, keluar dari shell MySQL dengan perintah berikut:

```sql
exit;
```

Di server2, masuk ke shell MySQL dengan perintah berikut:

```bash
mysql -u root -p
```

Berikan kata sandi root Anda saat diminta dan periksa apakah databasenya ada.

```sql
show databases;
```

Anda harus mendapatkan hasil berikut:

```sql
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| testDB1            |
| testDB2            |
+--------------------+
```

Di server3, masuk ke shell MySQL dengan perintah berikut:

```bash
mysql -u root -p
```

Berikan kata sandi root Anda saat diminta dan periksa apakah databasenya ada.

```sql
show databases;
```

Anda harus mendapatkan hasil berikut:

```sql
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| testDB1            |
| testDB2            |
+--------------------+
```

Output di atas dengan jelas menunjukkan bahwa replikasi berfungsi dengan baik.
