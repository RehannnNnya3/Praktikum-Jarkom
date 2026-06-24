# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 4 — DNS (Domain Name System)
---

## 1. Tujuan Praktikum
1. Mahasiswa dapat menginvestigasi cara kerja DNS menggunakan Wireshark.

## 2. Dasar Teori
Domain Name System (DNS) memiliki peran penting dalam infrastruktur internet: ia **mentranslasikan nama host menjadi alamat IP**. Peran sisi klien DNS relatif sederhana — klien hanya mengirimkan *query* (permintaan) ke server DNS lokal dan menerima *response* (balasan). Di "balik layar", server DNS yang tersusun secara hirarkis (root → TLD → authoritative) saling berkomunikasi untuk menyelesaikan permintaan klien secara **rekursif** maupun **iteratif**.

Beberapa tool yang digunakan:
- **`nslookup`** — mengirim query DNS ke server DNS tertentu dan menampilkan hasilnya. Sintaks umum:
  ```
  nslookup -option1 -option2 host-to-find dns-server
  ```
  Jika `dns-server` tidak diisi, query dikirim ke *default* server DNS lokal.
- **`ipconfig`** (Windows) / **`ifconfig`** (Linux/Unix) — menampilkan dan mengelola konfigurasi TCP/IP serta cache DNS host.

## 3. Alat dan Bahan
- Wireshark
- Command Prompt (Windows) / Terminal (Linux/Mac)
- Browser web
- Koneksi internet (atau *trace file* `dns-ethereal-trace-2` dari paket wireshark-traces.zip)

---

## 4. Langkah Percobaan & Hasil

### 4.1 Pengenalan `nslookup` (Uji Mandiri)

**1. Mendapatkan alamat IP server web di Asia.**
```
nslookup www.ust.hk
```
Contoh hasil:
```
Server:  [DNS lokal Anda]
Address: [IP DNS lokal]

Non-authoritative answer:
Name:    www.ust.hk
Address: 143.89.14.2     <-- alamat IP server (contoh)
```
> Alamat IP server tersebut: **143.89.14.2** 

**2. Mengetahui server DNS otoritatif untuk universitas di Eropa.**
```
nslookup -type=NS cam.ac.uk
```
Contoh hasil menampilkan beberapa *nameserver*, mis. `auth0.dns.cam.ac.uk`, `sns-pb.isc.org`, dll.

**3. Mencari informasi server email Yahoo! Mail melalui salah satu server di nomor 2.**
```
nslookup -type=MX yahoo.com [nameserver_dari_nomor_2]
```
Contoh hasil (record MX):
```
yahoo.com  MX preference = 1, mail exchanger = mta5.am0.yahoodns.net
yahoo.com  MX preference = 1, mail exchanger = mta6.am0.yahoodns.net
yahoo.com  MX preference = 1, mail exchanger = mta7.am0.yahoodns.net
```
> Alamat IP-nya dapat diperoleh dengan `nslookup mta5.am0.yahoodns.net` (contoh: `87.248.103.x`).

---

### 4.2 `ipconfig` untuk DNS
```
ipconfig /all          # menampilkan info TCP/IP (IP, DNS server, adapter)
ipconfig /displaydns   # menampilkan record DNS yang tersimpan + sisa TTL
ipconfig /flushdns     # menghapus seluruh cache DNS dan memuat ulang dari file host
```

---

### 4.3 Tracing DNS dengan Wireshark — Penjelajahan Web (`www.ietf.org`)

**Langkah:**
1. `ipconfig /flushdns` untuk mengosongkan cache DNS.
2. Kosongkan cache browser.
3. Buka Wireshark, pasang filter `ip.addr == <IP_Anda>`.
4. Mulai capture, lalu kunjungi `http://www.ietf.org`.
5. Hentikan capture.


**Jawaban Pertanyaan:**

| No | Pertanyaan | Jawaban |
|---|---|---|
| 1 | Pesan DNS dikirim melalui UDP atau TCP? | **UDP** |
| 2 | Port tujuan query? Port sumber balasan? | Port tujuan query = **53**; port sumber balasan = **53** |
| 3 | IP tujuan query = IP server DNS lokal? Apakah sama? | IP tujuan query = alamat *default* server DNS lokal Anda (dari `ipconfig /all`). **Ya, kedua alamat sama.** |
| 4 | "Type" pesan query? Mengandung answers? | Umumnya **Type A** (alamat IPv4). **Tidak** mengandung answers (query hanya berisi pertanyaan). |
| 5 | Berapa banyak answers pada balasan? Isinya? | Bervariasi (mis. **2–3 answers**), berisi record **A** (alamat IP) dan kadang **CNAME**. |
| 6 | IP pada paket TCP SYN sesuai IP pada balasan DNS? | **Ya**, host membuka koneksi TCP ke alamat IP yang baru saja di-*resolve* oleh DNS. |
| 7 | Perlu query DNS baru untuk tiap gambar di halaman? | **Tidak** — jika gambar berasal dari host yang sama dan record masih ada di cache, host tidak perlu mengirim query DNS baru. |

---

### 4.4 Tracing DNS — `nslookup www.mit.edu`

**Jawaban Pertanyaan:**

| No | Pertanyaan | Jawaban |
|---|---|---|
| 1 | Port tujuan query? Port sumber balasan? | **53** / **53** |
| 2 | IP tujuan query = default DNS lokal? | **Ya** (kecuali Anda menentukan server lain). |
| 3 | Type query? Mengandung answers? | **Type A**; **tidak** mengandung answers. |
| 4 | Berapa answers pada balasan? Isinya? | Beberapa record **A** berisi alamat IP `www.mit.edu` (contoh `104.x.x.x` jika di-*serve* lewat CDN). |

---

### 4.5 Tracing DNS — `nslookup -type=NS mit.edu`

| No | Pertanyaan | Jawaban |
|---|---|---|
| 1 | IP tujuan query = default DNS lokal? | **Ya** |
| 2 | Type query? Mengandung answers? | **Type NS**; **tidak** mengandung answers. |
| 3 | Nama server MIT pada balasan? Diberi alamat IP? | Balasan berisi nama *authoritative nameserver* MIT (mis. `bitsy.mit.edu`, `use2.mit.edu`, `usw2.mit.edu`). Server DNS lokal biasanya **juga** menyertakan alamat IP-nya secara "cuma-cuma" di bagian *Additional records*. |

---

### 4.6 Tracing DNS — `nslookup www.aiit.or.kr bitsy.mit.edu`

| No | Pertanyaan | Jawaban |
|---|---|---|
| 1 | IP tujuan query = default DNS lokal? | **Tidak** — query dikirim langsung ke `bitsy.mit.edu`, bukan ke DNS lokal. |
| 2 | Type query? Mengandung answers? | **Type A**; **tidak** mengandung answers. |
| 3 | Berapa answers pada balasan? Isinya? | Berisi record **A** alamat IP `www.aiit.or.kr` (server web di Korea). |

---

## 5. Analisis
- DNS menggunakan **UDP port 53** karena query/response berukuran kecil dan mengutamakan kecepatan; *overhead* TCP (three-way handshake) tidak diperlukan untuk transaksi singkat.
- Query DNS hanya berisi **pertanyaan** (jumlah answers = 0), sedangkan balasan membawa satu atau lebih *resource record*.
- Opsi `-type` menentukan jenis record yang diminta: **A** (IPv4), **AAAA** (IPv6), **NS** (nameserver), **MX** (mail exchanger), dst. Tanpa `-type`, default-nya adalah record **A**.
- Saat server DNS tujuan ditentukan secara eksplisit (`nslookup host dns-server`), query **tidak** melewati DNS lokal melainkan langsung ke server tersebut.

## 6. Kesimpulan
1. DNS berfungsi menerjemahkan nama host ke alamat IP melalui pertukaran query–response berbasis **UDP port 53**.
2. Tool `nslookup` memungkinkan pengiriman query manual ke server DNS tertentu dengan berbagai tipe record, sedangkan `ipconfig` digunakan untuk melihat/mengelola konfigurasi dan cache DNS host.
3. Melalui Wireshark, dapat diamati bahwa setiap *resolve* nama menghasilkan satu pasangan query–response, dan host langsung membuka koneksi TCP ke IP hasil resolusi DNS.

