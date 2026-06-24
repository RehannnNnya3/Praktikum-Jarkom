# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 7 — Socket Programming: Membuat Aplikasi Jaringan

---

## 1. Tujuan Praktikum
1. Mahasiswa bisa membuat program berbasis socket **UDP**.
2. Mahasiswa bisa membuat program berbasis socket **TCP**.

## 2. Dasar Teori
Aplikasi jaringan umumnya terdiri dari sepasang program — **klien** dan **server** — yang berjalan pada dua *end system* berbeda dan berkomunikasi dengan membaca/menulis ke **socket**. Socket dianalogikan sebagai "pintu": aplikasi berada di satu sisi, protokol *transport layer* di sisi lain.

Keputusan awal pengembang adalah memilih protokol transport:
- **UDP** (`SOCK_DGRAM`) — *connectionless*, tiap paket membawa alamat tujuannya sendiri, tanpa jaminan pengiriman.
- **TCP** (`SOCK_STREAM`) — *connection-oriented*, perlu **three-way handshake**; menyediakan aliran byte yang andal dan berurutan.

**Aplikasi contoh** yang dibuat:
1. Klien membaca satu baris karakter dari keyboard dan mengirimkannya ke server.
2. Server menerima data dan mengubahnya menjadi **huruf besar (uppercase)**.
3. Server mengirim balik data yang sudah dimodifikasi.
4. Klien menerima dan menampilkannya di layar.

Nomor port server yang digunakan: **12000**.

## 3. Alat dan Bahan
- Python 3
- Dua host (atau satu host dengan `localhost`/`127.0.0.1` untuk pengujian)
- Editor teks / IDE

---

## 4. Implementasi Program

### 4.1 Program Socket UDP

#### `UDPClient.py`
```python
from socket import *

serverName = 'hostname'      # ganti dengan IP/hostname server, mis. '127.0.0.1'
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_DGRAM)        # membuat socket UDP

message = input('Input lowercase sentence:')
clientSocket.sendto(message.encode(), (serverName, serverPort))

modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())

clientSocket.close()
```

#### `UDPServer.py`
```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))               # mengikat port 12000 ke socket
print('The server is ready to receive')

while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```

**Penjelasan baris kunci:**
- `socket(AF_INET, SOCK_DGRAM)` → `AF_INET` = IPv4, `SOCK_DGRAM` = socket UDP.
- `sendto(data, (host, port))` → mengirim paket UDP dengan menyertakan alamat tujuan.
- `recvfrom(2048)` → menerima paket (buffer 2048 byte) + alamat pengirim.
- `bind(('', port))` → server mengikat nomor port secara eksplisit agar paket ke port itu terarah ke socket ini.
- Server tidak menentukan port klien — diserahkan ke sistem operasi.

---

### 4.2 Program Socket TCP

#### `TCPClient.py`
```python
from socket import *

serverName = 'servername'    # ganti dengan IP/hostname server, mis. '127.0.0.1'
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_STREAM)       # membuat socket TCP
clientSocket.connect((serverName, serverPort))    # memulai three-way handshake

sentence = input('Input lowercase sentence:')
clientSocket.send(sentence.encode())

modifiedSentence = clientSocket.recv(2048)
print('From Server:', modifiedSentence.decode())

clientSocket.close()
```

#### `TCPServer.py`
```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort))
serverSocket.listen(1)                            # socket penyambut, antrian min. 1
print('The server is ready to receive')

while True:
    connectionSocket, addr = serverSocket.accept()   # socket khusus per-klien
    sentence = connectionSocket.recv(2048).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    connectionSocket.close()
```

**Penjelasan baris kunci (perbedaan dengan UDP):**
- `SOCK_STREAM` → socket TCP.
- `connect((serverName, serverPort))` → klien memulai **three-way handshake** sebelum mengirim data.
- `listen(1)` → server mendengarkan permintaan koneksi (parameter = panjang antrian).
- `accept()` → mengembalikan **`connectionSocket`** baru yang khusus melayani satu klien, terpisah dari **`serverSocket`** (pintu penyambut).
- `send()` / `recv()` → mengalirkan byte langsung lewat koneksi, tanpa menyertakan alamat tujuan per-paket (berbeda dari `sendto`/`recvfrom` di UDP).

---

## 5. Cara Pengujian
1. Jalankan **server** terlebih dahulu: `python UDPServer.py` (atau `TCPServer.py`).
2. Jalankan **klien** di host lain (atau terminal lain): `python UDPClient.py` (atau `TCPClient.py`).
3. Ketik kalimat huruf kecil di klien, lalu Enter.
4. Klien akan menampilkan kalimat yang sudah diubah menjadi huruf besar oleh server.

**Contoh output:**
```
Input lowercase sentence: hello jaringan komputer
HELLO JARINGAN KOMPUTER
```


---

## 6. Analisis — Perbandingan UDP vs TCP

| Aspek | UDP | TCP |
|---|---|---|
| Tipe socket | `SOCK_DGRAM` | `SOCK_STREAM` |
| Koneksi | Connectionless | Connection-oriented (3-way handshake) |
| Pengiriman data | `sendto()` / `recvfrom()` (alamat per paket) | `send()` / `recv()` (aliran byte) |
| Keandalan | Tidak dijamin | Dijamin & berurutan |
| Socket server | Satu socket | `serverSocket` (penyambut) + `connectionSocket` (per klien) |
| Method khusus server | — | `listen()`, `accept()` |

- Pada UDP, server hanya menggunakan satu socket dan setiap paket membawa alamatnya sendiri.
- Pada TCP, `accept()` membuat socket koneksi baru untuk tiap klien sehingga `serverSocket` tetap bebas melayani klien berikutnya.

## 7. Kesimpulan
1. Pemrograman socket UDP lebih sederhana (tanpa handshake), sedangkan TCP memerlukan pembentukan koneksi tetapi menjamin keandalan dan urutan data.
2. Perbedaan kunci kode terletak pada tipe socket (`SOCK_DGRAM` vs `SOCK_STREAM`) dan mekanisme pengiriman (`sendto/recvfrom` vs `connect/send/recv` + `listen/accept`).
3. Aplikasi uppercase sederhana berhasil mendemonstrasikan komunikasi klien–server pada kedua protokol.

