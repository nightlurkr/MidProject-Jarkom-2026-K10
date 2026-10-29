# Mid-Project Jarkom A - Kelompok 10

Catatan konfigurasi. Base network hasil UTS: **10.80.0.0/16**. Skema dipakai: **VLSM**.

File Packet Tracer: `K10_VLSM.pkt`

---

## 1. Perangkat

| Jenis | Jumlah | Model |
|---|---|---|
| Router | 8 | Cisco 2911 |
| Switch | 14 | Cisco 2960 |
| PC | 12 | PC-PT |
| Server | 3 | Server-PT |

Backbone antar-router memakai **serial**, bukan Ethernet. Alasannya: 2911 hanya punya 3 port GigabitEthernet, sedangkan R-PUSAT butuh 8 sambungan. Tiga port Gigabit dipakai untuk LAN, dan sambungan antar-router memakai modul **HWIC-2T** (2 port serial per modul).

Jumlah modul: R-PUSAT 3, R-UTARA 2, router lain 1 masing-masing.

---

## 2. Peta interface

### Backbone

| Link | Sisi A | IP A | Sisi B | IP B | Network |
|---|---|---|---|---|---|
| ISP | R-ISP Se0/3/0 | 203.0.113.42 | R-PUSAT Se0/1/0 | 203.0.113.41 | 203.0.113.40/30 |
| L1 | R-PUSAT Se0/1/1 | 10.80.241.33 | R-DC Se0/3/0 | 10.80.241.34 | 10.80.241.32/30 |
| L2 | R-PUSAT Se0/2/0 | 10.80.241.37 | R-UTARA Se0/3/0 | 10.80.241.38 | 10.80.241.36/30 |
| L3 | R-PUSAT Se0/2/1 | 10.80.241.41 | R-SELATAN Se0/3/0 | 10.80.241.42 | 10.80.241.40/30 |
| L4 | R-PUSAT Se0/3/0 | 10.80.241.45 | R-TIMUR Se0/3/0 | 10.80.241.46 | 10.80.241.44/30 |
| L5 | R-PUSAT Se0/3/1 | 10.80.241.49 | R-BARAT Se0/3/0 | 10.80.241.50 | 10.80.241.48/30 |
| L6 | R-UTARA Se0/2/0 | 10.80.241.53 | R-PELABUHAN Se0/3/0 | 10.80.241.54 | 10.80.241.52/30 |
| L7 | belum dipasang | | | | 10.80.241.56/30 |

L7 (R-DC ke R-UTARA) hanya dipasang di file OSPF, sesuai soal 7.

### LAN

| LAN | Router | Port | Gateway | Mask | Network |
|---|---|---|---|---|---|
| Pegawai Balai Kota | R-PUSAT | Gi0/0 | 10.80.224.1 | 255.255.248.0 | 10.80.224.0/21 |
| Server Balai Kota | R-PUSAT | Gi0/1 | 10.80.241.1 | 255.255.255.224 | 10.80.241.0/27 |
| CCTV Kota | R-DC | Gi0/0 | 10.80.96.1 | 255.255.224.0 | 10.80.96.0/19 |
| Server dan Storage DC | R-DC | Gi0/1 | 10.80.239.1 | 255.255.255.0 | 10.80.239.0/24 |
| WiFi Publik Barat | R-BARAT | Gi0/0 | 10.80.192.1 | 255.255.224.0 | 10.80.192.0/19 |
| Puskesmas dan Sekolah Barat | R-BARAT | Gi0/1 | 10.80.238.1 | 255.255.255.0 | 10.80.238.0/24 |
| WiFi Publik Utara | R-UTARA | Gi0/0 | 10.80.0.1 | 255.255.192.0 | 10.80.0.0/18 |
| Sensor Lalu Lintas Utara | R-UTARA | Gi0/1 | 10.80.236.1 | 255.255.254.0 | 10.80.236.0/23 |
| WiFi Publik Timur | R-TIMUR | Gi0/0 | 10.80.128.1 | 255.255.224.0 | 10.80.128.0/19 |
| Sensor Banjir Timur | R-TIMUR | Gi0/1 | 10.80.232.1 | 255.255.252.0 | 10.80.232.0/22 |
| WiFi Publik Selatan | R-SELATAN | Gi0/0 | 10.80.64.1 | 255.255.224.0 | 10.80.64.0/19 |
| PJU Pintar Selatan | R-SELATAN | Gi0/1 | 10.80.240.129 | 255.255.255.128 | 10.80.240.128/25 |
| Sensor dan SCADA Pelabuhan | R-PELABUHAN | Gi0/0 | 10.80.160.1 | 255.255.224.0 | 10.80.160.0/19 |
| Kantor Pelabuhan | R-PELABUHAN | Gi0/1 | 10.80.240.1 | 255.255.255.128 | 10.80.240.0/25 |
| Server internet | R-ISP | Gi0/0 | 8.8.8.1 | 255.255.255.0 | 8.8.8.0/24 |

---

## 3. Alamat statis yang sudah diisi

| Perangkat | IP | Mask | Gateway |
|---|---|---|---|
| SRV-BALAIKOTA | 10.80.241.2 | 255.255.255.224 | 10.80.241.1 |
| SRV-STORAGE-DC | 10.80.239.2 | 255.255.255.0 | 10.80.239.1 |
| SRV-INTERNET | 8.8.8.8 | 255.255.255.0 | 8.8.8.1 |

Seluruh PC sengaja dibiarkan kosong, menunggu konfigurasi DHCP (soal 4).

---

## 4. Static routing (soal 3)

Aturan yang dipakai: default route hanya mengarah ke router di atasnya, dan hanya R-PUSAT serta R-UTARA yang punya static route ke jaringan di bawahnya.

### Router daun

```
R-DC          ip route 0.0.0.0 0.0.0.0 10.80.241.33
R-BARAT       ip route 0.0.0.0 0.0.0.0 10.80.241.49
R-TIMUR       ip route 0.0.0.0 0.0.0.0 10.80.241.45
R-SELATAN     ip route 0.0.0.0 0.0.0.0 10.80.241.41
R-PELABUHAN   ip route 0.0.0.0 0.0.0.0 10.80.241.53
```

R-PELABUHAN mengarah ke R-UTARA karena posisinya di bawah R-UTARA, bukan langsung di bawah R-PUSAT.

### R-UTARA

```
ip route 0.0.0.0 0.0.0.0 10.80.241.37
ip route 10.80.160.0 255.255.224.0 10.80.241.54
ip route 10.80.240.0 255.255.255.128 10.80.241.54
```

### R-PUSAT

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

Dua LAN milik R-PELABUHAN diarahkan ke R-UTARA, bukan langsung, karena R-PELABUHAN hanya tersambung lewat R-UTARA.

### R-ISP

Tidak diberi rute apa pun ke alamat privat, sesuai syarat soal 5.

---

## 5. Bukti pengujian

| Dari | Tujuan | Hasil |
|---|---|---|
| R-ISP | 203.0.113.41 | 100% |
| R-PUSAT | 10.80.241.34, .38, .42, .46, .50 | 100% semua |
| R-UTARA | 10.80.241.54 | 100% |
| R-PUSAT | 10.80.241.2 (SRV-BALAIKOTA) | 100% |
| R-BARAT | 10.80.239.1, 10.80.240.1, 10.80.64.1 | 100% semua |
| R-PELABUHAN | 10.80.192.1, 10.80.241.1, 10.80.128.1 | 100% semua |

---

## 6. Catatan untuk anggota berikutnya

**Untuk B (soal 4, 5, 8):**
- Semua PC masih kosong, siap diisi DHCP
- Subnet CCTV Kota (10.80.96.0/19) harus dilayani server DHCP di LAN Server DC, bukan dari router. Artinya perlu `ip helper-address 10.80.239.2` di interface Gi0/0 R-DC
- NAT dipasang di R-PUSAT, interface luar Se0/1/0, interface dalam semua Gi dan Se lainnya
- Domain internal: `k10.smartcity.lab`

**Untuk C (soal 6, 7):**
- Tunnel GRE: 192.168.10.0/30, R-PUSAT .1 dan R-PELABUHAN .2
- L7 (R-DC ke R-UTARA) belum dipasang, alamat yang disiapkan 10.80.241.56/30, R-DC .57 dan R-UTARA .58
- File OSPF dibuat dari copy file utama, jangan edit file utama

---

## 7. Hal yang perlu dipahami sebelum demo

Pertanyaan yang mungkin muncul saat Uji 2, beserta jawabannya:

- **Kenapa backbone pakai serial?** Port Gigabit di 2911 cuma 3, sedangkan R-PUSAT butuh 8 sambungan
- **Kenapa link antar-router /30?** Hanya butuh 2 alamat host, satu per router. /30 memberi tepat 2
- **Apa itu clock rate?** Sisi DCE pada link serial yang menentukan kecepatan sinkronisasi. Sisi DTE mengikuti
- **Kenapa R-PUSAT butuh banyak static route?** Karena router di bawahnya hanya punya default route ke atas. R-PUSAT harus tahu jalan turun ke tiap LAN
- **Kenapa LAN Pelabuhan diarahkan ke R-UTARA di R-PUSAT?** Karena R-PELABUHAN tidak tersambung langsung ke R-PUSAT, jalurnya lewat R-UTARA
- **Kenapa R-ISP tidak punya rute ke 10.80.x?** Supaya alamat privat tidak bisa dijangkau dari luar. Akses keluar ditangani NAT di R-PUSAT
