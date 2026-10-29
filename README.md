# Mid-Project Jarkom A: Implementasi Jaringan Smart City Kota Surabaya

**Kelompok 10 (K = 10)** | Komunikasi Data dan Jaringan Komputer (ET234306) | Semester Gasal 2026/2027

| Keterangan | Nilai |
|---|---|
| Base network (hasil UTS) | 10.80.0.0/16 |
| Skema yang dipilih | **VLSM** |
| Domain internal | k10.smartcity.lab |
| File Packet Tracer | `K10_VLSM.pkt` |
| Spreadsheet subnetting | `K10_Subnetting_MidProject.xlsx` |

---

## Status pengerjaan

| Soal | Bagian | Status |
|---|---|---|
| 1 | Perancangan topologi | Selesai |
| 2 | Tabel subnetting | Selesai |
| 3 | Static routing | Selesai |
| 4 | DHCP | Belum |
| 5 | NAT | Belum |
| 6 | Tunnel GRE | Belum |
| 7 | Dynamic routing OSPF | Belum |
| 8 | Layanan aplikasi Smart City | Belum |

---

## Alasan memilih VLSM

Kebutuhan host terbesar adalah 11.000 (WiFi Publik Utara), yang memerlukan prefix /18 atau 16.384 alamat.

Pada skema CIDR seluruh subnet dibuat seragam, sehingga keempat belas LAN masing-masing memakai /18:

```
14 x 16.384 = 229.376 alamat
```

Induk 10.80.0.0/16 hanya menyediakan 65.536 alamat, jadi skema CIDR seragam tidak muat di dalamnya dan induknya harus diperlebar menjadi 10.80.0.0/14. Pemborosannya juga besar: LAN Server Balai Kota yang hanya butuh 20 host tetap mendapat 16.382 alamat.

Dengan VLSM, total alamat yang terpakai hanya 61.756 dari 65.536 yang tersedia, atau sekitar 94 persen. Seluruh rancangan tetap muat di dalam induk /16 yang diberikan. Karena itu VLSM dipilih untuk implementasi di Packet Tracer.

Perbandingan lengkap kedua skema, termasuk tabel supernet per router untuk CIDR, ada di file `K10_Subnetting_MidProject.xlsx`.

---

## Soal 1: Perancangan topologi

![Topologi lengkap](img/01-topologi-lengkap.png)

### Perangkat yang dipakai

| Jenis | Jumlah | Model |
|---|---|---|
| Router | 8 | Cisco 2911 |
| Switch | 14 | Cisco 2960 |
| PC | 12 | PC-PT |
| Server | 3 | Server-PT |

### Pertimbangan desain

**Backbone memakai serial, bukan Ethernet.** Router 2911 hanya memiliki 3 port GigabitEthernet bawaan, sedangkan R-PUSAT harus melayani 8 sambungan sekaligus: 1 ke R-ISP, 5 ke router wilayah (L1 sampai L5), dan 2 ke LAN miliknya sendiri. Solusinya, ketiga port Gigabit dipakai untuk LAN, dan seluruh sambungan antar-router memakai port serial dari modul tambahan **HWIC-2T** (2 port serial per modul).

![Modul HWIC-2T di R-PUSAT](img/02-modul-hwic2t-rpusat.png)

Jumlah modul per router: R-PUSAT 3 modul (6 port serial), R-UTARA 2 modul, router lainnya 1 modul.

**Satu switch per LAN.** Tiap LAN diberi switch 2960 tersendiri agar domain broadcast tiap subnet terpisah dan penambahan perangkat di kemudian hari tidak mengubah konfigurasi router.

**Satu perangkat perwakilan per LAN.** Sesuai ketentuan soal, tiap LAN cukup diisi satu perangkat, namun perhitungan subnet tetap memakai jumlah host penuh. LAN yang berisi server (Server Balai Kota, Server dan Storage DC) memakai perangkat Server-PT, sisanya PC-PT.

**Server internet tanpa switch.** SRV-INTERNET (8.8.8.8) disambung langsung ke R-ISP karena merupakan satu-satunya perangkat di segmen tersebut.

**Media link.** Seluruh sambungan router ke switch dan switch ke perangkat memakai kabel Copper Straight-Through, karena menghubungkan perangkat dengan jenis port berbeda. Sambungan antar-router memakai Serial DCE, dengan sisi DCE selalu berada di router yang posisinya lebih dekat ke inti jaringan.

### Penamaan

Seluruh perangkat diganti namanya agar menunjukkan lokasinya, misalnya `R-PUSAT`, `SW-CCTV-KOTA`, `PC-SENSOR-TIMUR`. Tiap LAN juga diberi label di kanvas berisi nama subnet, Network ID beserta prefix, dan jumlah host. Tiap link backbone diberi label berisi nomor link dan network /30 miliknya.

---

## Soal 2: Tabel subnetting

Tabel lengkap ada di `K10_Subnetting_MidProject.xlsx`, terdiri atas enam sheet: Ringkasan, VLSM, Link dan Tambahan, CIDR, Supernet per Router, dan Revisi.

### Ringkasan VLSM

| No | Subnet | Host | Prefix | Network ID | Range Host | Broadcast | Gateway |
|---|---|---|---|---|---|---|---|
| 1 | R-Utara-WiFi | 11.000 | /18 | 10.80.0.0 | 10.80.0.1 - 10.80.63.254 | 10.80.63.255 | 10.80.0.1 |
| 2 | R-Selatan-WiFi | 6.500 | /19 | 10.80.64.0 | 10.80.64.1 - 10.80.95.254 | 10.80.95.255 | 10.80.64.1 |
| 3 | R-DC-CCTV | 6.000 | /19 | 10.80.96.0 | 10.80.96.1 - 10.80.127.254 | 10.80.127.255 | 10.80.96.1 |
| 4 | R-Timur-WiFi | 5.500 | /19 | 10.80.128.0 | 10.80.128.1 - 10.80.159.254 | 10.80.159.255 | 10.80.128.1 |
| 5 | R-Pelabuhan-Sensor | 5.000 | /19 | 10.80.160.0 | 10.80.160.1 - 10.80.191.254 | 10.80.191.255 | 10.80.160.1 |
| 6 | R-Barat-WiFi | 4.500 | /19 | 10.80.192.0 | 10.80.192.1 - 10.80.223.254 | 10.80.223.255 | 10.80.192.1 |
| 7 | R-Pusat-Pegawai | 1.800 | /21 | 10.80.224.0 | 10.80.224.1 - 10.80.231.254 | 10.80.231.255 | 10.80.224.1 |
| 8 | R-Timur-Sensor | 700 | /22 | 10.80.232.0 | 10.80.232.1 - 10.80.235.254 | 10.80.235.255 | 10.80.232.1 |
| 9 | R-Utara-Sensor | 450 | /23 | 10.80.236.0 | 10.80.236.1 - 10.80.237.254 | 10.80.237.255 | 10.80.236.1 |
| 10 | R-Barat-Sekolah | 220 | /24 | 10.80.238.0 | 10.80.238.1 - 10.80.238.254 | 10.80.238.255 | 10.80.238.1 |
| 11 | R-DC-Server | 180 | /24 | 10.80.239.0 | 10.80.239.1 - 10.80.239.254 | 10.80.239.255 | 10.80.239.1 |
| 12 | R-Pelabuhan-Kantor | 120 | /25 | 10.80.240.0 | 10.80.240.1 - 10.80.240.126 | 10.80.240.127 | 10.80.240.1 |
| 13 | R-Selatan-PJU | 90 | /25 | 10.80.240.128 | 10.80.240.129 - 10.80.240.254 | 10.80.240.255 | 10.80.240.129 |
| 14 | R-Pusat-Server | 20 | /27 | 10.80.241.0 | 10.80.241.1 - 10.80.241.30 | 10.80.241.31 | 10.80.241.1 |

Gateway tiap LAN adalah alamat host pertama, sesuai ketentuan soal.

### Link antar-router

| Link | Network | Router A | IP A | Router B | IP B |
|---|---|---|---|---|---|
| L1 | 10.80.241.32/30 | R-PUSAT | 10.80.241.33 | R-DC | 10.80.241.34 |
| L2 | 10.80.241.36/30 | R-PUSAT | 10.80.241.37 | R-UTARA | 10.80.241.38 |
| L3 | 10.80.241.40/30 | R-PUSAT | 10.80.241.41 | R-SELATAN | 10.80.241.42 |
| L4 | 10.80.241.44/30 | R-PUSAT | 10.80.241.45 | R-TIMUR | 10.80.241.46 |
| L5 | 10.80.241.48/30 | R-PUSAT | 10.80.241.49 | R-BARAT | 10.80.241.50 |
| L6 | 10.80.241.52/30 | R-UTARA | 10.80.241.53 | R-PELABUHAN | 10.80.241.54 |
| L7 | 10.80.241.56/30 | R-DC | 10.80.241.57 | R-UTARA | 10.80.241.58 |

L7 adalah link cadangan yang hanya dipasang di file OSPF, sesuai soal 7. Prefix /30 dipakai karena tiap link hanya memerlukan dua alamat host.

### Alamat tambahan dari soal

| Keperluan | Alamat |
|---|---|
| Link R-PUSAT ke R-ISP | 203.0.113.40/30, R-PUSAT .41, R-ISP .42 |
| Server internet | 8.8.8.0/24, server 8.8.8.8, gateway 8.8.8.1 |
| Tunnel GRE | 192.168.10.0/30, R-PUSAT .1, R-PELABUHAN .2 |

### Tabel revisi

Terdapat kesalahan hitung pada lembar jawaban UTS yang sudah diperbaiki dan dicatat. Kesalahan utamanya ada pada baris R-Barat-WiFi: broadcast ditulis 10.80.199.255, padahal blok /19 berisi 8.192 alamat yang setara 32 angka pada oktet ketiga, sehingga broadcast yang benar adalah 10.80.223.255. Angka 199 berasal dari lompatan 8, yang merupakan pola /21. Karena alokasi VLSM bersifat berurutan, kesalahan ini menggeser delapan baris di bawahnya.

Rincian lengkap tiga belas baris revisi beserta alasannya ada di sheet **Revisi** pada file spreadsheet.

---

## Soal 3: Static routing

### Peta interface

| Router | Interface | Alamat | Keterangan |
|---|---|---|---|
| R-ISP | Gi0/0 | 8.8.8.1/24 | LAN server internet |
| R-ISP | Se0/3/0 | 203.0.113.42/30 | Link ke R-PUSAT |
| R-PUSAT | Gi0/0 | 10.80.224.1/21 | LAN Pegawai Balai Kota |
| R-PUSAT | Gi0/1 | 10.80.241.1/27 | LAN Server Balai Kota |
| R-PUSAT | Se0/1/0 | 203.0.113.41/30 | Link ke R-ISP |
| R-PUSAT | Se0/1/1 | 10.80.241.33/30 | L1 ke R-DC |
| R-PUSAT | Se0/2/0 | 10.80.241.37/30 | L2 ke R-UTARA |
| R-PUSAT | Se0/2/1 | 10.80.241.41/30 | L3 ke R-SELATAN |
| R-PUSAT | Se0/3/0 | 10.80.241.45/30 | L4 ke R-TIMUR |
| R-PUSAT | Se0/3/1 | 10.80.241.49/30 | L5 ke R-BARAT |
| R-DC | Gi0/0 | 10.80.96.1/19 | LAN CCTV Kota |
| R-DC | Gi0/1 | 10.80.239.1/24 | LAN Server dan Storage DC |
| R-DC | Se0/3/0 | 10.80.241.34/30 | L1 ke R-PUSAT |
| R-BARAT | Gi0/0 | 10.80.192.1/19 | LAN WiFi Publik Barat |
| R-BARAT | Gi0/1 | 10.80.238.1/24 | LAN Puskesmas dan Sekolah Barat |
| R-BARAT | Se0/3/0 | 10.80.241.50/30 | L5 ke R-PUSAT |
| R-UTARA | Gi0/0 | 10.80.0.1/18 | LAN WiFi Publik Utara |
| R-UTARA | Gi0/1 | 10.80.236.1/23 | LAN Sensor Lalu Lintas Utara |
| R-UTARA | Se0/3/0 | 10.80.241.38/30 | L2 ke R-PUSAT |
| R-UTARA | Se0/2/0 | 10.80.241.53/30 | L6 ke R-PELABUHAN |
| R-TIMUR | Gi0/0 | 10.80.128.1/19 | LAN WiFi Publik Timur |
| R-TIMUR | Gi0/1 | 10.80.232.1/22 | LAN Sensor Banjir Timur |
| R-TIMUR | Se0/3/0 | 10.80.241.46/30 | L4 ke R-PUSAT |
| R-SELATAN | Gi0/0 | 10.80.64.1/19 | LAN WiFi Publik Selatan |
| R-SELATAN | Gi0/1 | 10.80.240.129/25 | LAN PJU Pintar Selatan |
| R-SELATAN | Se0/3/0 | 10.80.241.42/30 | L3 ke R-PUSAT |
| R-PELABUHAN | Gi0/0 | 10.80.160.1/19 | LAN Sensor dan SCADA Pelabuhan |
| R-PELABUHAN | Gi0/1 | 10.80.240.1/25 | LAN Kantor Pelabuhan |
| R-PELABUHAN | Se0/3/0 | 10.80.241.54/30 | L6 ke R-UTARA |

### Alamat statis yang sudah diisi

| Perangkat | IP | Mask | Gateway |
|---|---|---|---|
| SRV-BALAIKOTA | 10.80.241.2 | 255.255.255.224 | 10.80.241.1 |
| SRV-STORAGE-DC | 10.80.239.2 | 255.255.255.0 | 10.80.239.1 |
| SRV-INTERNET | 8.8.8.8 | 255.255.255.0 | 8.8.8.1 |

Seluruh PC sengaja dibiarkan tanpa alamat karena akan memperoleh IP dinamis lewat DHCP (soal 4).

### Konfigurasi interface

Contoh pada R-PUSAT:

```
enable
configure terminal
hostname R-PUSAT
interface GigabitEthernet0/0
 description LAN Pegawai Balai Kota
 ip address 10.80.224.1 255.255.248.0
 no shutdown
exit
interface Serial0/1/0
 description Link ke R-ISP
 ip address 203.0.113.41 255.255.255.252
 clock rate 64000
 no shutdown
exit
end
write memory
```

Perintah `clock rate` diberikan pada tiap interface serial. Hanya sisi DCE yang benar-benar memakainya untuk menentukan kecepatan sinkronisasi link, sedangkan sisi DTE mengabaikannya.

### Rute

Aturan yang dipakai sesuai ketentuan soal: default route hanya mengarah ke router di atasnya, dan hanya R-PUSAT serta R-UTARA yang memiliki static route ke jaringan di bawahnya.

**Router daun**, masing-masing cukup satu default route:

```
R-DC          ip route 0.0.0.0 0.0.0.0 10.80.241.33
R-BARAT       ip route 0.0.0.0 0.0.0.0 10.80.241.49
R-TIMUR       ip route 0.0.0.0 0.0.0.0 10.80.241.45
R-SELATAN     ip route 0.0.0.0 0.0.0.0 10.80.241.41
R-PELABUHAN   ip route 0.0.0.0 0.0.0.0 10.80.241.53
```

R-PELABUHAN mengarah ke R-UTARA, bukan R-PUSAT, karena posisinya berada di bawah R-UTARA.

**R-UTARA**, default ke atas ditambah rute ke dua LAN milik R-PELABUHAN:

```
ip route 0.0.0.0 0.0.0.0 10.80.241.37
ip route 10.80.160.0 255.255.224.0 10.80.241.54
ip route 10.80.240.0 255.255.255.128 10.80.241.54
```

**R-PUSAT**, default ke R-ISP ditambah rute ke seluruh LAN di bawahnya:

```
ip route 0.0.0.0 0.0.0.0 203.0.113.42
ip route 10.80.96.0 255.255.224.0 10.80.241.34
ip route 10.80.239.0 255.255.255.0 10.80.241.34
ip route 10.80.0.0 255.255.192.0 10.80.241.38
ip route 10.80.236.0 255.255.254.0 10.80.241.38
ip route 10.80.160.0 255.255.224.0 10.80.241.38
ip route 10.80.240.0 255.255.255.128 10.80.241.38
ip route 10.80.241.52 255.255.255.252 10.80.241.38
ip route 10.80.64.0 255.255.224.0 10.80.241.42
ip route 10.80.240.128 255.255.255.128 10.80.241.42
ip route 10.80.128.0 255.255.224.0 10.80.241.46
ip route 10.80.232.0 255.255.252.0 10.80.241.46
ip route 10.80.192.0 255.255.224.0 10.80.241.50
ip route 10.80.238.0 255.255.255.0 10.80.241.50
```

Dua LAN milik R-PELABUHAN diarahkan ke R-UTARA karena R-PELABUHAN tidak tersambung langsung ke R-PUSAT.

**R-ISP** tidak diberi rute apa pun ke alamat privat, sesuai syarat soal 5. Akses host internal ke 8.8.8.8 akan ditangani NAT di R-PUSAT.

---

## Bukti pengujian

### Link ISP

![Uji link ISP](img/03-uji-link-isp.png)

### Backbone dari R-PUSAT ke lima router wilayah

![Uji backbone](img/04-uji-backbone-rpusat.png)

### Link L6 dari R-UTARA ke R-PELABUHAN

![Uji link L6](img/05-uji-link-l6.png)

### LAN Server Balai Kota dari R-PUSAT

![Uji LAN R-PUSAT](img/06-uji-lan-rpusat.png)

### Routing lintas wilayah dari R-BARAT

Menguji jangkauan ke LAN Server DC, LAN Kantor Pelabuhan (tiga hop lewat R-PUSAT dan R-UTARA), dan LAN WiFi Selatan.

![Uji routing R-BARAT](img/07-uji-routing-rbarat.png)

### Routing lintas wilayah dari R-PELABUHAN

Menguji arah sebaliknya, dari titik terjauh dalam topologi.

![Uji routing R-PELABUHAN](img/08-uji-routing-rpelabuhan.png)

Seluruh pengujian menghasilkan success rate 100 persen.

---

## Soal 4: DHCP

Belum dikerjakan.

## Soal 5: NAT

Belum dikerjakan.

## Soal 6: Tunnel GRE

Belum dikerjakan.

## Soal 7: Dynamic routing OSPF

Belum dikerjakan.

## Soal 8: Layanan aplikasi Smart City

Belum dikerjakan.
