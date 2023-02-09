# Setting konfigurasi TimeZone GMT +7 (Indonesian)

Zona waktu lokal Anda mungkin berbeda dari zona waktu MySQL server Anda. Itu membuat interpretasi data dalam database Anda sangat sulit. Idealnya, zona waktu MySQL harus sama dengan milik Anda untuk menangani data dengan lebih efisien. Terdapat 3 cara untuk melakukan pengaturan zona waktu.

## Konfigurasi my.cnf

Konfigurasi _timezone_ my.cnf biasanya diletakan di `/etc/mysqld/` dibawah seksi [mysqld]. Tapi konfigurasi ini tidak bisa dilakukan secara langsung, karena harus memiliki hak akses `root`.

```sh
default_time_zone='+07:00'
```

untuk melihat konfigurasi bisa menggunakan _query_ berikut:

```mysql
SELECT @@global.time_zone;
```

## Konfigurasi menggunakan skrip SQL

merubah _timezone_ melalui _query_ dan menggunakan akses `root`.

```mysql
SET GLOBAL time_zone = '+07:00';
SET @@global.time_zone='+07:00';
SET GLOBAL time_zone = 'Asia/Jakarta';
```

## Konfigurasi menggunakan _variable session_

_Variable session_ dapat diakses oleh semua user MySQL tapi hanya berlaku pada saat _session_ belum habis. Untuk melakukannya dapat mengunakan:

```mysql
SET time_zone = "+07:00";
SET @@session.time_zone = "+07:00";
SET time_zone = 'Asia/Jakarta';
```

untuk memeriksa hasil konfigurasinya. jalankan _query_ berikut ini:

```mysql
SELECT @@session.time_zone;
```
