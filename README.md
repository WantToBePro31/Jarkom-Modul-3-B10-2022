# Jarkom-Modul-3-B10-2022

## Kelompok B10
| Name                      | NRP        | 
| ------------------------- | ---------- |
| Frederick Wijayadi Susilo | 5025201111 |
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

Selain, itu untuk menjadikan Westalis sebagai DHCP Server, kita perlu menambahkan konfigurasi pada node Westalis pada file `/etc/default/isc-dhcp-server` yang kita buat konfigurasinya pada file temporary yaitu `isc-dhcp-server-1` dengan ditambahkan `INTERFACES="eth0"`

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
service isc-dhcp-server restart
```

Hal ini membuat masing-masing node memiliki server tersendiri.

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

Setelah itu, lakukan konfigurasi pada node Ostania pada file `/etc/default/isc-dhcp-relay` yang kita buat konfigurasinya pada file temporary yaitu `isc-dhcp-relay-2` yang berisi:

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

Lalu, kita jalankan script `soal2.sh` pada node Ostania yang berisi:

```shell
cp /root/isc-dhcp-relay-2 /etc/default/isc-dhcp-relay
service isc-dhcp-relay restart
```

Hal ini membuat node Ostania menjadi DHCP relay yang Servernya mengarah ke Westalis serta Interfacenya mengarah ke eth1, eth2, dan eth3 yang merupakan switch 1, 2, dan 3.

> Loid dan Franky menyusun peta tersebut dengan hati-hati dan teliti. Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu:

### 3-6
> Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server

### Penyelesaian
Untuk membuat semua client menggunakan konfigurasi IP dari DHCP Server, maka kita harus edit network configuration untuk masing-masing node menjadi:

```shell
auto eth0
iface eth0 inet dhcp
```

Setelah itu, kita lakukan konfigurasi pada file `/etc/dhcp/dhcpd.conf` pada node Westalis untuk mengatur parameter jaringan yang dapat didistribusikan oleh DHCP. Konfigurasi ini akan dihasilkan dengan file penuh `dhcpd3-6.conf` dengan script sebagai berikut

```shell
#
# Sample configuration file for ISC dhcpd for Debian
#
# Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
# configuration file instead of this file.
#
#

subnet 10.8.1.0 netmask 255.255.255.0 {
    range 10.8.1.50 10.8.1.88;
    range 10.8.1.120 10.8.1.155;
    option routers 10.8.1.1;
    option broadcast-address 10.8.1.255;
    option domain-name-servers 10.8.2.2; 
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 10.8.3.0 netmask 255.255.255.0 {
    range 10.8.3.10 10.8.3.30;
    range 10.8.3.60 10.8.3.85;
    option routers 10.8.3.1;
    option broadcast-address 10.8.3.255;
    option domain-name-servers 10.8.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}

subnet 10.8.2.0 netmask 255.255.255.0 {
    option routers 10.8.2.1;
}

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.

#subnet 10.152.187.0 netmask 255.255.255.0 {
#}

# This is a very basic subnet declaration.

#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}

# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.fugue.com";
#}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.fugue.com;
#}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}

#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
#}
```

Lalu, akan menjalankan script `soal3-6.sh` dengan isi sebagai berikut

```shell
cp /root/dhcpd3-6.conf /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart
```

### 3
> Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155

### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet `10.8.1.0` yang dilalui Switch1 dengan melakukan penetapan range IP sebagai berikut

```shell
subnet 10.8.1.0 netmask 255.255.255.0 {
    range 10.8.1.50 10.8.1.88;
    range 10.8.1.120 10.8.1.155;
    ...
}

subnet 10.8.3.0 netmask 255.255.255.0 {
    ...
}

subnet 10.8.2.0 netmask 255.255.255.0 {
    ...
}
```

Hal ini akan membuat Client yang melalui Switch1 memiliki range IP yang sudah ditetapkan.

### 4
> Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet `10.8.3.0` yang dilalui Switch3 dengan melakukan penetapan range IP sebagai berikut

```shell
subnet 10.8.1.0 netmask 255.255.255.0 {
    ...
}

subnet 10.8.3.0 netmask 255.255.255.0 {
    range 10.8.3.10 10.8.3.30;
    range 10.8.3.60 10.8.3.85;
    ...
}

subnet 10.8.2.0 netmask 255.255.255.0 {
    ...
}
```

Hal ini akan membuat Client yang melalui Switch3 memiliki range IP yang sudah ditetapkan.

### 5
> Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut

### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet yang dilalui Switch1 dan Switch3 dengan melakukan penetapan IP DNS Server yang terhubung sebagai berikut

```shell
subnet 10.8.1.0 netmask 255.255.255.0 {
    ...
    option domain-name-servers 10.8.2.2; 
    ...
}

subnet 10.8.3.0 netmask 255.255.255.0 {
    ...
    option domain-name-servers 10.8.2.2; 
    ...
}

subnet 10.8.2.0 netmask 255.255.255.0 {
    ...
}
```

Selain itu, kita juga akan membuat suatu temporary file untuk mengedit konfigurasi file `/etc/bind/named.conf.options` yaitu pada file `named-5.conf.options` dengan isi sebagai berikut

```shell
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
            192.168.122.1;
        };

        //=====================================================================$
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //=====================================================================$
        //dnssec-validation auto;
        allow-query{any;};
        
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
```

Lalu, akan menjalankan script `soal5.sh` dengan isi sebagai berikut

```shell
cp /root/named-5.conf.options /etc/bind/named.conf.options
service bind9 restart
```

Hal ini akan membuat Client mendapatkan DNS dari WISE dan terhubung dengan internet melalui DNS.

### 6
> Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit

### Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet yang dilalui Switch1 dan Switch3 dengan melakukan penetapan lama waktu dan waktu maksimal untuk peminjaman alamat IP kepada Client oleh DHCP Server sebagai berikut

```shell
subnet 10.8.1.0 netmask 255.255.255.0 {
    ...
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 10.8.3.0 netmask 255.255.255.0 {
    ...
    default-lease-time 600;
    max-lease-time 6900;
}

subnet 10.8.2.0 netmask 255.255.255.0 {
    ...
}
```

Hal ini akan membuat lama waktu untuk peminjaman alamat IP kepada Client oleh DHCP Server melalui Switch1 selama 5menit (300detik) dan melalui Switch3 selama 10menit (600detik), serta waktu maksimal peminjamannya selama 115menit (6900detik).


### 7
> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13

### Penyelesaian
Pertama, kita akan mendapatkan hwaddress pada node Eden dengan menggunakan command `ip a` dan akan didapatkan nilai `4a:99:b7:2c:f9:c2` pada `link/ether`

Setelah itu, dapat dibuat file temporary `dhcpd7.conf` dengan isi sebagai berikut

```shell
#
# Sample configuration file for ISC dhcpd for Debian
#
# Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
# configuration file instead of this file.
#
#

subnet 10.8.1.0 netmask 255.255.255.0 {
    range 10.8.1.50 10.8.1.88;
    range 10.8.1.120 10.8.1.155;
    option routers 10.8.1.1;
    option broadcast-address 10.8.1.255;
    option domain-name-servers 10.8.2.2; 
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 10.8.3.0 netmask 255.255.255.0 {
    range 10.8.3.10 10.8.3.30;
    range 10.8.3.60 10.8.3.85;
    option routers 10.8.3.1;
    option broadcast-address 10.8.3.255;
    option domain-name-servers 10.8.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}

subnet 10.8.2.0 netmask 255.255.255.0 {
    option routers 10.8.2.1;
}

host Eden {
    hardware ethernet 4a:99:b7:2c:f9:c2;
    fixed-address 10.8.3.13;
}

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.

#subnet 10.152.187.0 netmask 255.255.255.0 {
#}

# This is a very basic subnet declaration.

#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}

# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.fugue.com";
#}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.fugue.com;
#}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}

#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
```

Lalu, edit network configuration pada node Eden dengan menambahkan hwaddress sebagai berikut

```shell
auto eth0
iface eth0 inet dhcp
hwaddress ether 4a:99:b7:2c:f9:c2
```

Setelah itu, jalankan script `soal7.sh` dengan isi sebagai berikut

```shell
cp /root/dhcpd7.conf /etc/dhcp/dhcpd.conf
service isc-dhcp-server restart
```

Hal itu menyebabkan IP fixed address pada node Eden yang memiliki interface eth0 berubah menjadi `10.8.3.13/24`

![image](https://user-images.githubusercontent.com/67154280/200377235-889a77bd-8053-4d79-afa1-b0db828351e5.png)

> SSS digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data. Pada Proxy Server di SSS, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan SSS sebagai HTTP & HTTPS proxy. 

### Note
    Untuk testing, pada file `.bashrc` setiap node akan dilakukan penginstallan lynx dan speedtest serta mengaktifkan http proxy dengan command
    ```shell
       apt-get update
       apt-get install lynx -y
       apt install speedtest-cli -y
       export PYTHONHTTPSVERIFY=0
       export http_proxy=http://10.8.2.3:8080
    ```
    
> Adapun kriteria pengaturannya adalah sebagai berikut:

### 8
> Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

### Penyelesaian

- acl-1.conf
  
  ```shell
    acl CAN_ACCESS_1 time MTWHF 00:00-07:59
    acl CAN_ACCESS_2 time MTWHF 17:01-23:59
    acl CAN_ACCESS_3 time AS 00:00-23:59

    acl WORK_HOUR time MTWHF 08:00-17:00
  ```

- squid8-9.conf
  
  ```shell
    include /etc/squid/acl.conf

    http_port 8080
    visible_hostname Berlint

    http_access deny CERTAIN_DOMAIN CAN_ACCESS_1
    http_access deny CERTAIN_DOMAIN CAN_ACCESS_2
    http_access deny CERTAIN_DOMAIN CAN_ACCESS_3

    http_access allow CAN_ACCESS_1
    http_access allow CAN_ACCESS_2
    http_access allow CAN_ACCESS_3
  ```

Ketika di test pada client dengan `lynx http://its.ac.id` dan `lynx https://its.ac.id` akan menghasilkan seperti ini:

- Senin (10.00)

![image](https://user-images.githubusercontent.com/67154280/201596650-3accc237-a0a4-4dbc-bb78-0702e9518810.png)

- Senin (20.00) dan Sabtu (10.00)

![image](https://user-images.githubusercontent.com/67154280/201596969-0da0f06c-da91-4995-9eb7-86b2d9f2f5f5.png)

### 9 
> Adapun pada hari dan jam kerja sesuai nomor (8), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)

### Penyelesaian

**Node Wise**

    ```shell
        # /etc/bind/named.conf.local
        zone "loid-work.com" {
            type master;
            file "/etc/bind/wise/loid-work.com";
            allow-transfer { 10.8.3.13; };
        };

        zone "franky-work.com" {
            type master;
            file "/etc/bind/wise/franky-work.com";
            allow-transfer { 10.8.3.13; };
        };

        # /etc/bind/wise/loid-work.com
        ;
        ; BIND data file for local loopback interface
        ;
        $TTL    604800
        @       IN      SOA     loid-work.com. root.loid-work.com. (
                                2022100601      ; Serial
                                 604800         ; Refresh
                                  86400         ; Retry
                                2419200         ; Expire
                                 604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      loid-work.com.
        @       IN      A       10.8.3.13
        www     IN      CNAME   loid-work.com.

        # /etc/bind/wise/franky-work.com
        ;
        ; BIND data file for local loopback interface
        ;
        $TTL    604800
        @       IN      SOA     franky-work.com. root.franky-work.com. (
                                2022100601      ; Serial
                                 604800         ; Refresh
                                  86400         ; Retry
                                2419200         ; Expire
                                 604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      franky-work.com.
        @       IN      A       10.8.3.13
        www     IN      CNAME   franky-work.com.
    ```

**Node Eden**

```shell
   apt-get install apache2 -y
   apt-get install php -y
   apt-get install libapache2-mod-php7.0 -y

   # /etc/apache2/apache2.conf
   ...
   ServerName 10.8.3.13
   
   # /etc/apache2/sites-available/loid-work.com.conf
   <VirtualHost *:80>
        ServerName loid-work.com
        ServerAlias www.loid-work.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/loid-work.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

   # /etc/apache2/sites-available/franky-work.com.conf
   <VirtualHost *:80>
        ServerName franky-work.com
        ServerAlias www.franky-work.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky-work.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   
   a2ensite loid-work.com
   a2ensite franky-work.com

   # /var/www/loid-work.com/index.php
   <?php
        echo "Ini domain loid-work";
   ?>
   
   # /var/www/franky-work.com/index.php
   <?php
        echo "Ini domain franky-work";
   ?>
```

- acl-1.conf
  
  ```shell
    acl CAN_ACCESS_1 time MTWHF 00:00-07:59
    acl CAN_ACCESS_2 time MTWHF 17:01-23:59
    acl CAN_ACCESS_3 time AS 00:00-23:59

    acl WORK_HOUR time MTWHF 08:00-17:00
    acl CERTAIN_DOMAIN dstdomain .loid-work.com .franky-work.com
  ```

- squid8-9.conf

  ```shell
    include /etc/squid/acl.conf

    http_port 8080
    visible_hostname Berlint

    http_access deny CERTAIN_DOMAIN CAN_ACCESS_1
    http_access deny CERTAIN_DOMAIN CAN_ACCESS_2
    http_access deny CERTAIN_DOMAIN CAN_ACCESS_3
    
    http_access allow CAN_ACCESS_1
    http_access allow CAN_ACCESS_2
    http_access allow CAN_ACCESS_3
    
    http_access allow CERTAIN_DOMAIN WORK_HOUR
  ```

Ketika di test pada client dengan `lynx loid-work.com` dan `lynx franky-work.com` akan menghasilkan seperti ini:

- Senin (10.00)

![image](https://user-images.githubusercontent.com/67154280/201597262-bd21739a-d446-4372-897e-403cf624cbda.png)

- Senin (20.00) dan Sabtu (10.00)

![image](https://user-images.githubusercontent.com/67154280/201597322-528a9d46-26e5-46eb-9109-2a1ab10f9759.png)

![image](https://user-images.githubusercontent.com/67154280/201598535-b44e8e03-0eb9-4024-80b9-d0c9e83f35ae.png)

### 10 
> Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

### Penyelesaian 

- squid10.conf
  
  ```shell
    include /etc/squid/acl.conf

    http_port 8080
    visible_hostname Berlint

    http_access deny CERTAIN_DOMAIN CAN_ACCESS_1
    http_access deny CERTAIN_DOMAIN CAN_ACCESS_2
    http_access deny CERTAIN_DOMAIN CAN_ACCESS_3
    http_access deny all
    
    http_access allow CAN_ACCESS_1
    http_access allow CAN_ACCESS_2
    http_access allow CAN_ACCESS_3
    http_access deny all
    
    http_access allow CERTAIN_DOMAIN WORK_HOUR
  ```

Ketika di test pada client dengan  `lynx http://example.com` akan menghasilkan seperti ini:

- Senin (10.00)

![image](https://user-images.githubusercontent.com/67154280/201597707-f5dcc217-c56e-41b5-b481-054bf0997938.png)

- Senin (20.00) dan Sabtu (10.00)

![image](https://user-images.githubusercontent.com/67154280/201598216-72f97766-da0e-4f81-a56c-c76afc7e0541.png)

![image](https://user-images.githubusercontent.com/67154280/201598286-5c105af9-7bf8-4003-b536-354c1dd62342.png)

### 11 & 12
> Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)
> Setelah diterapkan, ternyata peraturan nomor (11) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

### Penyelesaian

- acl-bandwidth-1.conf

  ```shell
    delay_pools 1
    delay_class 1 1
    delay_parameters 1 16000/16000
    delay_access 1 allow CAN_ACCESS_3
  ```

- squid11-12.conf

   ```shell
     include /etc/squid/acl.conf
     include /etc/squid/acl-bandwidth.conf
     
     http_port 8080
     visible_hostname Berlint

     http_access deny CERTAIN_DOMAIN CAN_ACCESS_1
     http_access deny CERTAIN_DOMAIN CAN_ACCESS_2
     http_access deny CERTAIN_DOMAIN CAN_ACCESS_3
    
     http_access allow CAN_ACCESS_1
     http_access allow CAN_ACCESS_2
     http_access allow CAN_ACCESS_3
    
     http_access allow CERTAIN_DOMAIN WORK_HOUR
     
     http_access deny all
   ```

Ketika di test pada client dengan `speedtest` akan menghasilkan seperti ini:

- Senin (10.00)

![image](https://user-images.githubusercontent.com/67154280/201599463-b0d3b050-1626-404d-94d5-fecfc2ad245a.png)

- Senin (20.00) dan Sabtu (10.00)

![image](https://user-images.githubusercontent.com/67154280/201599874-6cc6ed09-9bb0-4eec-b576-96454b7ec792.png)

![image](https://user-images.githubusercontent.com/67154280/201603046-bbcd485c-63de-4cf3-b944-e09a66bc0705.png)

