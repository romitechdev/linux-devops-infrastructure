# Modul 03 - Nginx dan Reverse Proxy

## Pendahuluan
Nginx merupakan komponen penting pada banyak arsitektur aplikasi modern. Selain berfungsi sebagai web server, Nginx sangat populer sebagai reverse proxy yang menerima trafik dari internet lalu meneruskannya ke layanan backend internal. Pola ini meningkatkan keamanan, memusatkan terminasi TLS, serta mempermudah routing antar service.

Dalam konteks DevOps, Nginx sering menjadi gateway di depan aplikasi monolitik maupun microservices. Engineer perlu memahami konsep reverse proxy, struktur konfigurasi, optimasi performa, serta praktik hardening agar layanan stabil di produksi.

## Tujuan Pembelajaran
Setelah mempelajari modul ini, peserta mampu:

1. Menjelaskan konsep reverse proxy dan manfaatnya.
2. Memahami struktur konfigurasi Nginx (http, server, location, upstream).
3. Membuat konfigurasi reverse proxy untuk backend aplikasi.
4. Mengaktifkan HTTPS dengan sertifikat TLS.
5. Menerapkan best practice keamanan dan performa dasar.
6. Melakukan troubleshooting Nginx berbasis log dan validasi konfigurasi.

## Ruang Lingkup Materi
- Peran Nginx pada arsitektur modern.
- Alur request-response melalui reverse proxy.
- Header forwarding, timeout, dan buffering.
- Static file serving, gzip, caching singkat.
- TLS termination dengan Certbot.
- Logging dan troubleshooting operasional.

## Prasyarat
- Server Linux aktif dengan akses sudo.
- Aplikasi backend sudah berjalan (contoh Node.js di port 3000).
- Domain/subdomain mengarah ke IP server (untuk HTTPS).

## Konsep Dasar
### 1. Apa itu Reverse Proxy

Reverse proxy adalah server perantara di depan aplikasi. User mengakses Nginx, lalu Nginx meneruskan request ke backend (misalnya Node.js di port 3000).

Manfaat utama untuk pemula:

- Domain publik tetap rapi (cukup 80/443).
- Backend bisa disembunyikan di localhost/private network.
- Konfigurasi HTTPS cukup di satu tempat.

### 2. Struktur Dasar Konfigurasi Nginx

Bagian yang paling sering dipakai:

- `server`: aturan untuk satu domain.
- `location`: aturan untuk path tertentu.
- `proxy_pass`: tujuan backend.

### 3. Header Forwarding yang Wajib Dipahami

Saat request diteruskan, backend butuh informasi asal request. Karena itu, header berikut hampir selalu dipasang:

- `Host` untuk nama domain asli.
- `X-Real-IP` untuk IP client.
- `X-Forwarded-For` untuk rantai IP proxy.
- `X-Forwarded-Proto` untuk protocol `http`/`https`.

### 4. Validasi Konfigurasi Sebelum Reload

Aturan paling penting untuk pemula: setiap selesai edit konfigurasi, jalankan `nginx -t` dulu. Jika valid, baru `reload` agar layanan tidak putus.

### 5. Log untuk Troubleshooting

Nginx punya dua log penting:

- `access.log` untuk request masuk.
- `error.log` untuk error konfigurasi atau koneksi backend.

Dua log ini adalah titik pertama saat terjadi masalah 502, 404 tidak sesuai, atau domain salah arah.

### 6. Alur Cek Cepat untuk Pemula

Saat layanan belum bisa diakses, lakukan cek berurutan ini:

1. Pastikan service Nginx aktif.
2. Pastikan backend aplikasi aktif.
3. Pastikan domain mengarah ke IP server.
4. Validasi konfigurasi dengan `nginx -t`.
5. Baca `error.log` untuk menemukan penyebab utama.

## Cara Kerja
1. Klien mengakses domain aplikasi pada port 80/443.
2. Nginx menerima request dan mencocokkan server_name.
3. Nginx mengevaluasi location yang sesuai.
4. Request diteruskan ke backend melalui proxy_pass.
5. Respons backend diterima Nginx dan dikirim kembali ke klien.
6. Akses dicatat di access.log; kesalahan tercatat di error.log.

## Contoh Implementasi
Skenario: backend Node.js di 127.0.0.1:3000 dipublikasikan sebagai app.example.com.

### A. Instalasi Nginx

~~~bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `apt install -y nginx` memasang Nginx.
- `systemctl enable nginx` membuat service otomatis hidup saat boot.
- `systemctl start nginx` langsung menjalankan service tanpa reboot.

### B. Konfigurasi reverse proxy

Buat file: /etc/nginx/sites-available/app.example.com

~~~nginx
upstream app_backend {
    server 127.0.0.1:3000;
    keepalive 32;
}

server {
    listen 80;
    server_name app.example.com;

    client_max_body_size 10m;

    location / {
        proxy_pass http://app_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";
        proxy_read_timeout 60s;
        proxy_connect_timeout 10s;
    }

    location /health {
        access_log off;
        return 200 "ok\n";
    }
}
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `upstream app_backend` mendefinisikan alamat backend aplikasi.
- `server_name app.example.com` mengikat konfigurasi ke domain tertentu.
- `location /` meneruskan semua request utama ke backend via `proxy_pass`.
- `proxy_set_header ...` memastikan backend menerima informasi host dan IP asli.
- `location /health` menyediakan endpoint cek cepat untuk monitoring.

### C. Aktivasi konfigurasi

~~~bash
sudo ln -s /etc/nginx/sites-available/app.example.com /etc/nginx/sites-enabled/app.example.com
sudo nginx -t
sudo systemctl reload nginx
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `ln -s` mengaktifkan file site dengan symlink.
- `nginx -t` mengecek sintaks konfigurasi.
- `reload` memuat ulang konfigurasi tanpa menghentikan koneksi aktif.

### D. Uji endpoint setelah reverse proxy aktif

~~~bash
curl -I http://app.example.com
curl -s http://app.example.com/health
sudo tail -n 50 /var/log/nginx/access.log
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `curl -I` mengecek status HTTP dan header respons.
- Endpoint `/health` membantu verifikasi jalur request ke backend.
- `access.log` dipakai memastikan request benar benar masuk ke Nginx.

## Contoh Perintah dan Konfigurasi
### 1. HTTPS dengan Certbot

~~~bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d app.example.com --redirect -m admin@example.com --agree-tos -n
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `certbot` meminta sertifikat TLS dari Let's Encrypt.
- Opsi `--redirect` otomatis mengarahkan HTTP ke HTTPS.

### 2. Verifikasi sertifikat dan jadwal renewal

~~~bash
sudo certbot certificates
systemctl list-timers | grep certbot
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Perintah pertama melihat detail sertifikat aktif.
- Perintah kedua memastikan timer renewal otomatis tersedia.

### 3. Konfigurasi keamanan tambahan

~~~nginx
server_tokens off;
add_header X-Frame-Options SAMEORIGIN always;
add_header X-Content-Type-Options nosniff always;
add_header Referrer-Policy strict-origin-when-cross-origin always;
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `server_tokens off` menyembunyikan versi Nginx dari respons.
- Header tambahan membantu mengurangi risiko serangan browser-side.

### 4. Static file caching (contoh)

~~~nginx
location /assets/ {
    alias /var/www/app/assets/;
    expires 7d;
    add_header Cache-Control "public, max-age=604800";
}
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `expires 7d` membuat browser menyimpan aset statis selama 7 hari.
- Caching mengurangi beban backend dan mempercepat loading halaman.

### 5. Rate limiting sederhana

~~~nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    proxy_pass http://app_backend;
}
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `rate=10r/s` membatasi laju request per IP.
- `burst=20` memberi toleransi lonjakan singkat sebelum request ditolak.

### 6. Debug cepat konfigurasi aktif

~~~bash
sudo nginx -T | less
sudo systemctl status nginx --no-pager
sudo tail -n 100 /var/log/nginx/error.log
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `nginx -T` menampilkan seluruh konfigurasi final (termasuk file include).
- `status` dan `error.log` dipakai untuk memastikan service sehat setelah perubahan.

## Best Practice Singkat
- Jalankan nginx -t sebelum reload setiap perubahan.
- Gunakan HTTPS wajib di layanan publik.
- Simpan konfigurasi per domain agar maintainable.
- Batasi exposure backend ke localhost atau private network.
- Terapkan request size limit dan timeout sesuai beban.
- Gunakan access log untuk analisis trafik dan keamanan.

## Ringkasan
Nginx reverse proxy memegang peran strategis dalam pengelolaan trafik aplikasi modern. Dengan konfigurasi yang benar, Nginx meningkatkan keamanan, fleksibilitas routing, dan kemudahan operasional, sekaligus menjadi fondasi untuk scaling serta strategi deployment yang lebih matang.

## Referensi
1. Nginx Official Docs: https://nginx.org/en/docs/
2. Nginx Admin Guide: https://docs.nginx.com/nginx/admin-guide/
3. Certbot Documentation: https://eff.org/https-ecosystem/certbot/
4. Mozilla SSL Configuration Generator: https://ssl-config.mozilla.org/
5. RFC 7239 Forwarded HTTP Extension: https://www.rfc-editor.org/rfc/rfc7239
