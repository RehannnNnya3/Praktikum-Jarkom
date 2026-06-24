# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 13 — Ethernet dan ARP

---

## 1. Tujuan Praktikum
1. Mahasiswa dapat menginvestigasi cara kerja **Ethernet** dan **ARP** menggunakan Wireshark.

## 2. Dasar Teori
**Ethernet** adalah protokol *link layer* dominan pada LAN kabel. Frame Ethernet (Ethernet II) membungkus datagram IP dan memiliki struktur:

| Field | Ukuran | Keterangan |
|---|---|---|
| Destination MAC | 6 byte | Alamat fisik tujuan |
| Source MAC | 6 byte | Alamat fisik sumber |
| Type | 2 byte | Jenis payload (mis. `0x0800` = IPv4, `0x0806` = ARP) |
| Data (payload) | 46–1500 byte | Datagram IP / pesan ARP |
| CRC (FCS) | 4 byte | Deteksi error |

**ARP (Address Resolution Protocol)** — RFC 826 — memetakan **alamat IP → alamat MAC** pada satu subnet. Host menyimpan hasil pemetaan di **ARP cache**. Perlu dibedakan:
- **Perintah `arp`** → untuk melihat/memanipulasi isi cache ARP.
- **Protokol ARP** → mendefinisikan format & makna pesan ARP Request/Reply.

ARP Request dikirim secara **broadcast** ("siapa pemilik IP X?"), ARP Reply dikirim **unicast** ("saya, dengan MAC Y").

## 3. Alat dan Bahan
- Wireshark
- Command Prompt / Terminal (perintah `arp`)
- Browser web

---

## 4. Bagian 1 — Menangkap & Menganalisis Frame Ethernet

**Langkah:**
1. Kosongkan cache browser.
2. Mulai capture Wireshark.
3. Buka URL: `http://gaia.cs.umass.edu/wireshark-labs/HTTP-ethereal-lab-file3.html` (menampilkan Bill of Rights AS yang panjang).
4. Hentikan capture. Temukan nomor paket **HTTP GET** dari komputer Anda.
5. Agar fokus ke link layer: **Analyze → Enabled Protocols**, hapus centang **IP**, klik OK. Wireshark kini hanya menampilkan info protokol di bawah IP (Ethernet).


**Pertanyaan & Jawaban tipikal (frame yang membawa HTTP GET):**

| Pertanyaan | Jawaban |
|---|---|
| Alamat MAC sumber (komputer Anda)? | MAC NIC Anda, mis. `00:1a:2b:3c:4d:5e` |
| Alamat MAC tujuan? | MAC **gateway/router** (bukan MAC server gaia), karena tujuan berada di jaringan lain |
| Nilai field Type pada frame Ethernet? | `0x0800` (IPv4) |
| Berapa byte sejak awal frame Ethernet hingga karakter ASCII "G" pada "GET"? | Header Ethernet 14 byte + header IP 20 byte + header TCP 20 byte = **54 byte** (sebelum data HTTP) |
| MAC tujuan pada frame berisi HTTP reply dari gaia? | MAC **router/gateway** Anda (sumber frame dari sisi lokal) |

**Catatan:** Karena gaia.cs.umass.edu berada di jaringan berbeda, frame dikirim ke **alamat MAC gateway**, bukan MAC server tujuan. MAC tujuan akhir hanya digunakan jika berada di subnet yang sama.

---

## 5. Bagian 2 — Address Resolution Protocol (ARP)

### 5.1 Caching ARP
Melihat isi cache ARP:
```
arp -a          # Windows / Linux / Mac (lihat seluruh entri)
```
Mengosongkan cache ARP (perlu hak admin/root):
```
arp -d *        # menghapus semua entri (Windows: arp -d *)
```

### 5.2 Mengamati Aksi ARP
**Langkah:**
1. Kosongkan cache ARP (`arp -d *`) dan cache browser.
2. Mulai capture Wireshark.
3. Buka URL: `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-lab-file3.html`.
4. Hentikan capture. Sembunyikan protokol IP (**Analyze → Enabled Protocols**, hapus centang IP).
5. Cari pesan **ARP** (umumnya dua frame pertama berisi ARP Request & Reply).


**Pertanyaan & Jawaban tipikal:**

| Pertanyaan | Jawaban |
|---|---|
| Nilai field Type Ethernet untuk frame ARP? | `0x0806` |
| Alamat tujuan pada frame ARP Request? | **Broadcast** `ff:ff:ff:ff:ff:ff` |
| ARP Request bertanya tentang apa? | Meminta MAC pemilik suatu alamat IP (mis. IP gateway): *"Who has 192.168.1.1? Tell 192.168.1.x"* |
| Opcode ARP Request vs Reply? | Request = **1**, Reply = **2** |
| Field pada pesan ARP? | Hardware type, Protocol type, Hardware size, Protocol size, Opcode, Sender MAC/IP, Target MAC/IP |
| Bagaimana host tahu MAC tujuan setelah Reply? | ARP Reply (unicast) berisi **Sender MAC** = MAC yang dicari, lalu disimpan ke cache |

---

## 6. Analisis
- Ethernet bekerja di lapisan link dengan pengalamatan **MAC 6-byte**; field Type menentukan protokol payload (`0x0800` IPv4, `0x0806` ARP).
- Untuk tujuan di luar subnet, frame diarahkan ke **MAC gateway**, sementara alamat IP tujuan tetap IP server akhir — memperlihatkan perbedaan peran alamat link layer (hop-by-hop) vs network layer (end-to-end).
- ARP menjembatani lapisan jaringan dan link: tanpa MAC tujuan, frame tidak dapat dikirim. ARP Request broadcast memastikan seluruh host di subnet menerima pertanyaan, hanya pemilik IP yang membalas (unicast).

## 7. Kesimpulan
1. Frame Ethernet membungkus datagram IP dengan alamat MAC sumber/tujuan dan field Type penanda protokol payload.
2. Untuk komunikasi lintas-subnet, MAC tujuan frame adalah MAC gateway, bukan MAC host tujuan akhir.
3. ARP memetakan IP→MAC melalui pasangan Request (broadcast, opcode 1) dan Reply (unicast, opcode 2), dan hasilnya disimpan di ARP cache.

