# Modul 04 - Docker

## Pendahuluan
Docker adalah teknologi containerization yang merevolusi cara aplikasi dikembangkan, diuji, dan didistribusikan. Sebelum container populer, perbedaan konfigurasi lingkungan sering menyebabkan masalah klasik: berjalan di mesin developer tetapi gagal di staging atau production. Docker mengatasi persoalan ini dengan membungkus aplikasi beserta runtime dan dependensinya ke dalam image yang portabel.

Dalam praktik DevOps modern, Docker tidak hanya dipakai untuk local development, tetapi juga menjadi artefak utama pada pipeline CI/CD, strategi deployment, dan pengelolaan workload terisolasi.

## Tujuan Pembelajaran
Setelah mempelajari modul ini, peserta mampu:

1. Menjelaskan konsep dan manfaat Docker dalam siklus pengembangan software.
2. Membedakan virtual machine dan container secara teknis.
3. Memahami image lifecycle: build, tag, push, pull, run.
4. Menulis Dockerfile yang efisien dan aman.
5. Mengelola container, network, dan volume.
6. Menerapkan praktik hardening dasar pada image dan runtime.

## Ruang Lingkup Materi
- Arsitektur Docker Engine.
- Konsep image, layer, container, registry.
- Dockerfile dan strategi build.
- Runtime operation: logs, exec, inspect, resource limit.
- Data persistence dengan volume.
- Keamanan dasar image dan container.

## Prasyarat
- Linux server atau workstation dengan Docker Engine terpasang.
- Pemahaman dasar CLI Linux.
- Dasar pemrograman aplikasi web sederhana.

## Konsep Dasar
### 1. Container vs Virtual Machine

Container menjalankan aplikasi dengan berbagi kernel host, sedangkan virtual machine punya sistem operasi sendiri. Karena itu, container biasanya lebih ringan dan startup lebih cepat.

### 2. Image dan Container

Image adalah template aplikasi. Container adalah image yang sedang berjalan. Satu image bisa dipakai membuat banyak container.

### 3. Dockerfile

Dockerfile berisi langkah langkah membuat image, misalnya memilih base image, memasang dependency, menyalin source code, dan menentukan command saat container start.

### 4. Registry dan Tag

Registry adalah tempat menyimpan image. Tag (misalnya `1.0.0`) membantu tim tahu versi mana yang dipakai di server.

### 5. Volume dan Data

Data penting tidak boleh bergantung pada filesystem sementara container. Gunakan volume agar data tetap ada walau container dihapus lalu dibuat ulang.

### 6. Log Dasar Container

Troubleshooting awal di Docker biasanya dimulai dari `docker ps` untuk status dan `docker logs` untuk melihat error aplikasi.

### 7. Alur Kerja Docker untuk Pemula

Urutan belajar paling aman untuk pemula:

1. Buat aplikasi sederhana.
2. Tulis Dockerfile.
3. Build image.
4. Jalankan container lokal.
5. Uji endpoint aplikasi.
6. Push image jika hasil uji sudah stabil.

## Cara Kerja
1. Engineer menulis Dockerfile.
2. Docker build menghasilkan image.
3. Image diberi tag versi.
4. Image diuji sebagai container lokal.
5. Image dipush ke registry.
6. Environment target melakukan pull image versi yang sama.

Prinsip DevOps penting: build once, run anywhere.

## Contoh Implementasi
Skenario: containerisasi API Python Flask dengan Gunicorn.

### Struktur proyek

~~~text
flask-api/
  app.py
  requirements.txt
  Dockerfile
  .dockerignore
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Struktur ini minimal namun cukup untuk belajar build image end-to-end.
- Simpan Dockerfile di root project agar perintah `docker build .` bekerja langsung.

### app.py

~~~python
from flask import Flask

app = Flask(__name__)

@app.get("/health")
def health():
    return {"status": "ok"}, 200

@app.get("/")
def home():
    return {"message": "hello from docker"}, 200
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Endpoint `/health` dipakai untuk cek cepat apakah aplikasi hidup.
- Endpoint `/` hanya contoh respon sederhana untuk tes awal.
- Respon JSON memudahkan integrasi dengan service lain.

### requirements.txt

~~~text
flask==3.0.3
gunicorn==22.0.0
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `flask` adalah framework web.
- `gunicorn` adalah web server production-grade untuk menjalankan Flask app.

### .dockerignore

~~~text
__pycache__/
*.pyc
*.pyo
.git/
.env
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- File/folder ini tidak ikut dikirim ke proses build.
- Tujuannya membuat build lebih cepat dan image lebih kecil.

## Contoh Perintah dan Konfigurasi
### A. Dockerfile versi produksi dasar

~~~dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN addgroup --system app && adduser --system --ingroup app app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

USER app
EXPOSE 8000

CMD ["gunicorn", "-w", "2", "-k", "gthread", "-b", "0.0.0.0:8000", "app:app"]
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `FROM python:3.12-slim` memilih base image yang ringan.
- `WORKDIR /app` menetapkan direktori kerja di dalam container.
- `COPY requirements.txt` lalu `pip install` memasang dependency Python.
- `USER app` menjalankan proses sebagai user non-root agar lebih aman.
- `CMD ... app:app` menjalankan aplikasi Flask lewat Gunicorn di port 8000.

Catatan pemula:

- Urutan instruksi di Dockerfile memengaruhi cache build.
- Letakkan instruksi yang jarang berubah di atas agar build ulang lebih cepat.

### B. Build, run, dan verifikasi

~~~bash
docker build -t myorg/flask-api:1.0.0 .
docker run -d --name flask-api -p 8000:8000 myorg/flask-api:1.0.0
curl http://localhost:8000/health
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `docker build` membuat image dari Dockerfile.
- `docker run -d` menjalankan container di background.
- `-p 8000:8000` memetakan port host ke port container.
- `curl .../health` mengecek aplikasi merespons dengan benar.

### C. Operasional container

~~~bash
docker ps
docker logs -f flask-api
docker inspect flask-api
docker exec -it flask-api sh
docker stats flask-api
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `docker ps` melihat container aktif.
- `docker logs -f` mengikuti log secara real-time.
- `docker inspect` menampilkan detail konfigurasi container.
- `docker exec -it ... sh` masuk shell container untuk inspeksi manual.
- `docker stats` memantau penggunaan CPU/memori.

### D. Resource limit runtime

~~~bash
docker run -d --name flask-api-limited \
  --cpus="1.0" \
  --memory="512m" \
  -p 8001:8000 \
  myorg/flask-api:1.0.0
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `--cpus` dan `--memory` membatasi resource agar container tidak mengganggu service lain di host.

### F. Cleanup container dan image lokal

~~~bash
docker stop flask-api
docker rm flask-api
docker image ls
docker image prune -f
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `stop` menghentikan container yang sedang berjalan.
- `rm` menghapus container agar nama bisa dipakai lagi.
- `image prune -f` membersihkan image sisa yang tidak terpakai.

### E. Tag dan push ke registry

~~~bash
docker tag myorg/flask-api:1.0.0 ghcr.io/myorg/flask-api:1.0.0
docker push ghcr.io/myorg/flask-api:1.0.0
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `docker tag` membuat nama image sesuai format registry tujuan.
- `docker push` mengirim image agar bisa dipakai oleh server lain.

## Best Practice Singkat
- Gunakan image base minimal dan terpercaya.
- Jalankan aplikasi sebagai non-root user.
- Pin dependency dan image tag secara spesifik.
- Hindari menyimpan secret di image layer.
- Gunakan .dockerignore untuk menjaga konteks build tetap ramping.
- Tambahkan healthcheck untuk service kritis.

## Ringkasan
Docker mempermudah standarisasi runtime aplikasi lintas lingkungan, meningkatkan kecepatan delivery, dan menurunkan risiko konfigurasi yang inkonsisten. Dengan Dockerfile yang tepat, manajemen image yang disiplin, dan praktik keamanan dasar, container dapat menjadi fondasi deployment yang andal dalam ekosistem DevOps modern.

## Referensi
1. Docker Documentation: https://docs.docker.com/
2. Dockerfile Best Practices: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
3. OCI Specifications: https://opencontainers.org/
4. CIS Docker Benchmark: https://www.cisecurity.org/benchmark/docker
5. NIST SP 800-190 (Application Container Security Guide)
