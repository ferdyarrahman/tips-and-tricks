# INSTALL DAN KONFIGURSAI MARIADB GALERA CLUSTER

Membangun cluster MariaDB/MySQL kita bisa menggunakan Galera, Galera sudah termasuk dalam paket instalasi MariaDB di Ubuntu 20.04 jadi tidak perlu menginstall paket tambahan. Beberapa fitur Galera cluster

- True Multi-master, Active-Active Cluster Read and write to any node at any time.
- Synchronous Replication No slave lag, no data is lost at node crash.
- Tightly Coupled All nodes hold the same state. No diverged data between nodes allowed.
- Multi-threaded Slave For better performance. For any workload.
- No Master-Slave Failover Operations or Use of VIP.
- Hot Standby No downtime during failover (since there is no failover).
- Automatic Node Provisioning No need to manually back up the database and copy it to the new node.
- Supports InnoDB.
- Transparent to Applications Required no (or minimal changes) to the application.
- No Read and Write Splitting Needed.
- Easy to Use and Deploy

server yang digunakan:

|NAME SERVER|IP|HOSTNAME|DESCRIPTIONS|
|--|--|--|--|
|TDACI-HAPX-01|xxx.xx.x.196|DB-HAPROXY|HaProxy Galera Cluster|
|TDACI-DB-DEV-01|xxx.xx.x.191|DB-CLUSTER-1|Cluster NODE 1 (main)|
|TDACI-DB-ADEV-01|xxx.xx.x.192|DB-CLUSTER-2|Cluster NODE 2|
|TDACI-DB-PDEV-01|xxx.xx.x.193|DB-CLUSTER-3|Cluster NODE 3|

## Update System

Update system anda ke update paling baru yang tersedia, jalankan di semua server

```bash
apt update; apt upgrade -y
```

## Setting Hostname Server

Agar masing-masing server mudah dikenali, setting hostname masing-masing server

```bash
# server HaProxy
hostnamectl set-hostname --static DB-HAPROXY
# server 1
hostnamectl set-hostname --static DB-CLUSTER-1
# server 2
hostnamectl set-hostname --static DB-CLUSTER-2
# server 3
hostnamectl set-hostname --static DB-CLUSTER-3
```

di semua jalankan perintah ini untuk menambah mapping IP tersebut ke hostname

```bash
echo 'xxx.xx.x.196  DB-HAPROXY
xxx.xx.x.191  DB-CLUSTER-1
xxx.xx.x.192  DB-CLUSTER-2
xxx.xx.x.193  DB-CLUSTER-3' >> /etc/hosts
```

matikan services apparmor

```bash
systemctl stop apparmor
systemctl disable apparmor
```

ubah file permission `/tmp` agar bisa ditulis oleh MariaDB.

```bash
chmod 777 /tmp
```

## Install MariaDB Server

Di ketiga server tersebut jalankan perintah berikut ini untuk menginstall MySQL/MariaDB 10.3

```bash
apt install mariadb-server -y
```

di Ubuntu services MariaDB otomatis dijalankan, karena kita masih butuh untuk konfigurasinya, matikan service nya terlebih dahulu

```bash
systemctl stop mariadb
```

Untuk konfigurasi cluster berikut ini sesuaikan IP address dengan IP Server database yang anda miliki.

## Server Database 1 : DB-CLUSTER-1

Konfigurasi Galera cluster akan disimpan di `/etc/mysql/mariadb.conf.d/50-galera.cnf`, buat file tersebut lalu isi dengan

```bash
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=xxx.xx.x.191

# Galera Provider Configuration

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

wsrep_cluster_name="dbcluster"
wsrep_cluster_address="gcomm://xxx.xx.x.191,xxx.xx.x.192,xxx.xx.x.193"

wsrep_sst_method=rsync

wsrep_node_address="xxx.xx.x.191"
wsrep_node_name="DB-CLUSTER-1"
```

untuk binding IP address kita setting di `50-galera.cnf`, jadi bind di `50-server.cnf` dinonaktifkan

```bash
sed -i 's/bind-address/#bind-address/g'  /etc/mysql/mariadb.conf.d/50-server.cnf
```

Cluster database ini akan dijalankan dari **DB-CLUSTER-1** dengan perintah galera_new_cluster. Setelah menjalankan perintah tersebut cek status cluster anda.

```bash
mysql  -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

outputnya

```bash
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
```

karena kita baru menjalankan 1 server cluster, jadi hanya muncul 1 di ukuran clusternya.

Buat user `haproxy`, user ini nanti dibuat untuk mengecek status database.

```sql
CREATE USER 'haproxy'@'xxx.xx.x.196';
```

ganti `xxx.xx.x.196` dengan IP HaProxy.

## Server Database 2 : DB-CLUSTER-2

Konfigurasi Galera cluster akan disimpan di `/etc/mysql/mariadb.conf.d/50-galera.cnf`, buat file tersebut lalu isi dengan

```bash
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=xxx.xx.x.192

# Galera Provider Configuration

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

wsrep_cluster_name="dbcluster"
wsrep_cluster_address="gcomm://xxx.xx.x.191,xxx.xx.x.192,xxx.xx.x.193"

wsrep_sst_method=rsync

wsrep_node_address="xxx.xx.x.192"
wsrep_node_name="DB-CLUSTER-2"
```

untuk binding IP address kita setting di 50-galera.cnf, jadi bind di 50-server.cnf dinonaktifkan

```bash
sed -i 's/bind-address/#bind-address/g'  /etc/mysql/mariadb.conf.d/50-server.cnf
```

Jalankan services mariadb

```bash
systemctl start mariadb
```

Setelah menjalankan perintah tersebut cek status cluster anda.

```bash
mysql  -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

outputnya

```bash
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+
```

sekarang sudah keliatan ukuran cluster menjadi 2.

## Server Database 3 : DB-CLUSTER-3

Konfigurasi Galera cluster akan disimpan di `/etc/mysql/mariadb.conf.d/50-galera.cnf`, buat file tersebut lalu isi dengan

```bash
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=xxx.xx.x.193

# Galera Provider Configuration

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

wsrep_cluster_name="dbcluster"
wsrep_cluster_address="gcomm://xxx.xx.x.191,xxx.xx.x.192,xxx.xx.x.193"

wsrep_sst_method=rsync

wsrep_node_address="xxx.xx.x.193"
wsrep_node_name="DB-CLUSTER-3"
```

untuk binding IP address kita setting di `50-galera.cnf`, jadi bind di `50-server.cnf` dinonaktifkan

```bash
sed -i 's/bind-address/#bind-address/g'  /etc/mysql/mariadb.conf.d/50-server.cnf
```

Jalankan services mariadb

```bash
systemctl start mariadb
```

Setelah menjalankan perintah tersebut cek status cluster anda.

```bash
mysql  -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

outputnya

```bash
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```

ukuran cluster sudah menjadi 3, sesuai dengan jumlah server yang kita alokasikan.

## Testing Query Cluster

Untuk mengecek apakah cluster kita sudah bisa saling sinkronisasi data, buat database baru untuk testing. Login ke mysql/mariadb di CLUSTER-DB-1

```sql
mysql
```

lalu eksekusi query dibawah ini

```sql
CREATE DATABASE dbpesan;
CREATE TABLE dbpesan.pesan_singkat ( `id` INT NOT NULL AUTO_INCREMENT , `pesan` TEXT NOT NULL , PRIMARY KEY (`id`));
INSERT INTO dbpesan.pesan_singkat (`id`, `pesan`) VALUES (1, 'pesan pertama');
INSERT INTO dbpesan.pesan_singkat (`id`, `pesan`) VALUES (2, 'pesan kedua');
INSERT INTO dbpesan.pesan_singkat (`id`, `pesan`) VALUES (3, 'pesan ketiga');
```

dari ketiga server cek user hasil querynya

```sql
SELECT * FROM dbpesan.pesan_singkat;
```

keluaran query diatas

```sql
+----+---------------+
| id | pesan         |
+----+---------------+
|  1 | pesan pertama |
|  2 | pesan kedua   |
|  3 | pesan ketiga  |
+----+---------------+
3 rows in set (0.001 sec)
```

## HaProxy Galera Cluster

Sampai disini kita belum bisa membagi rata akses ke cluster database yang telah kita buat, untuk itu kita akan menggunakan HaProxy sebagai load balancer

Install HaProxy

```bash
apt install haproxy -y
```

Di konfigurasi HaProxy `/etc/haproxy/haproxy.cfg` tambahkan dibagian paling bawah

```bash
# statistik

listen stats
    bind :8080
    mode http
    stats enable
    stats hide-version
    stats realm Haproxy\ Statistics
    stats uri /
    stats auth jaranguda:WRmfp7Csprzq4NMLJbJsrhxLjPcmtX

listen galera_cluster
    bind xxx.xx.x.196:3306
    balance source
    mode tcp
    option tcpka
    option mysql-check user haproxy
    option tcplog

    server DB-CLUSTER-1 DB-CLUSTER-1:3306  check weight 1
    server DB-CLUSTER-2 DB-CLUSTER-2:3306  check weight 1
    server DB-CLUSTER-3 DB-CLUSTER-3:3306  check weight 1
```

`xxx.xx.x.196` ganti dengan IP HaProxy.

aktifkan haproxy waktu booting

```bash
systemctl enable haproxy
```

jalankan service haproxy

```bash
systemctl restart haproxy
```

bila anda membuat user baru di cluster mariadb tambahkan IP HaProxy untuk hostnamenya, contoh

```sql
CREATE DATABASE usr_pesanan
GRANT ALL PRIVILEGES ON userwp.* TO "usr_pesanan"@"xxx.xx.x.196" IDENTIFIED BY "pxwLzqdpH7";
```

jadi nanti aplikasi akan mengakses database lewat Haproxy.
