# Modul 02 - User Privilege dan SSH Key

## Pendahuluan
Keamanan akses server merupakan lapisan pertahanan pertama dalam pengelolaan infrastruktur. Banyak insiden operasional berawal dari kontrol akses yang lemah: akun bersama tanpa audit, login root langsung, penggunaan password sederhana, atau distribusi private key tanpa tata kelola. Dalam konteks DevOps dan Site Reliability Engineering, keamanan akses harus dirancang dengan prinsip least privilege, traceability, dan zero trust mindset.

Modul ini membahas model user privilege pada Linux serta implementasi autentikasi SSH berbasis key pair untuk akses remote yang lebih aman, auditable, dan sesuai praktik industri.

## Tujuan Pembelajaran
Setelah menyelesaikan modul ini, peserta mampu:

1. Menjelaskan konsep user, group, permission, dan ownership di Linux.
2. Menerapkan kontrol hak akses berbasis least privilege.
3. Mengonfigurasi sudo policy yang aman.
4. Membuat, mendistribusikan, dan mengelola SSH key pair.
5. Melakukan hardening layanan SSH untuk server publik.
6. Menyusun prosedur rotasi key dan audit akses berkala.

## Ruang Lingkup Materi
- Model akun Linux: root, user biasa, service account.
- Permission model: rwx, numeric mode, special bit (setuid/setgid/sticky).
- Sudoers policy dan audit command.
- SSH key-based authentication.
- Hardening sshd_config.
- Praktik rotasi key, revocation, dan incident response dasar.

## Prasyarat
- Memahami command Linux dasar.
- Memiliki akses ke server Linux dengan akun sudo.
- Client sudah memiliki OpenSSH.
- Koneksi jaringan stabil ke server target.

## Konsep Dasar
### 1. User dan Group

Linux adalah sistem multi-user. Setiap orang sebaiknya memakai akun masing masing agar aktivitas dapat dilacak dengan jelas.

Istilah penting:

- User: akun login untuk manusia atau service.
- Group: kumpulan user untuk memudahkan pembagian hak akses.
- UID/GID: identitas numerik user/group di sistem.

### 2. Permission dan Ownership

Setiap file punya owner dan izin akses. Izin ini menentukan siapa yang boleh membaca, menulis, atau menjalankan file.

Format izin umum:

- `r` (read), `w` (write), `x` (execute).
- Berlaku untuk owner, group, dan others.

### 3. Sudo untuk Hak Admin

`sudo` dipakai saat user biasa butuh menjalankan perintah sebagai admin. Untuk pemula, prinsipnya sederhana: gunakan sudo hanya saat perlu, lalu kembali ke perintah biasa.

### 4. SSH Key Authentication

Akses SSH paling aman untuk server adalah memakai key, bukan password.

Komponen key:

- Private key: disimpan di komputer client dan tidak boleh dibagikan.
- Public key: dipasang di server pada `authorized_keys`.

### 5. Hardening Dasar SSH

Langkah dasar yang penting untuk server publik:

- Nonaktifkan login root.
- Nonaktifkan password login.
- Batasi user yang boleh login SSH.

### 6. Audit Log Akses

Log SSH membantu mengecek siapa login, kapan login, dan apakah ada percobaan gagal berulang. Ini penting untuk troubleshooting dan keamanan harian.

### 7. Urutan Aman untuk Pemula

Agar tidak bingung, gunakan urutan kerja berikut setiap kali mengatur akses server:

1. Buat user personal terlebih dahulu.
2. Uji login user baru sebelum menonaktifkan akses lama.
3. Pasang SSH key dan pastikan login key berhasil.
4. Baru setelah itu nonaktifkan login password/root.
5. Simpan catatan perubahan agar mudah rollback bila terjadi lockout.

## Cara Kerja
Alur aman pengelolaan akses server:

1. Buat user personal untuk setiap engineer.
2. Tetapkan group sesuai peran.
3. Berikan hak sudo minimal sesuai tugas.
4. Aktifkan login SSH key-only.
5. Nonaktifkan root login dan password login.
6. Monitor log autentikasi.
7. Rotasi key secara periodik dan cabut akses yang tidak aktif.

## Contoh Implementasi
### A. Membuat user operasional

~~~bash
sudo adduser devops01
sudo usermod -aG sudo devops01
id devops01
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `adduser` membuat akun baru dan home directory.
- `usermod -aG sudo` menambahkan user ke grup sudo tanpa menghapus grup lama.
- `id` dipakai untuk memastikan UID, GID, dan grup user sudah benar.

### B. Membuat service account non-login

~~~bash
sudo useradd -r -s /usr/sbin/nologin -d /nonexistent appsvc
id appsvc
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `-r` membuat akun sistem untuk service.
- `-s /usr/sbin/nologin` mencegah akun dipakai login interaktif.
- `-d /nonexistent` menandai akun ini tidak dipakai untuk sesi user biasa.

### C. Menata ownership direktori aplikasi

~~~bash
sudo mkdir -p /srv/myapp
sudo chown -R devops01:devops01 /srv/myapp
sudo chmod -R 750 /srv/myapp
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `mkdir -p` membuat folder aplikasi jika belum ada.
- `chown -R` menjadikan `devops01` sebagai pemilik folder dan isinya.
- `chmod -R 750` memberi akses penuh ke owner, akses baca-jalankan ke group, dan menutup akses user lain.

### D. Membuat SSH key pair di client

~~~bash
ssh-keygen -t ed25519 -a 100 -C "devops01@infra" -f ~/.ssh/id_ed25519_infra
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `-t ed25519` memilih algoritma key modern yang cepat dan aman.
- `-a 100` menambah iterasi KDF agar private key lebih sulit di-bruteforce.
- `-C` memberi label agar key mudah dikenali.
- `-f` menentukan nama file output key.

### E. Mengirim public key ke server

~~~bash
ssh-copy-id -i ~/.ssh/id_ed25519_infra.pub devops01@203.0.113.10
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Perintah ini menyalin public key ke `~/.ssh/authorized_keys` milik user di server.
- Setelah berhasil, login berikutnya bisa memakai key tanpa password akun server.

### F. Uji login

~~~bash
ssh -i ~/.ssh/id_ed25519_infra devops01@203.0.113.10
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `-i` menunjuk private key yang akan dipakai.
- Jika berhasil login, berarti pasangan key dan konfigurasi permission sudah benar.

### G. Hardening SSH bertahap tanpa lockout

~~~bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo sshd -t
sudo systemctl restart ssh
sudo systemctl status ssh --no-pager
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Backup file konfigurasi dibuat dulu agar mudah rollback.
- `sshd -t` memastikan konfigurasi valid sebelum restart.
- Setelah restart, cek status service untuk memastikan SSH tetap aktif.

## Contoh Perintah dan Konfigurasi
### 1. Konfigurasi SSH hardening

Lokasi file: /etc/ssh/sshd_config

~~~conf
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AuthenticationMethods publickey
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowUsers devops01
UsePAM yes
~~~

Validasi dan reload:

~~~bash
sudo sshd -t
sudo systemctl restart ssh
sudo systemctl status ssh
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `PermitRootLogin no` menutup login langsung sebagai root.
- `PasswordAuthentication no` memaksa login menggunakan key.
- `AllowUsers devops01` membatasi hanya user tertentu yang boleh login.
- Perintah validasi/restart dipakai agar perubahan aktif dengan aman.

### 2. Permission direktori SSH user

~~~bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R $USER:$USER ~/.ssh
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `700` untuk folder `.ssh` berarti hanya pemilik yang bisa mengakses.
- `600` untuk `authorized_keys` mencegah file dibaca user lain.
- Ownership yang benar mencegah error `Permission denied (publickey)`.

### 3. Sudo policy granular (contoh)

Gunakan visudo untuk mencegah syntax error.

~~~bash
sudo visudo
~~~

Tambahkan rule contoh:

~~~text
%deploy ALL=(root) /bin/systemctl restart nginx, /bin/systemctl status nginx
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Rule ini memberi grup `deploy` hak terbatas hanya untuk dua command.
- Ini lebih aman daripada memberi akses sudo penuh ke semua command.

### 4. Audit login dan sudo

~~~bash
sudo grep "Accepted publickey" /var/log/auth.log | tail -n 20
sudo grep "sudo:" /var/log/auth.log | tail -n 20
last -a | head
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- Baris pertama menampilkan histori login SSH berbasis key.
- Baris kedua menampilkan aktivitas sudo terakhir.
- `last -a` menampilkan histori login user beserta alamat sumber.

### 5. Validasi cepat user, group, dan sudo

~~~bash
id devops01
groups devops01
sudo -l -U devops01
getent passwd devops01
~~~

Penjelasan kode:

Bagian ini menjelaskan fungsi setiap baris perintah atau konfigurasi secara ringkas agar mudah dipahami pemula.

- `id` dan `groups` memverifikasi keanggotaan grup user.
- `sudo -l -U` menampilkan command sudo yang diizinkan.
- `getent passwd` memastikan akun user terdaftar benar di sistem.

## Best Practice Singkat
- Gunakan ed25519 dengan passphrase kuat.
- Nonaktifkan root login dan password authentication di server publik.
- Terapkan least privilege pada sudoers.
- Audit auth.log secara rutin.
- Lakukan rotasi dan revocation key secara disiplin.
- Pisahkan akun manusia dan service account.

## Ringkasan
Manajemen user privilege dan SSH key adalah fondasi keamanan operasional Linux. Dengan menerapkan user non-root, policy sudo terkontrol, SSH key-only, serta audit berkala, organisasi dapat menurunkan risiko akses tidak sah dan meningkatkan ketertelusuran aktivitas administratif.

## Referensi
1. OpenSSH Manual: https://www.openssh.com/manual.html
2. OpenSSH Cookbook: https://en.wikibooks.org/wiki/OpenSSH/Cookbook
3. CIS Benchmarks Linux: https://www.cisecurity.org/cis-benchmarks
4. Debian Securing Manual: https://www.debian.org/doc/manuals/securing-debian-manual/
5. sudo Documentation: https://www.sudo.ws/docs/
