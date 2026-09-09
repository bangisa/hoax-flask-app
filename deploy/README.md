# Deployment VPS

Domain: https://hoaxflaskapp.isacool.my.id

Deployment aktif diverifikasi pada 9 September 2026. HTTP dialihkan ke HTTPS;
sertifikat awal berlaku hingga 7 Desember 2026 dan timer Certbot aktif.
Halaman, CSS, prediksi valid, serta error input kosong diuji melalui HTTPS
publik. Service berjalan sebagai user khusus dan aktif saat boot.
Simulasi pembaruan sertifikat (`certbot renew --cert-name
hoaxflaskapp.isacool.my.id --dry-run --no-random-sleep-on-renew`) juga berhasil.

## Arsitektur

Debian 12, Python 3.11, Apache sebagai reverse proxy, Gunicorn sebagai proses
aplikasi, dan systemd sebagai pengelola service. Nginx terpasang di server
namun tidak aktif; jangan mengaktifkannya karena port 80/443 digunakan Apache.

- Release aplikasi: `/opt/hoaxflaskapp/releases/1912f16`.
- Release aktif: `/opt/hoaxflaskapp/current` (symlink).
- Virtual environment: `/opt/hoaxflaskapp/venv`.
- User service tanpa login: `hoaxflaskapp`.
- Gunicorn: `127.0.0.1:18080`, dua worker sinkron.
- Service: `/etc/systemd/system/hoaxflaskapp.service`.
- Virtual host awal: `/etc/apache2/sites-available/hoaxflaskapp.conf`.
- HTTPS: dikelola Certbot dengan plugin Apache.

`hoaxflaskapp.apache.conf` adalah template HTTP awal. Certbot membuat
konfigurasi HTTPS dan redirect di server. Jangan menimpa konfigurasi HTTPS
hasil Certbot dengan template HTTP ini saat melakukan pembaruan aplikasi.

## File deployment

Sertakan `app.py`, `preprocessing.py`, `requirements.txt`, `models/`, stopword
dan dokumentasinya di `data/`, `templates/`, `static/`, dan empat JSON evaluasi.
Dataset penelitian, baseline model, dan `archive/` tidak diperlukan.

## Operasional

```sh
sudo systemctl status hoaxflaskapp
sudo journalctl -u hoaxflaskapp -n 100 --no-pager
sudo apache2ctl configtest
sudo certbot certificates
sudo systemctl status certbot.timer
```

Gunakan reload Apache setelah pemeriksaan konfigurasi berhasil. Jangan restart
PM2 atau mengubah virtual host situs lain untuk memperbarui aplikasi ini.

## Pembaruan dan rollback

Unggah release baru ke folder berbeda, instal dependency yang sesuai, lalu uji
startup, halaman, CSS, prediksi, dan error input kosong sebelum mengalihkan
symlink `current` dan me-restart service. Periksa juga akses HTTPS dari luar VPS.

Untuk rollback, arahkan `current` kembali ke release sebelumnya dan restart
service. Jika dependency berubah, pulihkan juga virtual environment yang sesuai.
Simpan release lama sampai pembaruan baru terverifikasi.

## DNS dan sertifikat

DNS A domain mengarah ke IP publik VPS. Jangan menambahkan AAAA kecuali IPv6
server tersedia. Sertifikat domain ini dipisahkan dari sertifikat situs lain.
Pembaruan sertifikat menggunakan timer Certbot sistem.

Akses SSH memakai key lokal yang sudah tersedia; credential tidak disertakan
ke repositori atau release aplikasi.
