# Modul 07 - Monitoring Server

## Pendahuluan
Monitoring server adalah proses pengamatan berkelanjutan terhadap kondisi infrastruktur dan aplikasi untuk menjaga availability, performance, dan reliability layanan. Pada lingkungan produksi, monitoring bukan fitur tambahan, melainkan sistem kendali utama agar tim dapat mendeteksi anomali lebih cepat, menurunkan MTTR, dan menjaga SLA/SLO.

Dalam praktik DevOps dan SRE, monitoring efektif dibangun di atas data observability: metrics, logs, dan traces. Modul ini berfokus pada monitoring infrastruktur server dengan stack populer Prometheus, Node Exporter, dan Grafana.

## Tujuan Pembelajaran
Setelah mempelajari modul ini, peserta mampu:

1. Menjelaskan konsep monitoring dan observability.
2. Memahami metrik server penting untuk operasi harian.
3. Mengimplementasikan stack monitoring dasar.
4. Menulis query PromQL sederhana untuk analisis.
5. Menyusun alert rule yang actionable.
6. Melakukan troubleshooting awal berbasis metrik dan log.

## Ruang Lingkup Materi
- Konsep metrics, logs, traces.
- Golden signals dan indikator kapasitas infrastruktur.
- Instalasi Node Exporter dan scrape Prometheus.
- Dashboard Grafana dasar.
- Alerting strategy dan anti-alert fatigue.
- Incident workflow dan runbook ringkas.

## Prasyarat
- Satu server Linux target monitoring.
- Docker dan Docker Compose (atau instalasi native Prometheus/Grafana).
- Akses jaringan antara Prometheus dan exporter.

## Konsep Dasar
### 1. Monitoring vs Observability

Monitoring mengawasi indikator yang diketahui, sedangkan observability menyediakan kemampuan analitis untuk memahami penyebab masalah baru dengan menggunakan telemetry yang terintegrasi.

### 2. Metrics, Logs, dan Traces

Untuk pemula, cukup pahami fungsi dasarnya:

- Metrics: angka time-series (CPU, memory, disk, request).
- Logs: catatan detail kejadian/error.
- Traces: jejak perjalanan request antar service.

### 3. Metrik Inti Server

Mulai dulu dari metrik yang paling mudah dipahami dan berdampak langsung:

- CPU usage
- Memory usage
- Disk usage
- Network traffic

### 4. Golden Signals

Empat sinyal penting:

- Latency
- Traffic
- Error
- Saturation

Saturation pada server biasanya terlihat dari CPU/memory/disk yang hampir penuh.

### 5. Alert Dasar

Alert dipakai agar tim cepat tahu saat nilai metrik melewati batas aman, misalnya CPU > 85% selama 10 menit.

### 6. Dashboard Dasar

Dashboard membantu melihat kondisi server secara ringkas. Untuk pemula, fokus dulu ke satu dashboard infrastruktur sebelum membuat dashboard yang lebih kompleks.

### 7. Urutan Belajar Monitoring untuk Pemula

Urutan paling aman:

1. Kumpulkan metrik host dulu (CPU, memory, disk).
2. Tampilkan metrik di dashboard.
3. Baru tambahkan alert sederhana.
4. Evaluasi apakah alert benar benar membantu.

## Cara Kerja
1. Exporter mengumpulkan metrik dari host.
2. Prometheus melakukan scrape endpoint metrik secara periodik.
3. Data disimpan dalam TSDB.
4. Grafana membaca data dan menampilkan dashboard.
5. Alert rule dievaluasi berkala; notifikasi dikirim jika kondisi terpenuhi.

## Contoh Implementasi
Skenario: monitoring satu VPS Linux dengan Prometheus + Node Exporter + Grafana.

### A. Menjalankan Node Exporter

~~~bash
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  -p 9100:9100 \
  prom/node-exporter:v1.8.1
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Node Exporter membaca metrik host Linux.
- Port `9100` dipakai Prometheus untuk scrape metrik.
- `--restart unless-stopped` membuat container otomatis hidup kembali jika host reboot.

### B. Menulis konfigurasi Prometheus

File prometheus.yml:

~~~yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["server-a.example.com:9100"]
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `scrape_interval: 15s` berarti Prometheus mengambil data tiap 15 detik.
- `targets` berisi alamat exporter yang ingin dipantau.
- `job_name` membantu pengelompokan target di dashboard dan query.

Tambahan pemula:

- Jika baru mulai, gunakan satu target dulu agar mudah validasi.
- Setelah stabil, baru tambah target server lain.

### C. Menjalankan Prometheus dan Grafana dengan Compose

File compose.monitoring.yaml:

~~~yaml
services:
  prometheus:
    image: prom/prometheus:v2.53.0
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prom_data:/prometheus
    ports:
      - "9090:9090"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:11.1.0
    container_name: grafana
    ports:
      - "3001:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    restart: unless-stopped

volumes:
  prom_data:
  grafana_data:
~~~

Jalankan stack:

~~~bash
docker compose -f compose.monitoring.yaml up -d
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Service `prometheus` menyimpan metrik.
- Service `grafana` menampilkan dashboard.
- Volume `prom_data` dan `grafana_data` menjaga data tetap ada saat container restart.
- Command `up -d` menjalankan stack monitoring di background.

### D. Verifikasi service monitoring aktif

~~~bash
docker compose -f compose.monitoring.yaml ps
curl -I http://127.0.0.1:9090
curl -I http://127.0.0.1:3001
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `ps` memastikan container Prometheus dan Grafana berjalan.
- Dua `curl -I` mengecek endpoint web masing masing service.

## Contoh Perintah dan Konfigurasi
### 1. Validasi endpoint metrics

~~~bash
curl -s http://127.0.0.1:9100/metrics | head -n 20
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Jika muncul banyak baris metrik, berarti Node Exporter berjalan normal.

### 2. Query PromQL dasar

~~~promql
# Persentase CPU usage (non-idle)
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Persentase memory used
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage root filesystem
(1 - (node_filesystem_avail_bytes{mountpoint="/",fstype!="tmpfs"} /
      node_filesystem_size_bytes{mountpoint="/",fstype!="tmpfs"})) * 100
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Query pertama: estimasi persentase CPU terpakai.
- Query kedua: persentase memori terpakai.
- Query ketiga: persentase disk root yang sudah terpakai.

### 3. Alert rules contoh

File alerts.yml:

~~~yaml
groups:
  - name: infra-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "CPU tinggi pada {{ $labels.instance }}"

      - alert: LowDiskSpaceRoot
        expr: (node_filesystem_avail_bytes{mountpoint="/",fstype!="tmpfs"} /
              node_filesystem_size_bytes{mountpoint="/",fstype!="tmpfs"}) * 100 < 15
        for: 15m
        labels:
          severity: critical
        annotations:
          summary: "Sisa disk root kurang dari 15% pada {{ $labels.instance }}"
~~~

Integrasikan alerts.yml di prometheus.yml:

~~~yaml
rule_files:
  - /etc/prometheus/alerts.yml
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Alert `HighCPUUsage` memicu peringatan jika CPU tinggi terus menerus.
- Alert `LowDiskSpaceRoot` memicu peringatan saat ruang disk hampir habis.
- `for` mencegah alert palsu dari lonjakan singkat.

### 4. Cek target scrape dari Prometheus API

~~~bash
curl -s http://127.0.0.1:9090/api/v1/targets | head -n 40
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Digunakan untuk melihat target mana yang statusnya `up` atau `down`.

## Best Practice Singkat
- Mulai dari metrik paling berdampak: CPU, memory, disk, network.
- Tulis alert yang actionable, bukan sekadar informatif.
- Simpan dashboard dan alert dalam version control.
- Evaluasi kualitas alert setiap incident review.
- Lakukan capacity planning berbasis tren data.

## Ringkasan
Monitoring server yang baik meningkatkan ketahanan layanan dengan deteksi dini, diagnosis cepat, dan respon insiden yang terarah. Implementasi Prometheus, exporter, dan Grafana memberikan fondasi observability praktis yang dapat dikembangkan menuju stack produksi yang lebih matang.

## Referensi
1. Prometheus Documentation: https://prometheus.io/docs/
2. Grafana Documentation: https://grafana.com/docs/
3. Google SRE Book: https://sre.google/sre-book/
4. Google SRE Workbook: https://sre.google/workbook/
5. OpenTelemetry Documentation: https://opentelemetry.io/docs/
