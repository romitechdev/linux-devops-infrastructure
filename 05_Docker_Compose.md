# Modul 05 - Docker Compose

## Pendahuluan
Aplikasi modern jarang berdiri sendiri. Umumnya terdapat beberapa komponen yang saling bergantung, seperti web service, database, cache, worker, dan message broker. Menjalankan tiap container dengan command manual mudah menimbulkan kesalahan dan inkonsistensi. Docker Compose menyelesaikan masalah ini dengan pendekatan deklaratif melalui satu berkas konfigurasi.

Dalam workflow DevOps, Compose berperan penting untuk local environment yang konsisten, integration testing, hingga deployment sederhana pada VPS.

## Tujuan Pembelajaran
Setelah mempelajari modul ini, peserta mampu:

1. Menjelaskan fungsi Docker Compose pada arsitektur multi-service.
2. Menulis file compose yang terstruktur untuk service aplikasi dan dependency.
3. Mengelola lifecycle stack menggunakan command Compose.
4. Mengonfigurasi network, volume, environment, dan healthcheck.
5. Menerapkan praktik pemisahan konfigurasi antar environment.

## Ruang Lingkup Materi
- Konsep service dan dependency.
- Compose network dan service discovery.
- Persistensi data dengan named volume.
- Penggunaan env file.
- Healthcheck dan restart policy.
- Debugging stack multi-container.

## Prasyarat
- Docker Engine dan Docker Compose plugin sudah terpasang.
- Dasar penggunaan Docker command.
- Proyek aplikasi sederhana yang membutuhkan database.

## Konsep Dasar
### 1. Apa itu Docker Compose

Docker Compose adalah cara menjalankan beberapa container sekaligus menggunakan satu file YAML. Ini memudahkan pemula karena semua service didefinisikan di satu tempat.

### 2. Service dalam Compose

Setiap service adalah satu container, misalnya `web`, `db`, dan `redis`. Service bisa saling terhubung lewat nama service sebagai hostname.

### 3. Volume untuk Data

Database perlu volume supaya data tidak hilang saat container restart atau di-recreate.

### 4. Environment Variable

Variabel konfigurasi seperti URL database sebaiknya ditaruh di file `.env`, bukan ditulis keras di kode aplikasi.

### 5. depends_on dan healthcheck

`depends_on` membantu urutan startup, sedangkan `healthcheck` membantu memastikan service benar benar siap.

### 6. Kapan Pakai Compose

Untuk pemula, Compose cocok digunakan ketika aplikasi punya lebih dari satu komponen, misalnya aplikasi web + database + cache, dan semua komponen ingin dijalankan dengan satu command.

### 7. Alur Aman Menjalankan Stack

Urutan dasar yang disarankan:

1. Validasi file Compose.
2. Jalankan stack.
3. Cek status container.
4. Cek log aplikasi.
5. Uji endpoint aplikasi.

## Cara Kerja
1. Engineer menulis compose.yaml.
2. Docker Compose membaca definisi service.
3. Network dan volume dibuat otomatis jika belum ada.
4. Semua service dijalankan dengan docker compose up.
5. Log, status, dan command operasional dikelola dari satu antarmuka CLI.
6. Stack dihentikan atau dihapus saat selesai.

## Contoh Implementasi
Skenario: API Node.js + PostgreSQL + Redis.

### compose.yaml

~~~yaml
services:
  web:
    image: node:20-alpine
    working_dir: /app
    command: sh -c "npm ci && npm run start"
    volumes:
      - ./web:/app
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: appsecret
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redisdata:/data
    restart: unless-stopped

volumes:
  pgdata:
  redisdata:
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Service `web` menjalankan aplikasi Node.js dan expose port `3000`.
- Service `db` memakai PostgreSQL dengan volume `pgdata` agar data tetap ada.
- Service `redis` dipakai cache/queue sederhana dengan volume `redisdata`.
- `depends_on` pada `web` menunggu `db` sehat sebelum start penuh.
- `restart: unless-stopped` membuat service otomatis hidup lagi saat crash atau reboot host.

### File .env contoh

~~~text
NODE_ENV=production
DATABASE_URL=postgres://appuser:appsecret@db:5432/appdb
REDIS_URL=redis://redis:6379
PORT=3000
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `DATABASE_URL` menunjuk service `db` (bukan localhost).
- `REDIS_URL` menunjuk service `redis`.
- Pemisahan konfigurasi ini memudahkan pindah environment.

### Verifikasi konfigurasi sebelum dijalankan

~~~bash
docker compose config
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Menampilkan konfigurasi final setelah menggabungkan `compose.yaml` dan `.env`.
- Berguna untuk memeriksa typo variabel atau service sebelum stack dijalankan.

## Contoh Perintah dan Konfigurasi
### A. Menjalankan stack

~~~bash
docker compose up -d
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Membuat network/volume otomatis lalu menjalankan semua service di background.

### B. Verifikasi service

~~~bash
docker compose ps
docker compose logs -f web
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `ps` mengecek status semua container di stack.
- `logs -f web` melihat log aplikasi web secara real-time.

### C. Eksekusi command di service

~~~bash
docker compose exec db psql -U appuser -d appdb -c "SELECT 1;"
docker compose exec web sh
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Command pertama menguji koneksi database dari dalam container `db`.
- Command kedua membuka shell di container `web` untuk debugging.

### D. Scale service web

~~~bash
docker compose up -d --scale web=2
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Menjalankan dua instance service `web` untuk uji skala dasar.

### E. Hentikan stack

~~~bash
docker compose down
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Menghentikan dan menghapus container/network stack, tetapi volume data tetap ada.

### F. Hapus stack beserta volume

~~~bash
docker compose down -v
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Sama seperti `down`, tetapi sekaligus menghapus volume. Gunakan hati hati karena data bisa hilang.

### G. Backup sederhana volume database

~~~bash
docker compose exec db pg_dump -U appuser -d appdb > backup_appdb.sql
ls -lh backup_appdb.sql
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Membuat dump database dari service `db` ke file lokal.
- Langkah ini cocok sebagai backup manual sebelum perubahan besar.

## Best Practice Singkat
- Definisikan healthcheck untuk service kritis.
- Gunakan env_file atau secret manager untuk data sensitif.
- Gunakan named volume untuk data stateful.
- Batasi port exposure ke host hanya saat diperlukan.
- Gunakan image tag versi spesifik, bukan latest di produksi.
- Review output docker compose config sebelum deploy.

## Ringkasan
Docker Compose menyederhanakan manajemen aplikasi multi-container melalui konfigurasi deklaratif. Dengan Compose, tim dapat menjalankan environment yang konsisten, mempercepat proses integrasi, dan mengurangi kesalahan operasional saat berpindah antar lingkungan.

## Referensi
1. Docker Compose Docs: https://docs.docker.com/compose/
2. Compose Specification: https://compose-spec.io/
3. PostgreSQL Docker Image: https://hub.docker.com/_/postgres
4. Redis Docker Image: https://hub.docker.com/_/redis
5. Twelve-Factor App: https://12factor.net/config
