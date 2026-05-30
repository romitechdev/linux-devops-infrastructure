# Modul 06 - Cloud Computing: VPS dan Object Storage

## Pendahuluan
Cloud computing memungkinkan organisasi mengakses sumber daya komputasi secara elastis dengan model bayar sesuai pemakaian. Untuk banyak tim produk, kombinasi VPS dan object storage adalah titik awal yang realistis: VPS digunakan untuk menjalankan service aplikasi, sementara object storage dipakai untuk aset statis, backup, dan arsip data.

Pemahaman dua layanan ini sangat penting karena menjadi jembatan antara praktik Linux administration dan kebutuhan deployment skala produksi.

## Tujuan Pembelajaran
Setelah mempelajari modul ini, peserta mampu:

1. Menjelaskan konsep cloud service model dengan fokus IaaS.
2. Memahami karakteristik VPS dan object storage.
3. Melakukan provisioning dan hardening awal VPS.
4. Mengintegrasikan object storage untuk upload, backup, dan lifecycle data.
5. Menerapkan praktik keamanan dan optimasi biaya cloud dasar.

## Ruang Lingkup Materi
- Konsep IaaS, region, availability zone.
- Provisioning VPS dan konfigurasi awal.
- IAM dan access key untuk object storage.
- Operasi bucket: create, upload, sync, lifecycle.
- Strategi backup dan restore.
- Kontrol biaya dan praktik governance dasar.

## Prasyarat
- Akun cloud provider aktif.
- Pengetahuan dasar Linux server.
- Akses terminal dengan tool SSH dan AWS CLI (atau S3-compatible CLI).

## Konsep Dasar
### 1. VPS untuk Menjalankan Aplikasi

VPS adalah server virtual yang bisa Anda kelola seperti server biasa: install paket, konfigurasi firewall, dan menjalankan aplikasi.

### 2. Object Storage untuk File dan Backup

Object storage cocok untuk menyimpan file statis, backup database, dan arsip. Data disimpan dalam bucket dan diakses lewat API/CLI.

### 3. Region dan Latency

Pilih region yang dekat pengguna agar akses lebih cepat. Region juga memengaruhi biaya transfer data.

### 4. Durability dan Availability

Durability berarti kemungkinan data tetap aman sangat tinggi. Availability berarti layanan tetap bisa diakses saat dibutuhkan.

### 5. Backup dan Restore

Backup penting, tetapi yang lebih penting adalah memastikan backup bisa di-restore saat insiden.

### 6. IAM Dasar

Gunakan akun/role dengan hak seperlunya saja. Hindari memakai kredensial admin utama untuk operasi harian.

### 7. Pola Dasar Arsitektur Pemula

Pola yang mudah dipahami untuk tahap awal:

- Satu VPS untuk aplikasi.
- Satu bucket untuk aset statis.
- Satu bucket terpisah untuk backup.

Pemisahan ini membantu manajemen biaya dan memudahkan troubleshooting.

## Cara Kerja
1. Tim melakukan provisioning VPS di region terpilih.
2. Server diamankan dan disiapkan untuk deployment aplikasi.
3. Bucket object storage dibuat untuk data non-ephemeral.
4. Aplikasi membaca/menulis object melalui SDK/CLI.
5. Backup otomatis dijadwalkan ke bucket.
6. Lifecycle policy mengelola retensi agar biaya efisien.

## Contoh Implementasi
### A. Provisioning dan hardening awal VPS

~~~bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ufw fail2ban curl git

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Update paket memastikan server memakai patch terbaru.
- `ufw` dipakai untuk firewall dasar (deny incoming, allow outgoing).
- Rule hanya membuka port penting: SSH, HTTP, dan HTTPS.

### B. Setup object storage dengan AWS CLI (S3-compatible)

~~~bash
sudo apt install -y awscli
aws configure

aws s3 mb s3://myapp-assets --region ap-southeast-1
aws s3 mb s3://myapp-backup --region ap-southeast-1
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `aws configure` mengisi access key, secret key, dan region default.
- `aws s3 mb` membuat bucket baru untuk aset dan backup.

### C. Upload aset statis dan backup

~~~bash
aws s3 cp ./public/logo.png s3://myapp-assets/public/logo.png
aws s3 sync ./public s3://myapp-assets/public --delete

# Contoh backup database
pg_dump -h 127.0.0.1 -U appuser appdb | gzip > backup-$(date +%F).sql.gz
aws s3 cp backup-$(date +%F).sql.gz s3://myapp-backup/daily/
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `cp` mengunggah satu file, sedangkan `sync` menyamakan isi folder lokal ke bucket.
- `pg_dump | gzip` membuat backup database yang terkompresi.
- Backup lalu diunggah ke bucket agar tersimpan di luar server aplikasi.

### D. Uji restore backup (simulasi dasar)

~~~bash
aws s3 cp s3://myapp-backup/daily/backup-2026-05-30.sql.gz .
gzip -t backup-2026-05-30.sql.gz
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Baris pertama mengunduh file backup dari bucket.
- `gzip -t` mengecek file backup tidak rusak sebelum dipakai restore.

## Contoh Perintah dan Konfigurasi
### 1. Bucket versioning

~~~bash
aws s3api put-bucket-versioning \
  --bucket myapp-assets \
  --versioning-configuration Status=Enabled
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Versioning menyimpan versi lama objek saat file ditimpa atau terhapus.
- Fitur ini membantu pemulihan cepat jika ada kesalahan upload.

### 2. Lifecycle policy

File lifecycle.json:

~~~json
{
  "Rules": [
    {
      "ID": "DailyBackupRetention",
      "Status": "Enabled",
      "Filter": {"Prefix": "daily/"},
      "Expiration": {"Days": 30}
    }
  ]
}
~~~

Apply policy:

~~~bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket myapp-backup \
  --lifecycle-configuration file://lifecycle.json
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Policy ini menghapus backup lama otomatis sesuai umur retensi.
- Tujuannya menjaga biaya storage tetap terkendali.

### 3. Presigned URL untuk akses sementara

~~~bash
aws s3 presign s3://myapp-assets/public/logo.png --expires-in 900
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Membuat URL sementara selama 900 detik tanpa membuka bucket publik.
- Cocok untuk berbagi file internal secara aman dan terbatas waktu.

### 4. Verifikasi isi bucket

~~~bash
aws s3 ls s3://myapp-assets/public/
aws s3 ls s3://myapp-backup/daily/
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Memastikan file aset dan backup benar benar sudah tersimpan di bucket.

### 5. Ringkasan ukuran data bucket

~~~bash
aws s3 ls s3://myapp-backup/daily/ --recursive --human-readable --summarize
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Menampilkan total jumlah file dan total ukuran data untuk pemantauan biaya.

## Best Practice Singkat
- Pilih region dekat pengguna utama untuk menekan latency.
- Gunakan IAM minimal privilege.
- Aktifkan versioning dan encryption pada bucket penting.
- Terapkan lifecycle untuk kontrol biaya.
- Pisahkan environment dev, staging, production.
- Uji proses restore secara periodik.

## Ringkasan
VPS dan object storage adalah kombinasi praktis untuk infrastruktur aplikasi modern. VPS memberi kontrol penuh terhadap runtime, sedangkan object storage menawarkan skalabilitas penyimpanan untuk aset dan backup. Dengan desain akses yang aman, lifecycle data yang jelas, serta disiplin backup-restore, tim dapat mencapai operasi cloud yang andal dan efisien.

## Referensi
1. AWS EC2 Documentation: https://docs.aws.amazon.com/ec2/
2. Amazon S3 Documentation: https://docs.aws.amazon.com/s3/
3. AWS Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/
4. Cloudflare R2 Docs: https://developers.cloudflare.com/r2/
5. DigitalOcean Droplets Docs: https://docs.digitalocean.com/products/droplets/
