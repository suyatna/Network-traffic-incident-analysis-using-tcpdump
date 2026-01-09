# Network traffic incident analysis using tcpdump

## 📑 Table of contents

1. [Introduction](#introduction)
2. [Incident scenario](#scenario)
3. [Objective](#objective)
4. [Cybersecurity incident report](#report)
5. [Conclusion](#conclusion)

---

## 👋 Introduction <a name="introduction">

Studi kasus ini saya buat sebagai bagian dari latihan cybersecurity yang berfokus pada keamanan jaringan dan analisis insiden. Skenario yang digunakan berupa simulasi gangguan akses website akibat masalah pada lalu lintas jaringan, sehingga alur kejadian bisa dipahami secara jelas dan masuk akal.

Pembahasan disusun berdasarkan pembelajaran saya di program Google Cybersecurity Professional Certificate, khususnya pada course Connect and Protect: Networks and Network Security. Proses analisis dilakukan dengan menelusuri lalu lintas DNS dan ICMP menggunakan tools analisis jaringan untuk menemukan penyebab gangguan dan menentukan langkah penanganan yang sesuai.

---

## 💭 Incident scenario <a name="scenario">

Beberapa pelanggan melaporkan tidak bisa mengakses website yummyrecipesforme.com. Halaman tidak pernah terbuka sepenuhnya dan selalu berakhir dengan pesan kesalahan destination port unreachable. Keluhan ini terjadi berulang kali dan mulai memengaruhi kepercayaan pengguna terhadap layanan klien.

<img width="902" height="448" alt="LKXsnNIhT0e1mAz5AEvxog_d363c94e0a4f4a8b90b0be403f6ee1f1_mMBaLWLyXG2omYBcSdjuR8y5_S59zow1ZEPYdjNyJzA1B0r55nI9KmDosI8QHXcEwE51NxM3N5gNtMgSOyVDHyJVLZvZA7_jJtkzUKfxuqFUJPHs57vVVES-LbG5teR8eir4idaqsxFaYJhhVJZn-a_S-txb7zQNIZq07XESgSkqDHuzfvALfYk3lipGVBY" src="https://github.com/user-attachments/assets/b87f32cf-0da8-41ca-9f4a-ba9c4495d203" />

Sebagai analis keamanan siber di perusahaan penyedia layanan TI, saya mencoba mengakses website tersebut dan menemukan masalah yang sama. Untuk melihat apa yang terjadi di sisi jaringan, saya menjalankan tcpdump lalu memuat ulang halaman. Saat browser mengirim permintaan DNS melalui protokol UDP untuk mencari alamat IP domain, respons yang diterima justru berupa pesan ICMP udp port 53 unreachable dari server DNS.

Log tcpdump memperlihatkan permintaan DNS dikirim berulang kali ke server DNS dengan IP 203.0.113.2, namun setiap permintaan selalu dibalas dengan pesan kesalahan ICMP. Kondisi ini menunjukkan bahwa layanan DNS pada port 53 tidak dapat diakses. Proses resolusi domain pun gagal dan browser tidak pernah sampai ke tahap mengirim permintaan HTTPS ke server website.

Temuan awal ini mengarah pada gangguan pada layanan DNS yang berdampak langsung pada akses website. Hasil analisis kemudian saya laporkan kepada atasan dan diteruskan ke tim teknis untuk penanganan lebih lanjut.

---

## 🎯 Objective <a name="objective">

Studi kasus ini dibuat untuk memahami dan menganalisis gangguan jaringan yang berdampak langsung pada akses website. Fokus utama diarahkan pada pencarian sumber masalah melalui penelusuran lalu lintas jaringan dari data hasil tangkapan paket.

Tujuan analisis meliputi:
- Mengidentifikasi protokol jaringan yang terlibat dan terdampak selama gangguan akses website
- Menelaah lalu lintas DNS dan ICMP berdasarkan log tcpdump untuk melihat pola kegagalan komunikasi jaringan
- Menentukan penyebab awal kegagalan resolusi domain yang membuat website tidak dapat diakses oleh pengguna

---

## 📋 Cybersecurity incident report <a name="report">

### Network traffic analysis

|Bagian 1: Rangkuman masalah utama yang ditemukan dari analisis lalu lintas DNS dan ICMP|Penjelasan|
|---|---|
|A. Penggunaan protokol DNS dan ICMP|Pada insiden ini, browser menggunakan protokol UDP untuk menjalankan proses DNS saat mencari alamat IP domain yummyrecipesforme.com. Permintaan DNS sudah berhasil dikirim ke server tujuan. Respons yang diterima bukan jawaban DNS, melainkan pesan kesalahan melalui protokol ICMP. Pesan ini menandakan adanya masalah saat sistem mencoba mengakses layanan DNS.|
|B. Pola lalu lintas pada log tcpdump|Log menunjukkan pola yang konsisten. Komputer klien mengirim paket UDP ke server DNS, lalu server membalas dengan pesan ICMP ke klien. Pesan kesalahan yang muncul adalah udp port 53 unreachable. Port 53 merupakan port standar untuk layanan DNS. Kemunculan pesan ini secara berulang mengindikasikan bahwa layanan DNS di server tujuan mengalami gangguan. Detail seperti nomor kueri 35084 dan penanda A? menunjukkan bahwa permintaan tersebut adalah permintaan DNS untuk record A, yaitu proses pencarian alamat IP dari sebuah domain.|
|C. Interpretasi masalah pada log|Pesan kesalahan ICMP mengarah pada kondisi di mana server DNS tidak merespons permintaan yang masuk ke port 53. Layanan DNS kemungkinan tidak aktif, diblokir, atau tidak dapat dijangkau. DNS merupakan tahap awal sebelum browser terhubung ke website. Kegagalan pada tahap ini membuat browser tidak pernah memperoleh alamat IP tujuan. Dampaknya, website tidak dapat diakses oleh pengguna.|

|Bagian 2: Penjelasan hasil analisis data yang dilakukan dan uraian penyebab utama terjadinya insiden|Penjelasan|
|---|---|
|D. Waktu pertama kali insiden terjadi|Insiden pertama kali terdeteksi pada pukul 13.24. Hal ini terlihat dari stempel waktu awal pada log tcpdump, yaitu 13:24:32.192571. Waktu tersebut menandai saat paket DNS pertama kali dikirim dan gagal diproses oleh server tujuan.|
|E. Skenario, peristiwa, dan gejala|Pengguna melaporkan bahwa website yummyrecipesforme.com tidak dapat diakses. Halaman tidak pernah berhasil dimuat dan selalu berakhir dengan pesan kesalahan destination port unreachable. Gejala ini muncul secara konsisten pada beberapa pengguna dan dapat direplikasi saat dilakukan pengujian oleh analis.|
|F. Status insiden saat ini|Pada saat analisis dilakukan, insiden masih berlangsung. Penanganan sedang dilakukan oleh tim keamanan siber bersama tim teknis penyedia layanan TI. Peran analis difokuskan pada investigasi lalu lintas jaringan serta penyampaian temuan awal untuk proses eskalasi dan tindak lanjut.|
|G. Informasi hasil investigasi|Analisis log tcpdump menunjukkan bahwa permintaan DNS dikirim menggunakan protokol UDP ke server DNS melalui port 53. Server tidak memberikan respons DNS dan justru membalas dengan pesan ICMP udp port 53 unreachable. Permintaan tersebut terjadi berulang kali dengan hasil yang sama. Kondisi ini membuat browser tidak pernah mendapatkan alamat IP tujuan, sehingga koneksi ke website gagal sejak tahap awal.|
|H. Dugaan akar penyebab|Akar masalah mengarah pada gangguan layanan DNS di sisi server tujuan. Kemungkinan penyebab meliputi layanan DNS yang tidak aktif, kesalahan konfigurasi jaringan, atau aturan firewall yang memblokir akses ke port 53. Pada beberapa kondisi, gangguan ini juga bisa dipicu oleh serangan yang menargetkan layanan DNS.|
|I. Langkah selanjutnya|Langkah berikutnya difokuskan pada pemulihan layanan DNS dan pencegahan kejadian serupa. Tim teknis perlu memastikan layanan DNS berjalan normal pada port 53 serta meninjau konfigurasi firewall agar lalu lintas DNS tidak terblokir. Pengujian konektivitas DNS dilakukan dari beberapa titik jaringan untuk memastikan masalah tidak bersifat lokal. Pemantauan lalu lintas DNS juga perlu ditingkatkan agar gangguan dapat terdeteksi lebih dini. Setelah perbaikan diterapkan, akses website diuji kembali untuk memastikan layanan sudah kembali stabil.|

---

## 🏁 Conclusion <a name="conclusion">

Hasil analisis lalu lintas jaringan menunjukkan bahwa insiden ini dipicu oleh kegagalan layanan DNS yang tidak dapat menerima permintaan pada port 53. Setiap permintaan DNS yang dikirim dari client melalui protokol UDP selalu dibalas dengan pesan ICMP destination port unreachable. Kondisi ini membuat proses resolusi domain gagal dan akses ke website terhenti.

Insiden ini memperlihatkan bahwa gangguan pada satu layanan inti seperti DNS bisa langsung memengaruhi ketersediaan layanan web secara keseluruhan. Analisis paket menggunakan tcpdump membantu mengidentifikasi protokol dan layanan yang terdampak dengan jelas. Temuan ini dapat menjadi pijakan bagi tim teknis untuk melakukan perbaikan serta mencegah kejadian serupa di kemudian hari.
