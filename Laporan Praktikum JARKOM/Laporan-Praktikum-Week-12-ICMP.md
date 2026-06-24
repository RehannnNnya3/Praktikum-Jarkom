# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 12 — ICMP dan Asistensi Tugas Besar

---

## 1. Tujuan Praktikum
1. Mahasiswa dapat menginvestigasi cara kerja protokol ICMP menggunakan Wireshark.
2. Mahasiswa dapat membuat program **ICMP Pinger**.
3. Melakukan **asistensi** dan laporan progress pengerjaan Tugas Besar.

## 2. Dasar Teori
**ICMP (Internet Control Message Protocol)** digunakan host dan router untuk saling bertukar informasi kontrol dan pesan kesalahan jaringan. ICMP berjalan **di atas IP** (nomor protokol IP = **1**).

Dua program yang memanfaatkan ICMP:
- **Ping** — memverifikasi apakah host aktif. Host sumber mengirim **ICMP Echo Request (Type 8, Code 0)**, host tujuan membalas **ICMP Echo Reply (Type 0, Code 0)**.
- **Traceroute** — menemukan jalur paket. Di Windows menggunakan paket **ICMP**; di Unix/Linux menggunakan **UDP** dengan port tujuan yang tidak lazim. Router yang menerima paket dengan TTL=0 mengirim **ICMP TTL-exceeded (Type 11, Code 0)**.

| Pesan ICMP | Type | Code |
|---|---|---|
| Echo Request (Ping) | 8 | 0 |
| Echo Reply (Ping) | 0 | 0 |
| TTL Exceeded (Traceroute) | 11 | 0 |
| Destination Unreachable | 3 | 0–15 |

## 3. Alat dan Bahan
- Wireshark
- Command Prompt (`ping`, `tracert`)
- Python 3 (untuk program ICMP Pinger)

---

## 4. Bagian 1 — ICMP dan Ping

**Langkah:**
1. Buka Command Prompt, jalankan Wireshark & mulai capture.
2. Jalankan: `ping -n 10 www.ust.hk` (mengirim 10 pesan ping ke host di benua lain).
3. Setelah selesai, hentikan capture; pasang filter `icmp`.

**Hasil & Pembahasan:**
- Daftar paket menampilkan **20 paket**: 10 Echo Request + 10 Echo Reply.
- Datagram IP yang membungkus ICMP memiliki **Protocol = 1** (ICMP).
- Paket Ping pertama (request) bertipe **Type 8, Code 0** (Echo Request).
- Paket ICMP berisi: **Checksum**, **Identifier**, **Sequence number**, dan **Data**.
- Balasan bertipe **Type 0, Code 0** (Echo Reply), dengan Identifier sama dan Sequence number berurutan.

**Pertanyaan & Jawaban tipikal:**
| Pertanyaan | Jawaban |
|---|---|
| Alamat IP sumber & tujuan paket ICMP? | Sumber = IP privat Anda (di belakang NAT, mis. `192.168.x.x`); tujuan = IP web server tujuan |
| Nomor protokol pada datagram IP? | **1** (ICMP) |
| Type & Code Echo Request? | Type **8**, Code **0** |
| Field pada paket ICMP Ping? | Type, Code, Checksum, Identifier, Sequence number, Data |
| Type & Code Echo Reply? | Type **0**, Code **0** |
| Apakah Sequence number bertambah? | **Ya**, naik 1 untuk tiap request berikutnya |

---

## 5. Bagian 2 — ICMP dan Traceroute

**Langkah:**
1. Mulai capture Wireshark.
2. Jalankan: `tracert www.inria.fr` (host di Eropa).
3. Hentikan capture saat selesai.

**Hasil & Pembahasan:**
- Untuk setiap nilai TTL, dikirim **3 paket probe**; tampilan tracert menunjukkan **RTT** tiap probe dan alamat IP (kadang nama) router yang membalas.
- Pada Windows, probe berupa **ICMP Echo Request**; router perantara membalas **ICMP TTL-exceeded (Type 11, Code 0)**.
- Paket kesalahan ICMP **TTL-exceeded berisi lebih banyak field** daripada Echo Reply Ping — yaitu menyertakan **header IP + 8 byte pertama** datagram asli yang memicu error.

**Pertanyaan & Jawaban tipikal:**
| Pertanyaan | Jawaban |
|---|---|
| Type & Code paket ICMP error dari router? | Type **11** (TTL exceeded), Code **0** |
| Apa isi tambahan pada paket ICMP error? | Salinan **header IP** + **8 byte pertama** payload datagram yang menyebabkan error |
| Bagaimana nilai TTL pada probe berturut-turut? | Bertambah: 1, 2, 3, … hingga mencapai tujuan |
| Berapa router (hop) hingga tujuan? | Sesuai jumlah baris pada output tracert (mis. 12–18 hop) |

---

## 6. Bagian 3 — Program ICMP Pinger (Python)

Program mengirim ICMP Echo Request dan mengukur RTT balasan.

```python
from socket import *
import os, sys, struct, time, select

ICMP_ECHO_REQUEST = 8

def checksum(data):
    csum = 0
    countTo = (len(data) // 2) * 2
    for count in range(0, countTo, 2):
        thisVal = data[count+1] * 256 + data[count]
        csum = (csum + thisVal) & 0xffffffff
    if countTo < len(data):
        csum = (csum + data[-1]) & 0xffffffff
    csum = (csum >> 16) + (csum & 0xffff)
    csum = csum + (csum >> 16)
    answer = ~csum & 0xffff
    answer = answer >> 8 | (answer << 8 & 0xff00)
    return answer

def receiveOnePing(mySocket, ID, timeout, destAddr):
    timeLeft = timeout
    while True:
        startedSelect = time.time()
        whatReady = select.select([mySocket], [], [], timeLeft)
        howLongInSelect = time.time() - startedSelect
        if whatReady[0] == []:           # timeout
            return "Request timed out."
        timeReceived = time.time()
        recPacket, addr = mySocket.recvfrom(1024)
        header = recPacket[20:28]
        type, code, checksum_, packetID, sequence = struct.unpack("bbHHh", header)
        if packetID == ID:
            timeSent = struct.unpack("d", recPacket[28:28+struct.calcsize("d")])[0]
            return "RTT = %.2f ms" % ((timeReceived - timeSent) * 1000)
        timeLeft -= howLongInSelect
        if timeLeft <= 0:
            return "Request timed out."

def sendOnePing(mySocket, destAddr, ID):
    myChecksum = 0
    header = struct.pack("bbHHh", ICMP_ECHO_REQUEST, 0, myChecksum, ID, 1)
    data = struct.pack("d", time.time())
    myChecksum = checksum(header + data)
    if sys.platform == 'darwin':
        myChecksum = htons(myChecksum) & 0xffff
    else:
        myChecksum = htons(myChecksum)
    header = struct.pack("bbHHh", ICMP_ECHO_REQUEST, 0, myChecksum, ID, 1)
    packet = header + data
    mySocket.sendto(packet, (destAddr, 1))

def doOnePing(destAddr, timeout):
    icmp = getprotobyname("icmp")
    mySocket = socket(AF_INET, SOCK_RAW, icmp)
    myID = os.getpid() & 0xFFFF
    sendOnePing(mySocket, destAddr, myID)
    delay = receiveOnePing(mySocket, myID, timeout, destAddr)
    mySocket.close()
    return delay

def ping(host, timeout=1):
    dest = gethostbyname(host)
    print("Pinging " + dest + " using Python:")
    while True:
        print(doOnePing(dest, timeout))
        time.sleep(1)

ping("www.google.com")
```
> Catatan: program ini memakai **raw socket** sehingga harus dijalankan dengan hak **administrator/root**.

---

## 8. Kesimpulan
1. ICMP (protokol IP nomor **1**) digunakan oleh Ping (Echo Request Type 8 / Reply Type 0) dan Traceroute (TTL-exceeded Type 11).
2. Paket ICMP error (TTL-exceeded) membawa salinan header IP + 8 byte pertama datagram asli sehingga lebih panjang dari paket Echo.
3. Program ICMP Pinger berhasil mengirim Echo Request dan mengukur RTT menggunakan raw socket.


