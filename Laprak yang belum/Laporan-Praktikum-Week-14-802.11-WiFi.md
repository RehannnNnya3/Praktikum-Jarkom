# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 14 — 802.11 WiFi

---

## 1. Tujuan Praktikum
1. Mahasiswa dapat menginvestigasi cara kerja **WiFi (802.11)** menggunakan Wireshark.

## 2. Dasar Teori
**IEEE 802.11 (WiFi)** adalah protokol *link layer* nirkabel. Berbeda dari Ethernet kabel, frame 802.11 ditangkap "di udara". Frame 802.11 dikelompokkan menjadi tiga jenis:

| Tipe Frame | Fungsi | Contoh |
|---|---|---|
| **Management** | Mengelola koneksi | Beacon, Probe, Association, Authentication |
| **Control** | Mengontrol akses media | RTS, CTS, ACK |
| **Data** | Membawa data payload | Data frame |

- **Beacon frame** — dipancarkan periodik oleh **AP (Access Point)** untuk mengiklankan keberadaannya (SSID, kemampuan, dsb.).
- **Association Request/Response** — proses host bergabung ke AP (tipe frame 0; subtipe 0 = request, subtipe 1 = response).
- Setiap frame 802.11 memuat alamat MAC (hingga 4 alamat), SSID, channel, dan informasi lapisan fisik.

## 3. Alat dan Bahan
- Wireshark
- *Trace file*: `Wireshark_802_11.pcap` (dari wireshark-traces.zip)

## 4. Skenario Trace
Trace dikumpulkan menggunakan **AirPcap** pada jaringan rumah dengan router/AP **Linksys 802.11g**. Frame ditangkap di **channel 6**. Aktivitas host nirkabel:
- Host telah terasosiasi dengan AP **"30 Munroe St"** saat trace dimulai.
- **t = 24.82** → host membuat HTTP request ke `http://gaia.cs.umass.edu/wireshark-labs/alice.txt` (IP gaia = `128.119.245.12`).
- **t = 32.82** → host membuat HTTP request ke `http://www.cs.umass.edu` (IP `128.119.240.19`).
- **t = 49.58** → host memutus dari "30 Munroe St" dan mencoba menyambung ke **"linksys_ses_24086"** (bukan AP terbuka, gagal).
- **t = 63.0** → host menyerah dan terasosiasi kembali dengan **"30 Munroe St"**.

**Langkah:** Buka `Wireshark_802_11.pcap` via File → Open. Untuk analisis, perluas detail "IEEE 802.11" pada jendela tengah.

---

## 5. Hasil dan Pembahasan

### 5.1 Beacon Frames
Beacon frame diiklankan AP untuk memberitahukan keberadaannya.

| Pertanyaan | Jawaban tipikal |
|---|---|
| Berapa interval pengiriman beacon? | Lihat field *Beacon Interval*, umumnya **0.1024 detik** (≈ 100 ms / 100 TU) |
| SSID yang diiklankan AP? | mis. **"30 Munroe St"** |
| Alamat MAC (BSSID) AP? | mis. `00:16:b6:f7:1d:51` (lihat Source/BSS Id) |
| Data rate yang didukung? | Tercantum di *Supported Rates* (mis. 1, 2, 5.5, 11 Mbps untuk 802.11g) |
| Channel yang digunakan? | **Channel 6** |

### 5.2 Data Transfer
Karena trace dimulai saat host sudah terasosiasi, transfer data terjadi melalui asosiasi 802.11.

| Pertanyaan | Jawaban tipikal |
|---|---|
| Pada t≈24.82, apa frame yang dikirim untuk HTTP request ke alice.txt? | **Data frame** 802.11 membungkus paket TCP/HTTP, ditujukan via AP |
| Alamat tujuan frame data dari host ke gaia? | Tujuan akhir IP `128.119.245.12`, tetapi frame 802.11 dialamatkan ke **MAC AP** (BSSID) |
| Apakah ada frame ACK 802.11? | **Ya** — setiap data frame yang sukses diterima dibalas dengan **ACK (control frame)** |
| Berapa alamat MAC dalam frame data 802.11? | Hingga **3 alamat** terpakai: alamat sumber, tujuan, dan BSSID (AP) |

### 5.3 Association / Disassociation
Asosiasi menggunakan **ASSOCIATE REQUEST** (host→AP, type 0 subtype 0) dan **ASSOCIATE RESPONSE** (AP→host, type 0 subtype 1).

| Pertanyaan | Jawaban tipikal |
|---|---|
| Pada t≈49.58, frame apa yang menandai host melepas AP "30 Munroe St"? | **Disassociation / Deauthentication** frame |
| Saat mencoba menyambung ke "linksys_ses_24086", frame apa yang dikirim? | **Authentication Request** lalu **Association Request** (gagal karena AP tertutup) |
| Mengapa host gagal terhubung ke linksys_ses_24086? | AP **bukan terbuka** (terenkripsi); host tidak punya kredensial sehingga autentikasi/asosiasi gagal |
| Pada t≈63.0, frame apa yang menandai host kembali ke "30 Munroe St"? | **Association Request** diikuti **Association Response (sukses)** |
| Tipe & subtipe Association Request/Response? | Request: type **0**, subtype **0**; Response: type **0**, subtype **1** |

---

## 6. Analisis
- Berbeda dari Ethernet, 802.11 memerlukan tahap **manajemen koneksi** (beacon → authentication → association) sebelum data dapat dipertukarkan.
- Frame data 802.11 dialamatkan ke **AP (BSSID)** terlebih dahulu, bukan langsung ke host tujuan akhir — AP bertindak sebagai relay ke jaringan kabel.
- Mekanisme **ACK per-frame** diperlukan karena media nirkabel rentan error/tabrakan (802.11 memakai CSMA/CA, bukan CSMA/CD).
- Kegagalan asosiasi ke AP tertutup menunjukkan peran autentikasi/keamanan dalam proses bergabung ke jaringan WiFi.

## 7. Kesimpulan
1. Frame 802.11 terdiri dari tipe **management** (beacon, association), **control** (ACK, RTS/CTS), dan **data**.
2. Beacon frame mengiklankan SSID, BSSID, channel, dan data rate AP secara periodik.
3. Proses bergabung ke WiFi melibatkan authentication dan association; frame data dialamatkan via AP dan dikonfirmasi dengan ACK per-frame.
4. Host gagal terhubung ke AP tertutup (`linksys_ses_24086`) karena tidak memiliki kredensial yang sesuai.

## 8. Daftar Pustaka
- J.F. Kurose & K.W. Ross, *Computer Networking: A Top-Down Approach*, 8th ed.
- IEEE Std 802.11, 1999 Edition (R2003).
- Modul Praktikum Jaringan Komputer, Laboratorium Informatika, Universitas Telkom, 2025/2026.
