# Modul 01 - Linux Server dan CLI

## Pendahuluan
Linux server merupakan tulang punggung sebagian besar infrastruktur digital modern, baik pada sistem berbasis cloud, on-premise, hybrid, maupun edge. Web server, API gateway, database node, message broker, observability stack, hingga pipeline CI/CD lazimnya berjalan di atas Linux karena stabil, fleksibel, dan efisien dalam konsumsi sumber daya.

Dalam praktik DevOps, kemampuan menggunakan Command Line Interface (CLI) bukan sekadar keterampilan teknis tambahan, melainkan kompetensi inti. Melalui CLI, engineer dapat melakukan provisioning, konfigurasi, deployment, inspeksi log, pengelolaan service, dan automasi dalam cara yang konsisten serta dapat direproduksi.

Modul ini dirancang sebagai fondasi operasional sebelum peserta mempelajari keamanan akses, reverse proxy, containerization, cloud resource management, dan monitoring.

## Tujuan Pembelajaran
Setelah mempelajari modul ini, peserta diharapkan mampu:

1. Menjelaskan peran Linux server dalam arsitektur sistem modern.
2. Memahami struktur filesystem Linux dan prinsip manajemen proses.
3. Menggunakan command CLI esensial untuk operasi server harian.
4. Mengelola service menggunakan systemd secara benar.
5. Menerapkan hardening dasar server untuk lingkungan produksi awal.
6. Menyusun langkah troubleshooting berbasis data log dan metrik sistem.

## Ruang Lingkup Materi
- Arsitektur Linux server dan komponen utama.
- Filesystem hierarchy standard (FHS).
- Manajemen user, file permission, dan process lifecycle.
- Network inspection dan validasi layanan.
- Pengelolaan service menggunakan systemd.
- Praktik operasional dan automasi sederhana dengan shell script.

## Prasyarat
- Memahami konsep dasar sistem operasi.
- Memiliki akses ke 1 VM/VPS Linux (disarankan Ubuntu Server 22.04/24.04 LTS).
- Memiliki akun dengan hak sudo.
- Memahami dasar jaringan: IP, port, dan protokol TCP/UDP.

## Konsep Dasar
### 1. Arsitektur Kernel dan Userland

Linux membagi fungsi sistem antara kernel (manajemen hardware, scheduling, networking, memory) dan userland (daemon, utilitas, runtime aplikasi). Pemisahan ini memungkinkan patch kernel terfokus tanpa mengubah aplikasi, serta memberi fleksibilitas modular untuk layanan jaringan.

Konsekuensi praktis:

- Administrasi kernel: konfigurasi sysctl, modul kernel, parameter keamanan.
- Administrasi userland: service management, package lifecycle, runtime environment.

### 2. Shell, Automasi, dan Reproduksibilitas

Shell adalah antarmuka tekstual utama untuk manajemen server. Selain perintah ad-hoc, skrip shell dan tool automasi (Ansible, shell scripts idempotent) menjamin reproduksibilitas konfigurasi.

Prinsip operasional:

- Tulis skrip dengan `set -euo pipefail` dan logging yang jelas.
- Hindari side-effect tersembunyi; buat skrip dapat diuji di lingkungan staging.
- Gunakan package manager dan konfigurasi deklaratif bila memungkinkan.

### 3. Hierarki Filesystem dan Data Semantik

Filesystem Linux mengikuti konvensi yang memisahkan konfigurasi, data dinamis, dan artefak aplikasi. Menjaga pemisahan ini memudahkan backup, restore, dan manajemen permission.

Rekomendasi singkat:

- Konfigurasi sistem di `/etc` harus dikelola melalui config management.
- Data stateful di `/var` atau volume terpisah yang dapat di-snapshot.
- Aplikasi pihak ketiga diisolasi di `/opt` atau container untuk mengurangi konflik.

### 4. Proses, Service Lifecycle, dan Observabilitas

Memahami lifecycle proses penting untuk operasi: process states, signals, dan cara systemd mengelola unit. Observabilitas pada level proses (ps, top, systemd-cgtop, journalctl) membantu diagnosa awal.

Praktik operasi:

- Gunakan `systemd` unit file yang mendeskripsikan restart policy, resource limits, dan environment.
- Kumpulkan logs ke systemd-journald dan/atau forwarder terpusat untuk analisis historis.
- Tetapkan resource limit via cgroups bila perlu untuk isolasi beban.

### 5. Manajemen Paket dan Patch

Pembaruan yang terkontrol (patch window, staging, canary) mengurangi risiko gangguan layanan. Gunakan channel paket resmi dan mekanisme rollback bila tersedia.

### 6. Jaringan Dasar dan Troubleshooting

Konsep penting: alamat IP, routing, firewall (iptables/nftables/ufw), dan tool diagnostik (ip, ss, tcpdump). Setelah konfigurasi jaringan, validasi end-to-end dan monitoring konektivitas harus dilakukan rutin.

### 7. Struktur Belajar Linux untuk Pemula

Agar tidak kewalahan, pelajari Linux berurutan dari yang paling sering dipakai:

1. Navigasi file (`pwd`, `ls`, `cd`).
2. Hak akses (`chmod`, `chown`).
3. Proses dan service (`ps`, `systemctl`).
4. Jaringan dasar (`ip`, `ss`, `ping`, `curl`).
5. Logging (`journalctl`, file log di `/var/log`).

### 8. Kebiasaan Aman Saat Menjalankan Perintah

Kebiasaan ini penting untuk pemula agar mengurangi kesalahan:

- Selalu cek perintah sebelum menekan Enter.
- Jalankan perintah baca status dulu sebelum perintah yang mengubah sistem.
- Simpan catatan command penting agar mudah diulang.
- Gunakan `sudo` hanya saat memang diperlukan.

Ringkasnya, `Konsep Dasar` di Linux adalah fondasi untuk keputusan operasional yang aman dan dapat direproduksi.

## Cara Kerja
Alur operasional umum engineer infrastruktur:

1. Provision server baru dari image standar.
2. Login awal via SSH.
3. Update patch sistem dan instal tools dasar.
4. Konfigurasi firewall dan akses administratif.
5. Instal service aplikasi.
6. Validasi service, cek log, dan observasi resource.
7. Dokumentasi command penting dalam runbook.
8. Otomasi tugas berulang dengan shell script atau tools konfigurasi.

Prinsip utama: setiap perubahan harus dapat dilacak, divalidasi, dan direproduksi.

## Contoh Implementasi
Skenario: menyiapkan server Ubuntu untuk aplikasi web backend.

### Tahap 1 - Update sistem

~~~bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `apt update` memperbarui daftar paket.
- `apt upgrade -y` memasang update keamanan dan bugfix.
- `autoremove -y` membersihkan paket yang tidak lagi dipakai.

### Tahap 2 - Instal utilitas dasar

~~~bash
sudo apt install -y curl wget git vim htop unzip ca-certificates jq
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Paket ini membantu operasi harian: unduh file, edit config, cek proses, dan parsing JSON.

### Tahap 3 - Verifikasi kondisi sistem

~~~bash
uname -a
cat /etc/os-release
uptime
free -h
df -h
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `uname -a` melihat informasi kernel.
- `os-release` memastikan versi distro.
- `uptime`, `free -h`, dan `df -h` dipakai untuk cek kondisi CPU/memory/disk awal.

### Tahap 4 - Konfigurasi waktu server

~~~bash
timedatectl
sudo timedatectl set-timezone Asia/Jakarta
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Zona waktu yang benar penting agar log, jadwal backup, dan cron konsisten.

## Contoh Perintah dan Konfigurasi
### A. Navigasi dan manajemen file

~~~bash
pwd
ls -lah
cd /var/log
mkdir -p /opt/project/{app,logs,scripts}
touch /opt/project/logs/bootstrap.log
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `pwd` menampilkan posisi direktori saat ini.
- `ls -lah` menampilkan daftar file dengan ukuran human-readable.
- `mkdir -p` membuat struktur folder aplikasi sekaligus.

### B. Permission dan ownership

~~~bash
sudo chown -R $USER:$USER /opt/project
chmod 750 /opt/project
chmod 640 /opt/project/logs/bootstrap.log
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `chown` mengubah pemilik folder ke user operasional.
- `750` memberi akses penuh ke owner, terbatas ke group, menutup others.
- `640` cocok untuk file log agar tidak bisa dieksekusi.

### C. Manajemen proses

~~~bash
ps aux --sort=-%mem | head -n 15
top
htop
pgrep -a nginx
kill -TERM <PID>
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `ps` menampilkan proses yang paling boros memory.
- `pgrep -a nginx` mencari PID dan command proses nginx.
- `kill -TERM` menghentikan proses secara graceful.

### D. Manajemen service dengan systemd

~~~bash
sudo systemctl status ssh
sudo systemctl restart ssh
sudo systemctl enable ssh
sudo systemctl list-units --type=service --state=running
journalctl -u ssh --since "30 minutes ago"
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `status` memeriksa kondisi service.
- `restart` menerapkan perubahan konfigurasi service.
- `enable` membuat service otomatis aktif saat boot.
- `journalctl` membaca log service untuk troubleshooting.

### E. Diagnostik jaringan

~~~bash
ip a
ip route
ss -tulpen
curl -I http://127.0.0.1
ping -c 4 1.1.1.1
dig example.com +short
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `ip a` dan `ip route` menampilkan konfigurasi interface dan routing.
- `ss -tulpen` melihat port yang sedang listening.
- `curl -I` mengetes endpoint lokal.
- `dig` memeriksa resolusi DNS.

### F. UFW firewall dasar

~~~bash
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

- Kebijakan default menolak koneksi masuk yang tidak diizinkan.
- Hanya port penting (SSH, HTTP, HTTPS) yang dibuka.
- `status verbose` memastikan rule firewall sudah aktif.

### G. Script health check sederhana

~~~bash
cat <<'EOF' > /opt/project/scripts/healthcheck.sh
#!/usr/bin/env bash
set -euo pipefail

echo "=== HEALTH CHECK $(date -Iseconds) ==="
echo "Load:"
uptime
echo

echo "Memory:"
free -h
echo

echo "Disk:"
df -h /
echo

echo "Listening ports:"
ss -tuln
EOF

chmod +x /opt/project/scripts/healthcheck.sh
/opt/project/scripts/healthcheck.sh
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Script ini mengumpulkan data kesehatan server secara cepat.
- Cocok dipakai sebelum dan sesudah perubahan konfigurasi.

### H. Simpan output pengecekan ke file laporan

~~~bash
/opt/project/scripts/healthcheck.sh > /opt/project/logs/health-$(date +%F).log
tail -n 30 /opt/project/logs/health-$(date +%F).log
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Output health check disimpan ke file log harian.
- `tail` membantu melihat hasil terbaru tanpa membuka seluruh file.

## Pendalaman Materi
### Pendalaman: Runbook singkat dan Triase Insiden (untuk pemula)

Untuk pemula, lebih berguna jika triase dan langkah respons disajikan dalam bentuk runbook singkat yang dapat diikuti. Berikut contoh runbook sederhana untuk insiden layanan tidak responsif.

Contoh runbook singkat (triase awal):

1. Konfirmasi dampak: cek akses layanan dari client dan metrik utama (`curl`, `uptime`, `uptime` pada host).
2. Periksa resource host: `top`/`htop`, `free -h`, `df -h` untuk mendeteksi bottleneck.
3. Periksa log service: `journalctl -u <service> -n 200 --no-pager` atau file di `/var/log`.
4. Cek konektivitas jaringan: `ss -tuln`, `ip route`, dan `ping`/`curl` terhadap dependency penting.
5. Terapkan mitigasi sementara (mis. restart service) hanya setelah identifikasi awal; catat perubahan pada runbook.

Contoh skrip kesehatan sederhana (tambahan):

~~~bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== HEALTH CHECK $(date -Iseconds) ==="
echo "Load:"; uptime
echo
echo "Memory:"; free -h
echo
echo "Disk:"; df -h /
echo
echo "Listening ports:"; ss -tuln
~~~

Gunakan skrip ini sebagai langkah awal dalam runbook untuk mengumpulkan bukti sebelum melakukan tindakan lebih lanjut.

 

## Kerangka Keputusan Operasional
### A. Kapan Restart Service

Restart diperlukan jika:

- Konfigurasi berubah dan service tidak mendukung reload.
- Proses berada pada kondisi hang.
- Memory leak telah dikonfirmasi sementara belum ada patch.

Restart tidak disarankan sebagai respons pertama untuk semua masalah. Engineer harus memeriksa log akar masalah agar insiden tidak berulang.

### B. Kapan Reboot Server

Reboot dilakukan pada kondisi tertentu:

- Kernel patch membutuhkan restart host.
- Resource kernel mengalami degradasi berulang.
- Proses zombie atau deadlock tidak pulih dengan restart service.

Pada lingkungan produksi, reboot perlu melalui change window, notifikasi pemangku kepentingan, dan validasi pasca reboot.

 
## Best Practice Singkat
- Gunakan versi LTS untuk produksi.
- Dokumentasikan setiap perubahan penting pada runbook.
- Hindari login root langsung untuk tugas rutin.
- Validasi dampak perubahan sebelum restart service.
- Pantau kapasitas disk, memory, dan load secara berkala.
- Gunakan script idempoten untuk tugas berulang.

## Ringkasan
Penguasaan Linux server dan CLI adalah fondasi kompetensi engineer infrastruktur. Dengan memahami filesystem, proses, service lifecycle, jaringan, dan logging, peserta dapat mengelola server secara aman, konsisten, dan efisien. Kemampuan ini menjadi prasyarat utama untuk topik lanjutan seperti hardening akses, reverse proxy, containerization, dan monitoring.

## Referensi
1. Ubuntu Server Documentation: https://documentation.ubuntu.com/server/
2. Linux Man Pages: https://man7.org/linux/man-pages/
3. systemd Documentation: https://www.freedesktop.org/wiki/Software/systemd/
4. GNU Coreutils Manual: https://www.gnu.org/software/coreutils/manual/
5. The Linux Command Line (William Shotts)
