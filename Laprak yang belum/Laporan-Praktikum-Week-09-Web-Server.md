# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 9 — Web Server

---

## 1. Tujuan Praktikum
1. Mahasiswa bisa membuat program **web server sederhana** berbasis TCP socket programming.

## 2. Dasar Teori
Pada modul ini dipelajari dasar pemrograman socket TCP dengan Python: membuat socket, mengikat (*bind*) ke alamat dan port, serta mengirim/menerima paket HTTP, ditambah dasar format header HTTP.

Web server yang dibuat menangani **satu permintaan HTTP dalam satu waktu**. Alur kerjanya:
1. Menerima dan **mengurai** (*parse*) permintaan HTTP dari klien.
2. Mengambil file yang diminta dari sistem file server.
3. Menyusun **pesan respons HTTP** = baris header + isi file.
4. Mengirim respons ke klien.
5. Jika file **tidak ada**, mengirim pesan **`404 Not Found`**.

## 3. Alat dan Bahan
- Python 3
- File HTML (mis. `HelloWorld.html`) diletakkan di direktori yang sama dengan server
- Browser web sebagai klien

---

## 4. Implementasi — Kode Web Server (Skeleton Dilengkapi)

```python
#import socket module
from socket import *
import sys # In order to terminate the program

serverSocket = socket(AF_INET, SOCK_STREAM)

#Prepare a server socket
#Fill in start
serverPort = 6789
serverSocket.bind(('', serverPort))
serverSocket.listen(1)
#Fill in end

while True:
    #Establish the connection
    print('Ready to serve...')
    connectionSocket, addr = serverSocket.accept()      #Fill in start / end
    try:
        message = connectionSocket.recv(1024).decode()  #Fill in start / end
        filename = message.split()[1]
        f = open(filename[1:])
        outputdata = f.readlines()                       #Fill in start / end

        #Send one HTTP header line into socket
        #Fill in start
        connectionSocket.send("HTTP/1.1 200 OK\r\n\r\n".encode())
        #Fill in end

        #Send the content of the requested file to the client
        for i in range(0, len(outputdata)):
            connectionSocket.send(outputdata[i].encode())
        connectionSocket.send("\r\n".encode())
        connectionSocket.close()

    except IOError:
        #Send response message for file not found
        #Fill in start
        connectionSocket.send("HTTP/1.1 404 Not Found\r\n\r\n".encode())
        connectionSocket.send("<html><head></head><body><h1>404 Not Found</h1></body></html>\r\n".encode())
        #Fill in end

        #Close client socket
        #Fill in start
        connectionSocket.close()
        #Fill in end

serverSocket.close()
sys.exit()  #Terminate the program after sending the corresponding data
```

### Penjelasan bagian yang dilengkapi
| Bagian `#Fill in` | Kode | Fungsi |
|---|---|---|
| Prepare server socket | `bind(('', 6789))` + `listen(1)` | Mengikat socket ke port 6789 dan mendengarkan koneksi |
| Establish connection | `serverSocket.accept()` | Menerima koneksi & membuat `connectionSocket` |
| Read message | `connectionSocket.recv(1024).decode()` | Membaca permintaan HTTP dari klien |
| outputdata | `f.readlines()` | Membaca isi file yang diminta |
| HTTP header | `send("HTTP/1.1 200 OK\r\n\r\n")` | Mengirim baris status sukses |
| File not found | `send("HTTP/1.1 404 Not Found\r\n\r\n")` + body | Mengirim respons 404 |
| Close socket | `connectionSocket.close()` | Menutup koneksi klien |

> Catatan: `filename[1:]` membuang karakter `/` di awal path (mis. `/HelloWorld.html` → `HelloWorld.html`).

---

## 5. Cara Menjalankan
1. Letakkan `HelloWorld.html` di direktori yang sama dengan server.
2. Jalankan: `python WebServer.py`.
3. Cari alamat IP host server (mis. `128.238.251.26`).
4. Dari host lain, buka browser ke:
   ```
   http://<IP_server>:6789/HelloWorld.html
   ```
5. Browser akan menampilkan isi `HelloWorld.html`.
6. Coba akses file yang tidak ada → muncul pesan **404 Not Found**.

**Contoh isi `HelloWorld.html`:**
```html
<html>
  <head><title>Hello World</title></head>
  <body><h1>Halo, ini Web Server sederhana berbasis socket TCP!</h1></body>
</html>
```

---

## 6. Latihan Tambahan 

### 6.1 Web Server Multithreaded
Server di atas hanya melayani satu permintaan dalam satu waktu. Dengan modul `threading`, thread utama mendengarkan di port tetap; setiap koneksi klien dilayani di **thread terpisah**.

```python
from socket import *
import threading

def handle_client(connectionSocket, addr):
    try:
        message = connectionSocket.recv(1024).decode()
        filename = message.split()[1]
        f = open(filename[1:])
        outputdata = f.readlines()
        connectionSocket.send("HTTP/1.1 200 OK\r\n\r\n".encode())
        for line in outputdata:
            connectionSocket.send(line.encode())
    except IOError:
        connectionSocket.send("HTTP/1.1 404 Not Found\r\n\r\n".encode())
        connectionSocket.send("<h1>404 Not Found</h1>".encode())
    finally:
        connectionSocket.close()

serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', 6789))
serverSocket.listen(5)
print('Ready to serve...')
while True:
    connectionSocket, addr = serverSocket.accept()
    threading.Thread(target=handle_client, args=(connectionSocket, addr)).start()
```

### 6.2 Klien HTTP Sendiri
Format perintah: `client.py server_host server_port filename`
```python
from socket import *
import sys

serverHost = sys.argv[1]
serverPort = int(sys.argv[2])
filename = sys.argv[3]

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverHost, serverPort))
request = "GET /" + filename + " HTTP/1.1\r\nHost: " + serverHost + "\r\n\r\n"
clientSocket.send(request.encode())

response = b""
while True:
    data = clientSocket.recv(1024)
    if not data:
        break
    response += data
print(response.decode(errors='ignore'))
clientSocket.close()
```

---

## 7. Analisis
- Web server membaca baris pertama permintaan HTTP (`GET /HelloWorld.html HTTP/1.1`) lalu mengambil nama file dari token kedua hasil `split()`.
- Pemisahan header dan body respons HTTP ditandai dengan **CRLF ganda** (`\r\n\r\n`).
- Versi multithreaded meningkatkan konkurensi karena tiap klien dilayani independen.

## 8. Kesimpulan
1. Web server sederhana berhasil dibangun dengan TCP socket: `bind → listen → accept → recv → kirim respons HTTP`.
2. Server mampu mengirim file HTML (status `200 OK`) dan menangani file tidak ditemukan (`404 Not Found`).
3. Dengan `threading`, server dapat ditingkatkan untuk melayani banyak permintaan secara bersamaan.

