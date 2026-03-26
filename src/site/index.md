---
title: Home
---

# Kolaborasi OSPF dan RIP
Kolaborasi ini menggunakan topologi yang menggabungkan antara routing RIP dan OSPF
komponen terdiri dari 3 router dan 2 pc. Pada bagian kiri digunakan untuk routing RIP dan bagian kanan digunakan routing OSPF

langkah-lamgkah : 
1. Membuat topologi dulu sesuai dengan yang ada digambar
2. Mengecek ping antar tetangga dan pastikan ping tersebut berhasil karena biasanya terjadi masalah dasar yang akan memengaruhi konfigurasi lanjutan, jadi disarankan untuk cek terlebih dahulu
3. Untuk setting ip pc dilakukan setiap kali project dijalankan, karena config ip di gns tidak dapat disimpan
```
	PC 1 -->
	PC1> ip 192.168.10.2/24 192.168.10.1
	Checking for duplicate address...
	PC1 : 192.168.10.2 255.255.255.0 gateway 192.168.10.1
	
	PC 2-->
	PC2> ip 192.168.20.2/24 192.168.20.1
	Checking for duplicate address...
	PC2 : 192.168.20.2 255.255.255.0 gateway 192.168.20.1
```

4. Sebelum masuk tahap konfigurasi disarankan untuk mereset mikrotik untuk menghilangkan sisa konfigurtasi dari praktik kemarin, atau bisa mendisable konfigurasi yang tidak diperlukan

5. Konfigurasi Router 1 (kiri)   pada bagian RIP tambahkan new instance dengan retributued connected, lakukan untuk inisialisi kedua ether
```
parameter sukses Router 1 (Kiri)

[admin@MikroTik 1] > ip route print 
Flags: D - DYNAMIC; A - ACTIVE; c - CONNECT, r - RIP
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
    DST-ADDRESS      GATEWAY            ROUTING-TABLE  DISTANCE
DAc 10.10.10.0/24    ether2             main                  0
DAr 10.10.20.0/24    10.10.10.2%ether2  main                120
DAc 192.168.10.0/24  ether1             main                  0
DAr 192.168.20.0/24  10.10.10.2%ether2  main                120
```

6. Konfigurasi Router 2 (kanan) pada bagian ospf klik new instance dengan retrisibuted connected dengan Router ID 2.2.2.2, setelah itu tammbahkan areas dengan nama backbone dan Area ID biarkan tetap 0.0.0.0, lalu tambahkan new interfaces template dan tambahkan untuk inisialisasi kedua ether
```
parameter sukses Router 2 (Kanan)

[admin@MikroTik 2] > ip route print
Flags: D - DYNAMIC; A - ACTIVE; c - CONNECT, o - OSPF
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
    DST-ADDRESS      GATEWAY            ROUTING-TABLE  DISTANCE
DAo 10.10.10.0/24    10.10.20.2%ether2  main                110
DAc 10.10.20.0/24    ether2             main                  0
DAo 192.168.10.0/24  10.10.20.2%ether2  main                110
DAc 192.168.20.0/24  ether1             main                  0
```

7. Konfigurasi Router 3 (tengah) pada bagian ini konfigurasi router penghubung antara routing rip dan ospf sangat penting dan perlu diketahui bahwa keberhasilan sangat ditentukan di bagian ini, langkah pertama silahkan untuk konfigurasi RIP terlebih dahulu tambahkan new instace dengan redistributrd connected dan ospf, setelah itu tambahkan interfaces template dan ethernya adalah ether 1 atau ether yang menghubungkan ke Router 1, setelah itu konfigurasi pada bagian OSPF tambahkan instance dengan redistributef connected dan RIP, setelah itu tambahkan bagian area dengan nama backbone dan Area ID 0.0.0.0 , dan terakhir tambahkan interfaces templatenya dengan ether ke Router 3 atau ether 2
```
parameter sukses Router 3 (Tengah)

[admin@MikroTik 3] > ip route print
Flags: D - DYNAMIC; A - ACTIVE; c - CONNECT, r - RIP, o - OSPF
Columns: DST-ADDRESS, GATEWAY, ROUTING-TABLE, DISTANCE
    DST-ADDRESS      GATEWAY                  ROUTING-TABLE  DISTANCE
DAc 10.10.10.0/24    ether1_to_R1             main                  0
DAc 10.10.20.0/24    ether2_to_R2             main                  0
DAr 192.168.10.0/24  10.10.10.1%ether1_to_R1  main                120
DAo 192.168.20.0/24  10.10.20.1%ether2_to_R2  main                110

```

8. Pengujiaan Ping antar PC 1 dan PC 2
```
PC1> ping 192.168.20.2

84 bytes from 192.168.20.2 icmp_seq=1 ttl=61 time=1.368 ms
84 bytes from 192.168.20.2 icmp_seq=2 ttl=61 time=1.218 ms
84 bytes from 192.168.20.2 icmp_seq=3 ttl=61 time=1.655 ms
84 bytes from 192.168.20.2 icmp_seq=4 ttl=61 time=1.604 ms
84 bytes from 192.168.20.2 icmp_seq=5 ttl=61 time=1.144 ms
```

9. Pengujiaan Ping antar PC 1 dan PC 2
```
PC2> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=61 time=1.713 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=61 time=1.904 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=61 time=1.525 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=61 time=1.335 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=61 time=1.710 ms
```

**Refrensi Claude AI**
## 🔴 MikroTik1-1 (R1) — RIP Only

### Step 1 — Set IP Address

bash

```bash
/ip address add address=192.168.10.1/24 interface=ether1
/ip address add address=10.10.10.1/24 interface=ether2
```

### Step 2 — Buat RIP Instance

bash

```bash
/routing rip instance add name=rip-instance redistribute=connected
```

### Step 3 — Buat RIP Interface Template

bash

```bash
/routing rip interface-template add instance=rip-instance interfaces=ether1
/routing rip interface-template add instance=rip-instance interfaces=ether2
```

### Step 4 — Verifikasi R1

bash

```bash
/ip address print
/routing rip instance print detail
/routing rip interface-template print detail
```

---

## 🟡 MikroTik2-1 (R2) — OSPF Only

### Step 1 — Set IP Address

bash

```bash
/ip address add address=10.10.20.1/24 interface=ether2
/ip address add address=192.168.20.1/24 interface=ether1
```

### Step 2 — Buat OSPF Instance

bash

```bash
/routing ospf instance add name=ospf-instance router-id=2.2.2.2 redistribute=connected
```

### Step 3 — Buat OSPF Area

bash

```bash
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf-instance
```

### Step 4 — Buat OSPF Interface Template

bash

```bash
/routing ospf interface-template add interfaces=ether2 area=backbone
/routing ospf interface-template add interfaces=ether1 area=backbone
```

### Step 5 — Verifikasi R2

bash

```bash
/ip address print
/routing ospf instance print detail
/routing ospf area print detail
/routing ospf interface-template print detail
```

---

## 🔁 MikroTik3-1 (R3) — RIP + OSPF + Redistribution

### Step 1 — Set IP Address

bash

```bash
/ip address add address=10.10.10.2/24 interface=ether1
/ip address add address=10.10.20.2/24 interface=ether2
```

### Step 2 — Buat RIP Instance

bash

```bash
/routing rip instance add name=rip-instance redistribute=connected,ospf
```

### Step 3 — Buat RIP Interface Template

bash

```bash
/routing rip interface-template add instance=rip-instance interfaces=ether1
```

### Step 4 — Buat OSPF Instance

bash

```bash
/routing ospf instance add name=ospf-instance router-id=3.3.3.3 redistribute=connected,rip
```

### Step 5 — Buat OSPF Area

bash

```bash
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf-instance
```

### Step 6 — Buat OSPF Interface Template

bash

```bash
/routing ospf interface-template add interfaces=ether2 area=backbone
```

### Step 7 — Verifikasi R3

bash

````bash
/ip address print
/routing rip instance print detail
/routing rip interface-template print detail
/routing ospf instance print detail
/routing ospf area print detail
/routing ospf interface-template print detail
```

---

## 💻 Konfigurasi VPCS

**PC1:**
```
ip 192.168.10.2/24 192.168.10.1
```

**PC2:**
```
ip 192.168.20.2/24 192.168.20.1
````

---

## ✅ Verifikasi Akhir Semua Router

Jalankan ini di **masing-masing router** setelah semua dikonfigurasi, tunggu **30-60 detik** dulu biar converge:

bash

````bash
/ip route print
/routing rip neighbor print
/routing ospf neighbor print
```

### Hasil yang diharapkan:

**R1 route table:**
```
DAc 192.168.10.0/24  ether1
DAc 10.10.10.0/24    ether2
DAr 10.10.20.0/24    10.10.10.2   ← dari RIP redistribusi OSPF
DAr 192.168.20.0/24  10.10.10.2   ← dari RIP redistribusi OSPF
```

**R2 route table:**
```
DAc 10.10.20.0/24    ether2
DAc 192.168.20.0/24  ether1
DAo 10.10.10.0/24    10.10.20.2   ← dari OSPF redistribusi RIP
DAo 192.168.10.0/24  10.10.20.2   ← dari OSPF redistribusi RIP
```

**R3 route table:**
```
DAc 10.10.10.0/24    ether1
DAc 10.10.20.0/24    ether2
DAr 192.168.10.0/24  10.10.10.1   ← dari RIP R1
DAo 192.168.20.0/24  10.10.20.1   ← dari OSPF R2
````
