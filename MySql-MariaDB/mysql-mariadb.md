# MySQL / MariaDB

Zona waktu lokal Anda mungkin berbeda dari zona waktu MySQL server Anda. Itu membuat interpretasi data dalam database Anda sangat sulit. Idealnya, zona waktu MySQL harus sama dengan milik Anda untuk menangani data dengan lebih efisien. Terdapat 3 cara untuk melakukan pengaturan zona waktu.

## Konfigurasi my.cnf
Konfigurasi timezone my.cnf biasanya diletakan di /etc/mysqld/ dibawah seksi [mysqld]. Tapi konfigurasi ini tidak bisa dilakukan secara langsung, karena harus memiliki hak akses root.

```sh
default_time_zone='+07:00'
```

untuk melihat konfigurasi bisa menggunakan query berikut:
