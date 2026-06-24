# LAPORAN PRAKTIKUM JARINGAN KOMPUTER
## Modul 5 — UDP (User Datagram Protocol)

---

## 1. Tujuan Praktikum
1. Mahasiswa dapat menginvestigasi cara kerja protokol UDP menggunakan Wireshark.

## 2. Dasar Teori
**UDP (User Datagram Protocol)** adalah protokol *transport layer* yang sederhana dan *connectionless* (tanpa koneksi). Berbeda dengan TCP, UDP **tidak** melakukan handshake, **tidak** menjamin pengiriman, urutan, maupun bebas-duplikat. UDP cocok untuk aplikasi yang mengutamakan kecepatan dan toleran terhadap kehilangan paket (mis. DNS, SNMP, streaming, VoIP).

Header UDP sangat ringkas, hanya **8 byte**, terdiri dari **4 field** masing-masing **2 byte (16 bit)**:

| Field | Ukuran | Keterangan |
|---|---|---|
| Source Port | 2 byte | Nomor port pengirim |
| Destination Port | 2 byte | Nomor port tujuan |
| Length | 2 byte | Panjang total segmen UDP (header + data) |
| Checksum | 2 byte | Deteksi error |

## 3. Alat dan Bahan
- Wireshark
- Koneksi internet (atau *trace file* yang disediakan, mis. ekstrak `http-ethereal-trace-5` dari wireshark-traces.zip)

## 4. Langkah Percobaan
1. Mulai capture di Wireshark.
2. Lakukan aktivitas jaringan agar host mengirim/menerima paket UDP. Tanpa melakukan apa pun pun biasanya akan terekam paket **SNMP** (yang berjalan di atas UDP).
3. Hentikan capture.
4. Pasang filter `udp` agar hanya paket UDP yang ditampilkan.
5. Pilih satu paket UDP, lalu perluas (*expand*) bagian detail header UDP-nya.

---

## 5. Hasil dan Pembahasan (Jawaban Pertanyaan)

**1. Berapa banyak field pada header UDP? Sebutkan namanya.**
> Terdapat **4 field**: **Source Port**, **Destination Port**, **Length**, dan **Checksum**.

**2. Berapa panjang (byte) masing-masing field pada header UDP?**
> Masing-masing field berukuran **2 byte (16 bit)**, sehingga total header UDP = 4 × 2 = **8 byte**.

**3. Nilai pada "Length" menyatakan apa? Verifikasi melalui paket UDP.**
> Field **Length** menyatakan **panjang total segmen UDP** = header (8 byte) + payload (data).
> Verifikasi: jika sebuah paket memiliki payload _N_ byte, maka Length = _N_ + 8. Contoh: payload 32 byte → Length = 40.

**4. Berapa jumlah maksimum byte dalam payload UDP?**
> Field Length berukuran 16 bit → nilai maksimum = 2¹⁶ − 1 = **65.535 byte** untuk seluruh segmen. Karena header = 8 byte, maka payload maksimum = 65.535 − 8 = **65.527 byte**.

**5. Berapa nomor port terbesar yang dapat menjadi port sumber?**
> Field port berukuran 16 bit → port terbesar = 2¹⁶ − 1 = **65.535**.

**6. Berapa nomor protokol untuk UDP (heksadesimal & desimal)?**
> Pada field *Protocol* di header IP, UDP bernilai **17 (desimal)** = **0x11 (heksadesimal)**.

**7. Hubungan nomor port pada pasangan paket UDP (request–reply).**
> Pada pasangan request–reply, nomor port saling **bertukar**: **Source Port** paket pertama menjadi **Destination Port** paket kedua, dan **Destination Port** paket pertama menjadi **Source Port** paket kedua. Hal ini memungkinkan kedua host saling mengarahkan balasan ke soket yang tepat.

---

## 6. Analisis
- Kesederhanaan header UDP (hanya 8 byte) membuat *overhead* sangat kecil dibanding TCP (minimal 20 byte), sehingga UDP efisien untuk transaksi singkat seperti DNS dan SNMP.
- Karena tidak ada mekanisme koneksi/ACK, keandalan (jika dibutuhkan) harus diimplementasikan di lapisan aplikasi.
- Field Length yang mencakup header menjelaskan mengapa payload maksimum (65.527) lebih kecil 8 byte dari nilai Length maksimum (65.535).

## 7. Kesimpulan
1. Header UDP terdiri dari 4 field (Source Port, Destination Port, Length, Checksum), masing-masing 2 byte, total 8 byte.
2. UDP adalah protokol *connectionless* dengan nomor protokol IP = 17 (0x11).
3. Pada komunikasi request–reply, nomor port sumber dan tujuan saling bertukar antar paket.

