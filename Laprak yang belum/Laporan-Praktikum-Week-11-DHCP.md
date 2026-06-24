# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 11 — DHCP (Dynamic Host Configuration Protocol)

---

## 1. Tujuan Praktikum
1. Mahasiswa dapat menginvestigasi cara kerja protokol DHCP menggunakan Wireshark.

## 2. Dasar Teori
**DHCP (Dynamic Host Configuration Protocol)** digunakan secara luas pada LAN kabel/nirkabel di perusahaan, kampus, dan rumah untuk **menetapkan alamat IP secara dinamis** ke host serta mengonfigurasi informasi jaringan lain (subnet mask, gateway, DNS server).

DHCP bekerja melalui empat pesan yang dikenal dengan akronim **DORA**:

| Pesan | Arah | Fungsi |
|---|---|---|
| **DISCOVER** | Client → Server (broadcast) | Klien mencari server DHCP |
| **OFFER** | Server → Client | Server menawarkan alamat IP |
| **REQUEST** | Client → Server (broadcast) | Klien meminta alamat yang ditawarkan |
| **ACK** | Server → Client | Server mengonfirmasi penyewaan (*lease*) |

DHCP berjalan di atas **UDP**: server di **port 67**, klien di **port 68**.

## 3. Alat dan Bahan
- Wireshark
- Command Prompt / Terminal
- *Trace file*: `dhcp-wireshark-trace1-1.pcapng`

## 4. Langkah Percobaan

**Pada PC (Windows):**
1. Buka Command Prompt, jalankan:
   ```
   ipconfig /release      # melepaskan alamat IP saat ini
   ```
2. Mulai capture di Wireshark.
3. Jalankan:
   ```
   ipconfig /renew        # meminta alamat IP baru dari server DHCP
   ```
4. Tunggu beberapa detik, hentikan capture.
5. Pasang filter tampilan `dhcp` (atau `bootp` pada Wireshark lama).

**Pada Mac:** `sudo ipconfig set en0 none` lalu `sudo ipconfig set en0 dhcp`.
**Pada Linux:** `sudo ip addr flush en0; sudo dhclient -r` lalu `sudo dhclient en0`.

---

## 5. Hasil dan Pembahasan

**Pengamatan empat pesan DHCP (DORA):**

| Pesan | Source IP | Destination IP | Keterangan |
|---|---|---|---|
| DISCOVER | `0.0.0.0` | `255.255.255.255` | Broadcast — klien belum punya IP |
| OFFER | IP server DHCP | `255.255.255.255` (atau IP yang ditawarkan) | Tawaran alamat |
| REQUEST | `0.0.0.0` | `255.255.255.255` | Broadcast — meminta alamat |
| ACK | IP server DHCP | IP yang diberikan | Konfirmasi lease |

**Pertanyaan & Jawaban tipikal:**

1. **DHCP berjalan di atas UDP atau TCP?** → **UDP** (server port **67**, klien port **68**).
2. **DHCP message type / opcode pesan DISCOVER?** → Message type = **1 (Discover)**, BOOTP opcode = **1 (Boot Request)**.
3. **Transaction ID (xid) pada Discover?** → Nilai acak, mis. `0x1535f43c`. **Transaction ID pada Offer, Request, ACK?** → **Sama** dengan Discover (mengikat satu transaksi DORA).
4. **Alamat IP yang ditawarkan server pada pesan OFFER?** → Lihat field *Your (client) IP address* (yiaddr), mis. `192.168.1.105`.
5. **Berapa lama lease time pada pesan ACK?** → Lihat option *IP Address Lease Time*, mis. `86400` detik (1 hari).
6. **Mengapa Discover & Request dikirim broadcast (255.255.255.255)?** → Karena klien **belum memiliki alamat IP** dan **belum tahu** alamat server DHCP, sehingga pesan disiarkan ke seluruh subnet.
7. **Alamat MAC pada field Client hardware address?** → Alamat MAC kartu jaringan klien (mis. `00:1a:2b:3c:4d:5e`).
8. **Nilai opsi yang dibawa server (subnet mask, router, DNS)?** → Pada OFFER/ACK terdapat option **Subnet Mask**, **Router (gateway)**, dan **Domain Name Server**.

---

## 6. Analisis
- Urutan **DORA** memastikan negosiasi dua arah: server menawarkan, klien memilih & meminta, server mengonfirmasi. Hal ini juga menangani kasus banyak server DHCP dalam satu jaringan (klien memilih satu OFFER).
- Penggunaan broadcast pada Discover/Request wajar karena klien belum punya identitas IP; **Transaction ID** yang konsisten mengaitkan keempat pesan sebagai satu transaksi.
- DHCP menggunakan UDP karena bersifat sederhana, ringan, dan kompatibel dengan broadcast (TCP tidak mendukung broadcast).

## 7. Kesimpulan
1. DHCP menetapkan alamat IP dan parameter jaringan secara dinamis melalui empat pesan **Discover–Offer–Request–ACK** berbasis **UDP (port 67/68)**.
2. Keempat pesan dalam satu proses berbagi **Transaction ID** yang sama.
3. Pesan Discover dan Request dikirim secara broadcast karena klien belum memiliki/mengetahui alamat IP.

## 8. Daftar Pustaka
- J.F. Kurose & K.W. Ross, *Computer Networking: A Top-Down Approach*, 8th ed.
- Modul Praktikum Jaringan Komputer, Laboratorium Informatika, Universitas Telkom, 2025/2026.
