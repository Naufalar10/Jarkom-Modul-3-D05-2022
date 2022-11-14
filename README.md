# Lapres Jarkom Modul 3 Kelompok D05 #

| Nama Praktikan  | NRP Praktikan |
| ------------- | ------------- |
| Ichlasul Hasanat  | 5025201091  |
| Haidar Fico Ramadhan Aryputra | 5025201185  |
| Naufal Ariq Putra Yosyam | 5025201112 |


# SOAL DAN JAWABAN PRAKTIKUM

# SOAL 1
Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server

## Penyelesaian
Kita dapat melakukannya dengan membuat suatu script ``` install.sh ``` yang isinya sebagai berikut :
```
# WISE
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install bind9 -y

# westalis
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y

# berlint
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install squid -y

cp /etc/squid/squid.conf /etc/resolv.conf

service squid start
```

Selain itu, Kita juga membuat sebuat script yaitu ``` isc-dhcp-server ``` di westalis yang isinya :
```
INTERFACES="eth0" > /etc/default/isc-dhcp-server
```

Setelah itu, kita jalankan script ``` soal1-6.sh ``` pada node Westalis yang berisi:
```
cp isc-dhcp-server /etc/default/isc-dhcp-server
cp dhcpd.conf /etc/dhcp/dhcpd.conf

service isc-dhcp-server start
service isc-dhcp-server status
```

# SOAL 2
dan Ostania sebagai DHCP Relay

## Penyelesaian
Kita dapat melakukan permintaan soal dengan terlebih dahulu membuat script ``` isc-dhcp-relay ``` di Ostania dengan isi sebagai berikut :
```
SERVERS="192.187.2.4"

INTERFACES="eth1 eth2 eth3"

OPTIONS=""
root@Ostania:~# cat soal2.sh 
cp isc-dhcp-relay /etc/default/isc-dhcp-relay
```

lalu membuat script ``` install. sh  ``` dengan isi :
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.187.0.0/16

apt-get update
echo "" | apt-get install isc-dhcp-relay -y
```

lalu membuat script ``` soal2.sh ``` lalu jalankan dengan isi :
```
cp isc-dhcp-relay /etc/default/isc-dhcp-relay

service isc-dhcp-relay start
```

# SOAL 3-6
Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server

## Penyelesaian
Untuk membuat semua client menggunakan konfigurasi IP dari DHCP Server, maka kita harus edit network configuration untuk masing-masing node menjadi:
```
auto eth0
iface eth0 inet dhcp
```

Setelah itu, kita lakukan konfigurasi pada file ``` /etc/dhcp/dhcpd.conf ``` pada node Westalis untuk mengatur parameter jaringan yang dapat didistribusikan oleh DHCP. Kita membuat script ``` dhcpd.conf ``` yang isinya :
```
subnet 192.187.1.0  netmask 255.255.255.0 {
    range 192.187.1.50 192.187.1.88;
    range 192.187.1.120 192.187.1.155;
    option routers 192.187.1.1;
    option broadcast-address 192.187.1.255;
    option domain-name-servers 192.187.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.187.3.0  netmask 255.255.255.0 {
    range 192.187.3.10 192.187.3.30;
    range 192.187.3.60 192.187.3.85;
    option routers 192.187.3.1;
    option broadcast-address 192.187.3.255;
    option domain-name-servers 192.187.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.187.2.0 netmask 255.255.255.0 {
        option routers 192.187.2.1;
}
```

lalu jalankan script ``` soal1-6.sh ``` sebelumnya.

# SOAL 3
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155

## Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet 192.187.1.0 yang dilalui Switch1 dengan melakukan penetapan range IP sebagai berikut :
```
subnet 192.187.1.0  netmask 255.255.255.0 {
    range 192.187.1.50 192.187.1.88;
    range 192.187.1.120 192.187.1.155;
  ...
}

subnet 192.187.3.0  netmask 255.255.255.0 {
  ...

subnet 192.187.2.0 netmask 255.255.255.0 {
  ...
}
```
Hal ini akan membuat Client yang melalui Switch1 memiliki range IP yang sudah ditetapkan.

# SOAL 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

## Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet 192.187.3.0 yang dilalui Switch3 dengan melakukan penetapan range IP sebagai berikut :
```
subnet 192.187.1.0  netmask 255.255.255.0 {
  ...
}

subnet 192.187.3.0  netmask 255.255.255.0 {
    range 192.187.3.10 192.187.3.30;
    range 192.187.3.60 192.187.3.85;

subnet 192.187.2.0 netmask 255.255.255.0 {
  ...
}
```
Hal ini akan membuat Client yang melalui Switch3 memiliki range IP yang sudah ditetapkan.

# SOAL 5
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut

## Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet yang dilalui Switch1 dan Switch3 dengan melakukan penetapan IP DNS Server yang terhubung sebagai berikut:
```
subnet 192.187.1.0  netmask 255.255.255.0 {
   ...
    option domain-name-servers 192.187.2.2;
   ...
}

subnet 192.187.3.0  netmask 255.255.255.0 {
    ...
    option domain-name-servers 192.187.2.2;
    ...
}

subnet 192.187.2.0 netmask 255.255.255.0 {
     ...;
}
```

Lalu kita membuat sebuah script ``` named.conf.options ``` di wise yang isinya :
```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable 
        // nameservers, you probably want to use them as forwarders.  
        // Uncomment the following block, and insert the addresses replacing 
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        // dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

Lalu buat dan jalankan script ``` soal5.sh ``` dengan isi :
```
cp named.conf.options /etc/bind/named.conf.options

service bind9 restart
```

Hal ini akan membuat Client mendapatkan DNS dari WISE dan terhubung dengan internet melalui DNS.

# SOAL 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit

## Penyelesaian
Kita harus melakukan konfigurasi terhadap subnet yang dilalui Switch1 dan Switch3 dengan melakukan penetapan lama waktu dan waktu maksimal untuk peminjaman alamat IP kepada Client oleh DHCP Server sebagai berikut
```
subnet 192.187.1.0  netmask 255.255.255.0 {
    ...
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.187.3.0  netmask 255.255.255.0 {
   ...
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.187.2.0 netmask 255.255.255.0 {
   ...
}
```

Hal ini akan membuat lama waktu untuk peminjaman alamat IP kepada Client oleh DHCP Server melalui Switch1 selama 5menit (300detik) dan melalui Switch3 selama 10menit (600detik), serta waktu maksimal peminjamannya selama 115menit (6900detik).

# SOAL 7
Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13

## Penyelesaian 
Pertama, kita akan mendapatkan hwaddress pada node Eden dengan menggunakan command ip a dan akan didapatkan nilai 32:f3:ed:8f:75:19 pada link/ether
Setelah itu, dapat dibuat file temporary dhcpd7.conf dengan isi sebagai berikut
```
subnet 192.187.1.0  netmask 255.255.255.0 {
    range 192.187.1.50 192.187.1.88;
    range 192.187.1.120 192.187.1.155;
    option routers 192.187.1.1;
    option broadcast-address 192.187.1.255;
    option domain-name-servers 192.187.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.187.3.0  netmask 255.255.255.0 {
    range 192.187.3.10 192.187.3.30;
    range 192.187.3.60 192.187.3.85;
    option routers 192.187.3.1;
    option broadcast-address 192.187.3.255;
    option domain-name-servers 192.187.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.187.2.0 netmask 255.255.255.0 {
        option routers 192.187.2.1;
}

host Eden {
    hardware ethernet 32:f3:ed:8f:75:19;
    fixed-address 192.187.3.13;
}
```

Lalu buat script ``` interfaces ``` di Eden dengan isi :
```
auto eth0
iface eth0 inet dhcp
hwaddress ether 32:f3:ed:8f:75:19
```

lalu buat script ``` soal7.sh ``` di Eden dengan isi :
```
cp interfaces /etc/network/interfaces
```

Lalu buat script ``` soal7.sh``` di Westalis dengan isi :
```
cp dhcpd7.conf /etc/dhcp/dhcpd.conf

service isc-dhcp-server restart 
```
Hal itu menyebabkan IP fixed address pada node Eden yang memiliki interface eth0 berubah menjadi 192.187.3.13/24
