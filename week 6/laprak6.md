# Laporan Praktikum modul 6

# Tujuan
TCP

1. kita ke link http://gaia.cs.umass.edu/wireshark-labs/alice.txt. lalu tekan ctrl + s untuk save file alice.txt nya
2. buka wireshark dan mulai capture nya

<img width="1919" height="923" alt="Screenshot 2026-04-12 225912" src="https://github.com/user-attachments/assets/fb52aa8f-3007-4e5b-b99a-7db7cd8268f5" />

3. buka link http://gaia.cs.umass.edu/wireshark-labs/TCP-wireshark-file1.html . lalu tekan upload alice.txt

<img width="1919" height="592" alt="Screenshot 2026-04-12 231106" src="https://github.com/user-attachments/assets/1c090b20-81d1-49f3-b514-b371505076ae" />

<img width="1919" height="773" alt="Screenshot 2026-04-12 231000" src="https://github.com/user-attachments/assets/d9c67b09-a38f-44a4-846c-9059656f0bc3" />

4. stop capture diwireshark
<img width="1912" height="819" alt="Screenshot 2026-04-12 231257" src="https://github.com/user-attachments/assets/ef44ee50-676c-4d72-9b5b-5186affe93a3" />

5. filter dengan tcp.port == 80
<img width="1919" height="783" alt="image" src="https://github.com/user-attachments/assets/eae0caea-9490-4cf6-8cc5-76a3d031e93d" />

# jawab soal I
1. alamat IP dan nomor port TCP yang digunakan oleh komputer klien?
   56972
2. Apa alamat IP dari gaia.cs.umass.edu? Pada nomor port berapa ia mengirim dan menerima segmen TCP untuk koneksi ini?
   ip server = 128.119.245.12 , ip address pc = 10.122.2.146 , nomor port = 80 

# TCP Congestion Control in Action

masuk ke statistic klik TCP Stream Graphs, lalu klik Time-Sequence (Stevens)
<img width="697" height="1052" alt="image" src="https://github.com/user-attachments/assets/d0f091b8-6ace-4e2f-b8db-93a4ed55452d" />
<img width="1288" height="829" alt="image" src="https://github.com/user-attachments/assets/2a4e14c4-b0c5-437b-81a2-3a2de4ab0d06" />


