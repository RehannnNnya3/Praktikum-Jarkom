# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 10 — IP (Internet Protocol)

---

## 1. Tujuan Praktikum
1. Mahasiswa dapat menginvestigasi cara kerja protokol IP menggunakan Wireshark.

## 2. Dasar Teori
Modul ini mempelajari protokol IP, fokus pada **datagram IPv4** dan **IPv6**, melalui tiga bagian:
1. **IPv4 Dasar** — menganalisis datagram IPv4 yang dikirim/diterima oleh `traceroute`.
2. **Fragmentasi** — mengamati datagram besar yang dipecah menjadi beberapa fragmen.
3. **IPv6** — sekilas datagram IPv6.

**Cara kerja `traceroute`:** mengirim datagram dengan field **TTL** (Time-To-Live) bertahap (1, 2, 3, …). Setiap router mengurangi TTL sebesar 1; jika TTL mencapai 0, router mengembalikan pesan **ICMP Type 11 (TTL exceeded)** ke pengirim. Dengan demikian, pengirim dapat mengetahui alamat IP setiap router di sepanjang jalur dari alamat sumber pesan ICMP tersebut.

- **Linux/MacOS:** `traceroute` mengirim segmen **UDP**, ukuran datagram bisa diatur, mis. `traceroute gaia.cs.umass.edu 3000`.
- **Windows:** `tracert` mengirim **ICMP Echo Request**, ukuran tidak dapat diubah (sehingga fragmentasi sulit dipaksa dari Windows).

## 3. Alat dan Bahan
- Wireshark
- Command Prompt / Terminal (`tracert` / `traceroute`)
- *Trace file*: `ip-wireshark-trace1-1.pcapng`, `ip-wireshark-trace2-1.pcapng`

## 4. Langkah Percobaan
1. Mulai capture Wireshark.
2. Jalankan dua perintah traceroute ke `gaia.cs.umass.edu`: pertama 56 byte, kedua 3000 byte (Linux/Mac). Pada Windows: `tracert gaia.cs.umass.edu`.
3. Hentikan capture.
4. Gunakan filter tampilan `udp || icmp` (Linux/Mac) atau `icmp` (Windows).


---

## 5. Hasil dan Pembahasan

### 5.1 Bagian 1 — IPv4 Dasar

Filter untuk melihat segmen UDP dari komputer Anda: `ip.src==<IP_Anda> && ip.dst==128.119.245.12 && udp && !icmp`.
Filter untuk paket ICMP balasan: `ip.dst==<IP_Anda> && icmp`.

**Analisis header datagram IPv4 (paket pertama traceroute):**

| Field IP | Contoh Nilai | Keterangan |
|---|---|---|
| Version | 4 | IPv4 |
| Header length | 20 byte | Tanpa option |
| Upper-layer protocol | 17 (UDP) / 1 (ICMP) | Bergantung OS |
| Source IP | `192.168.86.61` | Alamat komputer Anda |
| Destination IP | `128.119.245.12` | gaia.cs.umass.edu |
| TTL | 1, 2, 3, … | Bertambah tiap seri probe |
| Identification | berubah tiap paket | Pengenal datagram |

**Pertanyaan & Jawaban tipikal:**
- **Alamat IP komputer Anda?** → lihat Source IP (`ip.src`), mis. `192.168.86.61`.
- **Protokol upper-layer di datagram IP?** → **UDP (17)** untuk Linux/Mac, **ICMP (1)** untuk Windows.
- **Berapa byte header IP?** → **20 byte**. **Berapa byte payload?** → Total length − 20.
- **Apakah datagram terfragmentasi?** → Untuk paket kecil **tidak** (flag *More fragments* = 0, Fragment offset = 0).
- **Field IP yang selalu berubah antar datagram?** → **Identification** dan **TTL** (serta Header checksum karena TTL berubah).
- **Field IP yang tetap konstan?** → Version, Header length, Source IP, Destination IP, Upper-layer protocol.
- **Pola pada field Identification paket ICMP TTL-exceeded balasan?** → Identification ditentukan oleh masing-masing router, sehingga **tidak berpola** seragam.

### 5.2 Bagian 2 — Fragmentasi (datagram 3000 byte)

Datagram 3000 byte melebihi MTU Ethernet (1500 byte) sehingga **dipecah menjadi beberapa fragmen**.

**Pertanyaan & Jawaban tipikal:**
- **Apakah datagram pertama difragmentasi?** → **Ya**. Penanda: flag **More fragments = 1**, Fragment offset = 0.
- **Field yang menunjukkan fragmentasi?** → **Flags (More fragments)** dan **Fragment offset**.
- **Fragmen pertama:** Fragment offset = **0**, More fragments = **1**, panjang ≈ 1480 byte data.
- **Fragmen kedua:** Fragment offset = **185** (185 × 8 = 1480 byte), More fragments bisa 1 atau 0 (jika fragmen terakhir).
- **Field yang berubah antar fragmen?** → **Fragment offset**, **Flags (MF)**, **Total length**, **Header checksum**.
- **Field yang tetap?** → **Identification** (sama untuk semua fragmen dari satu datagram asli), Source/Destination IP.
- **Berapa fragmen dibuat dari datagram 3000 byte?** → Sekitar **3 fragmen** (1480 + 1480 + sisa).


### 5.3 Bagian 3 — IPv6

Trace `ip-wireshark-trace2-1.pcapng` dihasilkan saat membuka `youtube.com` (mendukung IPv6). Paket ke-20 (t≈3.814489) adalah **query DNS tipe AAAA** (resolusi nama ke alamat **IPv6**).

**Pengamatan header IPv6:**

| Field IPv6 | Keterangan |
|---|---|
| Version | 6 |
| Traffic class / Flow label | QoS & identifikasi aliran |
| Payload length | Panjang payload |
| Next header | Protokol berikutnya (mis. UDP/TCP) |
| Hop limit | Padanan TTL pada IPv4 |
| Source / Destination Address | Alamat 128-bit |

- Perbedaan utama IPv6 vs IPv4: alamat **128-bit** (vs 32-bit), header lebih sederhana, **tidak ada fragmentasi oleh router** (fragmentasi hanya di host sumber), dan **tidak ada header checksum**.

---

## 6. Kesimpulan
1. `traceroute`/`tracert` memanfaatkan field **TTL** dan pesan **ICMP TTL-exceeded** untuk memetakan jalur paket antar router.
2. Pada header IPv4, field **Identification** dan **TTL** berubah antar datagram, sedangkan alamat sumber/tujuan tetap.
3. Datagram yang melebihi MTU difragmentasi; fragmen-fragmen berbagi **Identification** yang sama tetapi berbeda **Fragment offset** dan **flag More fragments**.
4. IPv6 menggunakan alamat 128-bit dengan header yang lebih ringkas dibanding IPv4.

