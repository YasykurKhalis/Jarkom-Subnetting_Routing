# Tugas Jarkom - Subnetting & Routing (VLSM, CIDR)

## Deskripsi Tugas

Tugas ini mencakup perancangan dan implementasi jaringan untuk Yayasan Pendidikan ARA yang terdiri dari Kantor Pusat dan Kantor Cabang. Implementasi ini mencakup:
1.  Perhitungan subnetting menggunakan **VLSM** berdasarkan kebutuhan host.
2.  Perhitungan agregasi rute menggunakan **CIDR (Supernetting)**.
3.  Desain dan konfigurasi topologi jaringan pada **Cisco Packet Tracer**.
4.  Konfigurasi **Static Routing** untuk menghubungkan kedua kantor.

### Kebutuhan Host (Soal)

| Ruang | Jumlah Host Aktif |
| :--- | :--- |
| Sekretariat | 380 host |
| Bidang Kurikulum | 220 host |
| Bidang Guru & Tendik | 95 host |
| Bidang Sarana Prasarana | 45 host |
| Bidang Pengawas Sekolah (Branch) | 18 host |
| Server & Admin | 6 host |
| **Link WAN (Antar-Router)** | **2 host** |

---

## Implementasi & Perhitungan

### 1. Perhitungan Base Network

Berdasarkan aturan `10.(NRP mod 256).0.0`:
* NRP: `5027241112`
* Perhitungan: `5027241112 mod 256 = 152`
* **Base Network:** `10.152.0.0`

### 2. Tabel VLSM (Subnetting)

Berikut adalah hasil alokasi subnet VLSM, diurutkan dari kebutuhan host terbesar ke terkecil.

| Ruang | Kebutuhan Host | Network | Mask | Prefix | Range Host | Broadcast | Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Sekretariat | 380 | `10.152.0.0` | `255.255.254.0` | /23 | `10.152.0.1` - `10.152.1.254` | `10.152.1.255` | `10.152.0.1` |
| Kurikulum | 220 | `10.152.2.0` | `255.255.255.0` | /24 | `10.152.2.1` - `10.152.2.254` | `10.152.2.255` | `10.152.2.1` |
| Guru & Tendik | 95 | `10.152.3.0` | `255.255.255.128` | /25 | `10.152.3.1` - `10.152.3.126` | `10.152.3.127` | `10.152.3.1` |
| Sarana Prasarana | 45 | `10.152.3.128`| `255.255.255.192` | /26 | `10.152.3.129` - `10.152.3.190`| `10.152.3.191` | `10.152.3.129`|
| Pengawas (Branch)| 18 | `10.152.3.192`| `255.255.255.224` | /27 | `10.152.3.193` - `10.152.3.222`| `10.152.3.223` | `10.152.3.193`|
| Server & Admin | 6 | `10.152.3.224`| `255.255.255.248` | /29 | `10.152.3.225` - `10.152.3.230`| `10.152.3.231` | `10.152.3.225`|
| Link WAN | 2 | `10.152.3.232`| `255.255.255.252` | /30 | `10.152.3.233` - `10.152.3.234`| `10.152.3.235` | `...233` & `...234`|

### 3. Tabel CIDR (Supernetting / Agregasi)

Rute agregasi (supernet) yang merangkum semua 7 subnet di atas adalah sebagai berikut. Rute ini digunakan pada `R_Cabang` untuk meringkas semua jaringan di `R_Pusat`.

| Deskripsi | Network | Mask | Prefix | Range Host | Broadcast | Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Agregasi Penuh | `10.152.0.0` | `255.255.252.0` | /22 | `10.152.0.1` - `10.152.3.254` | `10.152.3.255` | N/A (Route) |

---

## Desain Topologi (Packet Tracer)

Desain topologi dibagi menjadi dua lokasi yang terhubung melalui link Serial WAN.

* **Kantor Pusat (R_Pusat):** Menggunakan topologi **Router-on-a-Stick**. Karena `R_Pusat` perlu melayani 5 LAN namun router `4331` memiliki port Layer 3 (routed port) yang terbatas, digunakanlah Sub-interface dan VLAN.
    * `VLAN 10`: Sekretariat
    * `VLAN 20`: Kurikulum
    * `VLAN 30`: Guru & Tendik
    * `VLAN 40`: Sarana Prasarana
    * `VLAN 50`: Server & Admin

* **Kantor Cabang (R_Cabang):** Menggunakan desain LAN standar, di mana port `GigabitEthernet0/0/0` router langsung terhubung ke switch `SW_Pengawas`.

* **Routing:** Konektivitas antara Kantor Pusat dan Kantor Cabang diatur menggunakan **Static Routing**.
