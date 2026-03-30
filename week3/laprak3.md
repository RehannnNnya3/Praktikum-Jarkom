# Laporan Praktikum week 3

## Tujuan 
Konsep IP Addressing, Dasar Routing

## langkah langkah
1. buka wire shark 
2. klik wifi agar wireshark berjalan
3. pergi ke http://gaia.cs.umass.edu/wireshark-labs/HTTPwireshark-file2.html
4. kembali ke wireshark dan filter http saja
<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/54728f60-d777-42d9-8eff-403e519bbba5" />
<img width="1614" height="811" alt="image" src="https://github.com/user-attachments/assets/6428d3ba-4598-4b75-aa01-2bdf8fb4dfc6" />

## LINK 3 HTTP CONDITIONAL GET
5. paste link http://gaia.cs.umass.edu/wireshark-labs/HTTPwireshark-file3.html ke browser
6. capture menggunakan wireshark, lihat responnya lalu filter menggunakan HTTP
<img width="1919" height="1021" alt="Screenshot 2026-03-02 143944" src="https://github.com/user-attachments/assets/cbb86ac0-3bf2-44da-93d9-2eed191203e5" />
<img width="1593" height="775" alt="Screenshot 2026-03-30 212349" src="https://github.com/user-attachments/assets/4eb78fc7-8c2d-4cec-bf96-c047770a2eed" />

## LINK 4 HTML Documents dengan Embedded Objects
7. pergi ke browser lalu ke link https://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file4.html
8. capture pakai wireshark dan filter menggunakan http 
<img width="1908" height="1006" alt="image" src="https://github.com/user-attachments/assets/d1190d3e-553e-4245-a1f3-d4a7be437200" />
<img width="1628" height="860" alt="image" src="https://github.com/user-attachments/assets/f6ff8457-e5c8-417d-a25d-3a4a99d7313e" />

## LINK 5 HTTP Authentication
9. search dibrowser http://gaia.cs.umass.edu/wireshark-labs/protected_pages/HTTP-wireshark-file5.html
10. web akan meminta untuk memasukan username dan passsword 
<img width="1919" height="1024" alt="image" src="https://github.com/user-attachments/assets/f82d57a8-f95c-40f1-a2fb-dc58f9f0111f" />

11. buka wireshark kembali untuk capture link yang di tempelkan ke browser tersebut dengan kembali mengaktifkan filter http
<img width="1606" height="854" alt="image" src="https://github.com/user-attachments/assets/84957acc-ccae-488e-bed7-4aef4c3c2c77" />

12. masukkan password dan username yang benar yaitu username: wireshark-students dan password: network,dan klik sign in. setelahnya, cek apa respon dari dari web
<img width="1621" height="832" alt="image" src="https://github.com/user-attachments/assets/2a88c87b-a6a1-40cc-a682-163ceb2a2921" />

13. nyalakan filter http
<img width="1617" height="858" alt="Screenshot 2026-03-30 214717" src="https://github.com/user-attachments/assets/f2bd2283-4395-4158-a66b-5872b3d10f0d" />

Server membalas dengan pesan HTTP/1.1 200 OK. Artinya, server tersebut berhasil menerima permintaannya, menemukan halamannya, dan berhasil mengirimkannya kembali ke browser.

