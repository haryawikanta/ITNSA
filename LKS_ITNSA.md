# Writeup Sementara ITNSA Debian 13 Full Lab

## Identitas Server

| Komponen   | Keterangan  |
| ---------- | ----------- |
| OS         | Debian 13   |
| Hostname   | linsrv1     |
| Domain     | itnsa.local |
| Web Server | Apache2     |
| SMTP       | Postfix     |
| IMAP/POP3  | Dovecot     |
| Webmail    | Roundcube   |
| Database   | MariaDB     |
| DNS        | Bind9       |
| SSH        | OpenSSH     |
| User Mail  | harya       |

---

# 1. Konfigurasi Network

## Cek Interface

```bash
ip a
```

---

## Konfigurasi Interface Static

```bash
nano /etc/network/interfaces
```

Contoh:

```conf
auto ens33
iface ens33 inet dhcp
```

---

## Restart Network

```bash
systemctl restart networking
```

---

## Verifikasi

```bash
ip a
ping 8.8.8.8
```

---

# 2. Repository Debian (OPSIONAL) Kalau Sudah Bisa Install Langsung Ini Tidak Perlu 

## Edit Repository

```bash
nano /etc/apt/sources.list
```

Isi:

```bash
deb http://deb.debian.org/debian trixie main contrib non-free-firmware
deb http://deb.debian.org/debian trixie-updates main contrib non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free-firmware
```

---

## Update Package

```bash
apt update && apt upgrade -y
```

---

# 3. Konfigurasi SSH

## Install SSH

```bash
apt install openssh-server -y
```

---

## Cek Service

```bash
systemctl status ssh
```

---

## Enable Root Login

```bash
nano /etc/ssh/sshd_config
```

Ubah:

```conf
PermitRootLogin yes
PasswordAuthentication yes
```

---

## Restart SSH

```bash
systemctl restart ssh
```

---

## Verifikasi

```bash
ss -tunlp | grep :22
```

---

# 4. Konfigurasi DNS Server (Bind9)

## Install Bind9

```bash
apt install bind9 bind9utils bind9-doc -y
```

---

## Edit named.conf.local

```bash
nano /etc/bind/named.conf.local
```

Tambahkan:

```conf
zone "itnsa.local" {
    type master;
    file "/etc/bind/db.itnsa.local";
};

zone "0.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```

---

## Forward Zone

```bash
cp /etc/bind/db.local /etc/bind/db.itnsa.local
nano /etc/bind/db.itnsa.local
```

Isi:

```conf
$TTL    604800
@       IN      SOA     itnsa.local. root.itnsa.local. (
                              2
                         604800
                          86400
                        2419200
                         604800 )
;
@       IN      NS      ns.itnsa.local.
ns      IN      A       192.168.0.149
mail    IN      A       192.168.0.149
www     IN      A       192.168.0.149
@       IN      MX 10   mail.itnsa.local.
```

---

## Reverse Zone

```bash
cp /etc/bind/db.127 /etc/bind/db.192
nano /etc/bind/db.192
```

Isi:

```conf
$TTL    604800
@       IN      SOA     itnsa.local. root.itnsa.local. (
                              2
                         604800
                          86400
                        2419200
                         604800 )
;
@       IN      NS      ns.itnsa.local.
149     IN      PTR     mail.itnsa.local.
```

---

## Restart Bind9

```bash
systemctl restart bind9
```

---

## Verifikasi DNS

```bash
named-checkconf
named-checkzone itnsa.local /etc/bind/db.itnsa.local

nslookup mail.itnsa.local
```

---

# 5. Konfigurasi Apache Web Server

## Install Apache

```bash
apt install apache2 -y
```

---

## Cek Service

```bash
systemctl status apache2
```

---

## Membuat Virtual Host

```bash
nano /etc/apache2/sites-available/web1.conf
```

Isi:

```apache
<VirtualHost *:80>

    ServerName mail.itnsa.local
    DocumentRoot /var/www/web1

</VirtualHost>
```

---

## Membuat Document Root

```bash
mkdir -p /var/www/web1
```

---

## Membuat Index

```bash
echo "WEB ITNSA" > /var/www/web1/index.html
```

---

## Enable Site

```bash
a2ensite web1.conf
```

---

## Disable Default Site

```bash
a2dissite 000-default.conf
```

---

## Restart Apache

```bash
systemctl restart apache2
```

---

# 6. Konfigurasi MariaDB

## Install MariaDB

```bash
apt install mariadb-server -y
```

---

## Enable Service

```bash
systemctl enable mariadb
systemctl restart mariadb
```

---

## Masuk Database

```bash
mysql
```

---

## Membuat Database Roundcube

```sql
CREATE DATABASE roundcube;

CREATE USER 'roundcube'@'localhost' IDENTIFIED BY '123';

GRANT ALL PRIVILEGES ON roundcube.* TO 'roundcube'@'localhost';

FLUSH PRIVILEGES;

EXIT;
```

---

# 7. Konfigurasi Mail Server (Postfix)

## Install Postfix

```bash
apt install postfix mailutils -y
```

---

## Konfigurasi Main

```bash
nano /etc/postfix/main.cf
```

Tambahkan dan pagar bagian ini:

```conf
#myhostname = linsrv1.localdomain

#mydestination = $myhostname, itnsa.local, linsrv1, localhost.localdomain, localhost

myhostname = mail.itnsa.local
mydomain = itnsa.local
myorigin = /etc/mailname
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
home_mailbox = Maildir/
```

---

## Restart Postfix

```bash
systemctl restart postfix
```

---

## Verifikasi SMTP

```bash
ss -tunlp | grep :25
```

---

# 8. Konfigurasi Dovecot

## Install Dovecot

```bash
apt install dovecot-core dovecot-imapd dovecot-pop3d dovecot-lmtpd -y
```

---

## Konfigurasi Maildir

```bash
nano /etc/dovecot/conf.d/10-mail.conf
```

Ubah:

```conf
mail_location = maildir:~/Maildir
```

---

## Konfigurasi Protocol

```bash
nano /etc/dovecot/dovecot.conf
```

Isi:

```conf
protocols = imap pop3 lmtp
```

---

## Konfigurasi Authentication

```bash
nano /etc/dovecot/conf.d/10-auth.conf
```

Isi:

```conf
auth_mechanisms = plain login
ssl = no
auth_allow_cleartext = yes
```

---

## Restart Dovecot

```bash
systemctl restart dovecot
```

---

## Verifikasi

```bash
ss -tunlp | grep dovecot
```

---

# 9. Konfigurasi TLS/SSL

## Generate SSL

```bash
openssl req -new -x509 -days 365 -nodes \
-out /etc/ssl/certs/mailcert.pem \
-keyout /etc/ssl/private/mailkey.pem
```

---

## Konfigurasi SSL Dovecot

```bash
nano /etc/dovecot/conf.d/10-ssl.conf
```

Isi:

```conf
ssl = yes

ssl_cert = </etc/ssl/certs/mailcert.pem
ssl_key = </etc/ssl/private/mailkey.pem
```

---

## Konfigurasi TLS Postfix

```bash
nano /etc/postfix/main.cf
```

Tambahkan:

```conf
smtpd_tls_cert_file=/etc/ssl/certs/mailcert.pem
smtpd_tls_key_file=/etc/ssl/private/mailkey.pem

smtpd_use_tls=yes
```

---

## Restart Service

```bash
systemctl restart postfix
systemctl restart dovecot
```

---

# 10. Konfigurasi User Mail

## Membuat User

```bash
adduser harya
```

---

## Membuat Maildir

```bash
su - harya
mkdir -p ~/Maildir/{cur,new,tmp}
```

---

# 11. Konfigurasi Alias Mail

## Edit Alias

```bash
nano /etc/aliases
```

Isi:

```conf
admin: harya
webmaster: harya
```

---

## Generate Alias

```bash
newaliases
```

---

# 12. Konfigurasi Roundcube

## Install Roundcube

```bash
apt install dovecot-imapd dovecot-pop3d dovecot-core -y
```

---

## Reconfigure Package

```bash
dpkg-reconfigure roundcube-core
```

Konfigurasi:

| Opsi               | Nilai     |
| ------------------ | --------- |
| Configure database | Yes       |
| Database type      | mysql     |
| Connection method  | tcp/ip    |
| Host               | localhost |
| Port               | 3306      |
| Database           | roundcube |
| User               | roundcube |
| Password           | 123       |

---

## Enable Apache Config

```bash
a2enconf roundcube
```

---

## Edit Roundcube Apache Config

```bash
nano /etc/apache2/conf-enabled/roundcube.conf
```

Ubah:

```apache
Alias /roundcube /var/lib/roundcube/public_html
```

---

## Restart Apache

```bash
systemctl restart apache2
```

---

## Konfigurasi SMTP Roundcube

```bash
nano /etc/roundcube/config.inc.php
```

Ubah:

```php
$config['imap_host'] = ["localhost:143"];

$config['smtp_host'] = 'localhost:25';

$config['smtp_user'] = '';

$config['smtp_pass'] = '';
```

---

# 13. Pengujian Mail Server

## Test SMTP CLI

```bash
echo "HELLO ITNSA" | mail -s "TEST" harya
```

---

## Test TLS

```bash
openssl s_client -connect localhost:993
```

---

## Test GUI

Akses:

```text
http://192.168.0.149/roundcube
```

Login:

| Field    | Isi            |
| -------- | -------------- |
| Username | sesuai yang anda isi          |
| Password | password Linux |

---

# 14. Service yang Berhasil

| Service       | Status |
| ------------- | ------ |
| Network       | ✅      |
| SSH           | ✅      |
| DNS Bind9     | ✅      |
| Apache2       | ✅      |
| MariaDB       | ✅      |
| SMTP Postfix  | ✅      |
| IMAP Dovecot  | ✅      |
| TLS/SSL       | ✅      |
| Roundcube     | ✅      |
| Mail Delivery | ✅      |
| Webmail GUI   | ✅      |

---

# 15. Port Service

| Service | Port |
| ------- | ---- |
| SSH     | 22   |
| HTTP    | 80   |
| SMTP    | 25   |
| IMAP    | 143  |
| IMAPS   | 993  |
| DNS     | 53   |
| MariaDB | 3306 |

---

# 16. Kesimpulan

Lab Debian 13 berhasil dikonfigurasi sebagai:

* DNS Server
* Web Server
* Database Server
* SMTP Server
* IMAP Server
* Webmail Server
* TLS/SSL Mail Server

Semua service berhasil dijalankan dan diuji menggunakan CLI maupun GUI.
