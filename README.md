# Network traffic incident analysis using tcpdump

## 📌 Table of contents

1. [Background](#background)
2. [Incident overview](#scenario)
3. [Objective of analysis](#objective)
4. [Network traffic analysis](#analysis)
5. [Insights and lessons learned](#insight)

---

## 🧠 Background <a name="background">

Studi kasus ini saya susun sebagai latihan cybersecurity dengan fokus pada keamanan jaringan dan analisis insiden. Skenario yang digunakan berupa simulasi gangguan akses website karena masalah pada lalu lintas jaringan, sehingga alur kejadiannya mudah dipahami dan terasa realistis.

Pembahasan mengacu pada pembelajaran di program Google Cybersecurity Professional Certificate, khususnya course Connect and Protect: Networks and Network Security. Proses analisis dimulai dengan menelusuri lalu lintas DNS dan ICMP menggunakan tools analisis jaringan untuk menemukan penyebab gangguan dan menentukan langkah penanganan yang tepat.

---

## 🚨 Incident overview <a name="scenario">

Beberapa pelanggan melaporkan website yummyrecipesforme.com tidak bisa diakses. Halaman tidak pernah terbuka penuh dan selalu muncul pesan error destination port unreachable. Keluhan ini terjadi berulang dan mulai menurunkan kepercayaan pengguna terhadap layanan.

<img width="902" height="448" alt="LKXsnNIhT0e1mAz5AEvxog_d363c94e0a4f4a8b90b0be403f6ee1f1_mMBaLWLyXG2omYBcSdjuR8y5_S59zow1ZEPYdjNyJzA1B0r55nI9KmDosI8QHXcEwE51NxM3N5gNtMgSOyVDHyJVLZvZA7_jJtkzUKfxuqFUJPHs57vVVES-LbG5teR8eir4idaqsxFaYJhhVJZn-a_S-txb7zQNIZq07XESgSkqDHuzfvALfYk3lipGVBY" src="https://github.com/user-attachments/assets/b87f32cf-0da8-41ca-9f4a-ba9c4495d203" />

Sebagai analis keamanan siber di perusahaan penyedia layanan TI, saya mencoba mengakses website tersebut dan menemukan kendala yang sama. Pengecekan sisi jaringan dilakukan dengan menjalankan tcpdump lalu memuat ulang halaman. Browser mengirim permintaan DNS lewat UDP untuk mencari IP domain, tetapi balasan yang diterima justru pesan ICMP udp port 53 unreachable dari server DNS.

Log tcpdump menunjukkan permintaan DNS terus dikirim ke server DNS dengan IP 203.0.113.2. Setiap permintaan selalu dibalas pesan error ICMP. Kondisi ini menandakan layanan DNS di port 53 tidak bisa diakses. Proses resolusi domain pun gagal dan browser tidak pernah masuk ke tahap mengirim permintaan HTTPS ke server website.

Temuan awal ini mengarah pada gangguan layanan DNS yang langsung berdampak ke akses website. Hasil analisis kemudian saya sampaikan ke atasan dan diteruskan ke tim teknis untuk ditangani lebih lanjut.

---

## 🎯 Objective of analysis <a name="objective">

Studi kasus ini dibuat untuk memahami gangguan jaringan yang bikin website tidak bisa diakses. Fokusnya ada pada mencari sumber masalah dengan menelusuri lalu lintas jaringan lewat data hasil tangkapan paket.

Tujuan analisisnya:
- Mengenali protokol jaringan yang terlibat dan terdampak selama gangguan
- Mengecek lalu lintas DNS dan ICMP lewat log tcpdump untuk melihat pola kegagalan komunikasi
- Menemukan penyebab awal kegagalan resolusi domain yang bikin website tidak bisa diakses pengguna

---

## 🔍 Network traffic analysis <a name="analysis">

| Masalah utama dari analisis lalu lintas DNS dan ICMP | Penjelasan                                                                                                                                                                                                                                                                                                                     |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Penggunaan protokol DNS dan ICMP                     | Browser mengirim permintaan DNS lewat UDP untuk mencari alamat IP domain yummyrecipesforme.com. Permintaan sampai ke server, tapi responsnya bukan jawaban DNS. Server malah membalas dengan pesan ICMP yang menunjukkan port 53 tidak bisa dijangkau. Ini menandakan ada masalah saat sistem mencoba akses layanan DNS.       |
| Pola lalu lintas di log tcpdump                      | Log menunjukkan pola jelas. Klien kirim paket UDP ke server DNS, server balas dengan ICMP udp port 53 unreachable. Port 53 standar untuk DNS. Pesan ini muncul berulang, menandakan layanan DNS di server bermasalah. Detail kueri, misal 35084 dan A?, menunjukkan ini permintaan record A, alias pencarian alamat IP domain. |
| Interpretasi masalah di log                          | Pesan ICMP menandakan server DNS tidak merespons port 53. Layanan DNS mungkin mati, diblokir, atau tidak bisa dijangkau. DNS penting sebelum browser bisa terhubung ke website. Kegagalan ini bikin browser tidak dapat alamat IP dan website tidak bisa diakses.                                                              |

| Hasil analisis dan penyebab utama insiden | Penjelasan                                                                                                                                                                                                                                                                                 |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Waktu insiden pertama kali                | Insiden terlihat pukul 13.24, tercatat di log tcpdump: 13:24:32.192571. Saat itu paket DNS pertama dikirim tapi server gagal merespons.                                                                                                                                                    |
| Skenario, peristiwa, dan gejala           | Pengguna lapor website yummyrecipesforme.com tidak bisa dibuka. Halaman tidak pernah termuat dan selalu muncul pesan destination port unreachable. Gejala sama muncul di beberapa pengguna dan bisa direplikasi saat dites.                                                                |
| Status insiden                            | Saat analisis, insiden masih berlangsung. Tim keamanan siber dan teknis sedang menangani. Analis fokus memeriksa lalu lintas jaringan dan menyampaikan temuan awal untuk eskalasi.                                                                                                         |
| Hasil investigasi                         | Log tcpdump menunjukkan permintaan DNS lewat UDP ke server port 53 gagal. Server membalas ICMP udp port 53 unreachable. Hal ini terjadi berulang kali. Browser tidak dapat alamat IP, koneksi website gagal sejak awal.                                                                    |
| Dugaan akar penyebab                      | Masalah utama gangguan layanan DNS di server. Kemungkinan karena DNS mati, konfigurasi jaringan salah, firewall blok port 53, atau serangan yang menargetkan layanan DNS.                                                                                                                  |
| Langkah selanjutnya                       | Fokus pemulihan layanan DNS dan mencegah kejadian sama. Tim teknis harus pastikan DNS berjalan normal di port 53, cek konfigurasi firewall, uji konektivitas dari beberapa titik, tingkatkan pemantauan lalu lintas DNS, dan setelah perbaikan, tes akses website untuk memastikan stabil. |

**Analysis notes:**

Analisis dilakukan pakai tcpdump untuk melihat pola lalu lintas jaringan dan menemukan aktivitas yang aneh. Fokus diarahkan ke sumber dan tujuan paket, protokol yang dipakai, dan frekuensi paket dalam waktu singkat. Observasi menunjukkan ada pola trafik yang menyimpang dari normal. Temuan ini jadi titik awal untuk investigasi lebih lanjut, menelusuri penyebab dan dampak insiden.

---

## 💡 Insights and lessons learned <a name="insight">

Insiden ini menegaskan bahwa DNS adalah layanan penting yang langsung memengaruhi ketersediaan website. Gangguan di port 53 bikin resolusi domain gagal, meskipun server web sendiri masih berjalan normal.

Analisis paket pakai tcpdump menunjukkan permintaan DNS lewat UDP terus dikirim, tapi dibalas ICMP destination port unreachable. Pola ini jelas menandakan layanan DNS bermasalah dan membantu cepat menemukan akar masalah di jaringan.

Studi kasus ini mengajarkan pentingnya pemantauan layanan inti dan pengecekan respons jaringan secara rutin. Analisis paket memberi tim teknis gambaran akurat untuk perbaikan dan memperkuat sistem supaya lebih tahan terhadap gangguan serupa di masa depan.
