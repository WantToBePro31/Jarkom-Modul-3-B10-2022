# Jarkom-Modul-3-B10-2022

## Kelompok B10
| Name                      | NRP        | 
| ------------------------- | ---------- |
| Frederick Wijayadi Susilo | 5025201111 |
| Doanda Dresta Rahma       | 5025201049 |
| Zunia Aswaroh             | 5025201058 |

IP Prefix Kelompok: `10.8`

## Soal dan Jawaban

### Pendahuluan
![image](https://user-images.githubusercontent.com/67154280/200232115-f0139aa4-4858-4f07-af7f-49708a3675e2.png)

### 1
> Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server

### Penyelesaian
Kita dapat melakukan penginstallan masing-masing kebutuhan node yang dilakukan langsung di file `/root/.bashrc` dengan format sebagai berikut:

```shell
# WISE
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install bind9 -y

# Westalis
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y

# Berlint
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install squid -y
```

Selain, itu untuk menjadikan Westalis sebagai DHCP Server, kita perlu menambahkan konfigurasi pada node Westalis pada file `/etc/default/isc-dhcp-server` yang kita buat konfigurasinya pada file temporary yaitu `isc-dchp-server-1` dengan ditambahkan `INTERFACES="eth0"`

```shell
# Defaults for isc-dhcp-server initscript
# sourced by /etc/init.d/isc-dhcp-server
# installed at /etc/default/isc-dhcp-server by the maintainer scripts

#
# This is a POSIX shell fragment
#

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPD_CONF=/etc/dhcp/dhcpd.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPD_PID=/var/run/dhcpd.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES="eth0"
```

Setelah itu, kita jalankan script `soal1.sh` pada node Westalis yang berisi:

```shell
cp /root/isc-dhcp-server-1 /etc/default/isc-dhcp-server
service isc-dhcp-server start
```

### 2
> dan Ostania sebagai DHCP Relay

### Penyelesaian
Kita dapat melakukan penginstallan di node Ostania yang dilakukan langsung di file `/root/.bashrc` dengan format sebagai berikut:

```shell
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.8.0.0/16
cat /etc/resolv.conf
apt-get update
echo "" | apt-get install isc-dhcp-relay -y
```

Setelah itu, lakukan konfigurasi pada node Ostania pada file `/etc/default/isc-dhcp-relay` yang kita buat konfigurasinya pada file temporary yaitu `isc-dchp-relay-2` yang berisi:

```shell
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="10.8.2.4"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```

Lalu, kita jalankan script `soal2.sh` pada node Westalis yang berisi:

```shell
cp /root/isc-dhcp-relay-2 /etc/default/isc-dhcp-relay
service isc-dhcp-relay start
```

> Loid dan Franky menyusun peta tersebut dengan hati-hati dan teliti. Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu:

### 3-6
> Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server

### Penyelesaian


### 3
> Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155

### Penyelesaian


### 4
> Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

### Penyelesaian


### 5
> Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut

### Penyelesaian


### 6
> Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit

### Penyelesaian


### 7
> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13

### Penyelesaian
