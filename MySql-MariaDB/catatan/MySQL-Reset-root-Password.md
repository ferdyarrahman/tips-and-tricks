# Reset root Password in MySQL

## If you in skip-grant-tables mode

### mysqld_safe

```mysql
UPDATE mysql.user SET authentication_string=null WHERE User='root';
FLUSH PRIVILEGES;
exit;
```

and then, in terminal:

```mysql
mysql -u root
```

### mysql

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'yourpasswd';
```

## not in skip-grant-tables mode

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'yourpasswd';
```

## Changed your mysql default authentication plugin

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpasswd'; 
```
