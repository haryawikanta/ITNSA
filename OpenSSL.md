Berikut dokumentasi sementara yang rapi dan bisa kamu pakai untuk laporan / catatan ujian.

---

# 📄 DOKUMENTASI SEMENTARA

## Implementasi Certificate Authority (OpenSSL) & HTTPS Apache

---

# 1. 🎯 Tujuan

Membuat infrastruktur **Certificate Authority (CA)** menggunakan OpenSSL untuk:

* Membuat Root CA
* Generate sertifikat server
* Sign sertifikat menggunakan CA
* Mengaktifkan HTTPS pada Apache
* Testing koneksi SSL/TLS

---

# 2. 📦 Instalasi OpenSSL

Pastikan OpenSSL sudah terpasang:

```bash
apt install openssl -y
```

---

# 3. 🏗️ Membuat Root CA

## 3.1 Generate Private Key + Self-Signed CA Certificate

```bash
openssl req -x509 -new -nodes -newkey rsa:4096 \
-keyout /root/ca/private/ca.key \
-out /root/ca/certs/ca.crt \
-days 3650 \
-subj "/C=ID/ST=West Java/L=Depok/O=ITNSA/OU=CA/CN=ITNSA-ROOT-CA"
```

## 3.2 Verifikasi CA

```bash
openssl x509 -in /root/ca/certs/ca.crt -text -noout
```

Hasil yang benar:

* Subject = ITNSA-ROOT-CA
* Issuer = ITNSA-ROOT-CA (self-signed root CA)

---

# 4. 🌐 Membuat Certificate Server

## 4.1 Generate Private Key + CSR

```bash
openssl req -new -newkey rsa:2048 -nodes \
-keyout www.harya-nsa.com.key \
-out www.harya-nsa.com.csr \
-subj "/C=ID/ST=West Java/L=Depok/O=ITNSA/CN=www.harya-nsa.com"
```

---

## 4.2 Sign Certificate menggunakan CA

```bash
openssl x509 -req \
-in www.harya-nsa.com.csr \
-CA /root/ca/certs/ca.crt \
-CAkey /root/ca/private/ca.key \
-CAcreateserial \
-out /root/ca/certs/www.harya-nsa.com.crt \
-days 365 -sha256
```

---

## 4.3 Verifikasi Certificate Server

```bash
openssl x509 -in /root/ca/certs/www.harya-nsa.com.crt -text -noout
```

Hasil yang benar:

* Subject: [www.harya-nsa.com](http://www.harya-nsa.com)
* Issuer: ITNSA-ROOT-CA

---

# 5. 🌐 Konfigurasi Apache HTTPS

Edit file:

```bash
/etc/apache2/sites-enabled/default-ssl.conf
```

Tambahkan:

```apache
SSLEngine on

SSLCertificateFile /root/ca/certs/www.harya-nsa.com.crt
SSLCertificateKeyFile /root/www.harya-nsa.com.key
SSLCACertificateFile /root/ca/certs/ca.crt
```

---

# 6. 🔄 Restart Apache

```bash
systemctl restart apache2
```

---

# 7. 🧪 Testing SSL

## 7.1 Test dengan OpenSSL

```bash
openssl s_client -connect localhost:443 -showcerts
```

Hasil sukses:

* Certificate chain muncul
* Verify return code: 0 (ok)

---

## 7.2 Test via Browser

Akses:

```
https://www.harya-nsa.com
```

---

# 8. 🔐 Instalasi CA di Windows

## 8.1 Copy CA ke Windows

```bash
scp root@server:/root/ca/certs/ca.crt .
```

## 8.2 Install ke Trusted Root

```cmd
certutil -addstore Root ca.crt
```

---

# 9. ⚠️ Troubleshooting Umum

| Masalah                 | Penyebab                    |
| ----------------------- | --------------------------- |
| HTTPS merah             | CA belum trusted di Windows |
| Issuer = domain sendiri | Self-signed certificate     |
| Verify error 19         | CA chain tidak dipercaya    |
| Apache error SSL        | Path certificate salah      |

---

# 10. 🧠 Kesimpulan

Sistem berhasil dibuat dengan:

* Root CA (ITNSA-ROOT-CA)
* Server certificate signed by CA
* Apache HTTPS aktif
* SSL testing berhasil dengan OpenSSL

---

# 🚀 Status Akhir

✔ CA dibuat
✔ Certificate server signed
✔ Apache HTTPS aktif
✔ SSL handshake berhasil
⚠ Browser warning tergantung trust Windows CA store

---

Kalau kamu mau, aku bisa lanjutkan versi **rapi laporan PDF + diagram PKI + step-by-step untuk ujian ITNSA** biar tinggal print.
