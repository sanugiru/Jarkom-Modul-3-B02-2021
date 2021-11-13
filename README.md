# LAPRES JARKOM B02 MODUL 3 2021  
### Modul 3: DHCP & Proxy Server
**Anggota:**
- 05111940000006 	[Daffa Tristan Firdaus](https://www.github.com/DaffaTristan)  
- 05111940000086 	[Nabil Fikri Arief](https://www.github.com/alwaysyu)
- 05111940000158 	[Shahnaaz Anisa Firdaus](https://www.github.com/sanugiru)

## Konfigurasi Topologi
![topologi](/screenshots/topologi.png)
- Konfigurasi Foosha

  ```
  auto eth0
  iface eth0 inet dhcp

  auto eth1
  iface eth1 inet static
  address 10.8.1.1
  netmask 255.255.255.0

  auto eth2
  iface eth2 inet static
  address 10.8.2.1
  netmask 255.255.255.0

  auto eth3
  iface eth3 inet static
  address 10.8.3.1
  netmask 255.255.255.0
  ```
- Konfigurasi EniesLobby

  ```
  auto eth0
  iface eth0 inet static
	address 10.8.2.2
	netmask 255.255.255.0
	gateway 10.8.2.1
  ```
- Konfigurasi Water7

  ```
  auto eth0
  iface eth0 inet static
	address 10.8.2.3
	netmask 255.255.255.0
	gateway 10.8.2.1
  ```
- Konfigurasi Jipangu

  ```
  auto eth0
  iface eth0 inet static
	address 10.8.2.4
	netmask 255.255.255.0
	gateway 10.8.2.1
  ```
- Konfigurasi TottoLand

  ```
  auto eth0
  iface eth0 inet static
	address 10.8.3.2
	netmask 255.255.255.0
	gateway 10.8.3.1
  ```
- Konfigurasi Skypie

  ```
  auto eth0
  iface eth0 inet static
	address 10.8.3.3
	netmask 255.255.255.0
	gateway 10.8.3.1
  ```
 - Konfigurasi Internet
 
   Foosha
   ```
   iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.8.0.0/16
   ```
   Lainnya
   ```
   echo nameserver 192.168.122.1 > /etc/resolv.conf
   ```
## Soal 1
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server 

**Pembahasan:**
- EniesLobby (DNS Server)
  ```
  apt-get update
  apt-get install bind9 -y
  service bind9 start
  service bind9 status
  ```
- Jipangu (DHCP Server)
  ```
  apt-get update
  apt-get install isc-dhcp-server -y
  ```
- Water7 (Proxy Server)
  ```
  apt-get update
  apt-get install squid -y
  service squid restart
  service squid status
  ```
  
## Soal 2
Foosha sebagai DHCP Relay

**Pembahasan:**
- Foosha (DHCP Relay)

  ```
  apt-get update
  apt-get install isc-dhcp-relay -y
  ```
  Edit file ```/etc/default/isc-dhcp-relay``` dan isi

  ```
  SERVERS=”10.8.2.4"
  INTERFACES="eth1 eth2 eth3"
  ```  
  Restart DHCP Relay

  ```
  service isc-dhcp-relay restart
  ```
- Jipangu (DHCP Server)

  Edit file ```/etc/default/isc-dhcp-server``` dan isi

  ```
  INTERFACES="eth0"
  ```
  
## Soal 3
Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

**Pembahasan:**
- Jipangu (DHCP Server)

  Edit file ```/etc/default/isc-dhcp-server``` dan isi

  ```
  INTERFACES="eth0"
  ```
  Edit file ```/etc/dhcp/dhcpd.conf``` dan isi

  ```
  subnet 10.8.2.0 netmask 255.255.255.0 {
    option routers 10.8.2.1;
  }
  subnet 10.8.1.0 netmask 255.255.255.0 {
    range 10.8.1.20 10.8.1.99;
    range 10.8.1.150 10.8.1.169;
    option routers 10.8.1.1;
    option broadcast-address 10.8.1.255;
    option domain-name-servers 10.8.2.2;
    default-lease-time 360;
    max-lease-time 7200;
  }
  ```
  Restart dan cek status DHCP Server

  ```
  service isc-dhcp-server restart
  service isc-dhcp-server status
  ```
## Soal 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

**Pembahasan:**
- Jipangu (DHCP Server)

  Edit file ```/etc/default/isc-dhcp-server``` dan isi

  ```
  subnet 10.8.3.0 netmask 255.255.255.0 {
    range 10.8.3.30 10.8.3.50;
    option routers 10.8.3.1;
    option broadcast-address 10.8.3.255;
    option domain-name-servers 10.8.2.2;
    default-lease-time 720;
    max-lease-time 7200;
  }
  ```
## Soal 5
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut

**Pembahasan:**
- Menambahkan DNS Forwarder dan comment ```dnssec-validation auto;``` pada file ```/etc/bind/named.conf.options``` 

  ```
  forwarders {
    192.168.122.1;
  };

  // dnssec-validation auto;
  allow-query{any;};
  ```

## Soal 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

**Pembahasan:**

Pada Jipangu menambahkan ```default-lease-time``` dan ```max-lease-time``` pada subnet di file ```/etc/dhcp/dhcpd.conf```
- Pada Subnet Switch1 menambahkan
  ```
  default-lease-time 360;
  max-lease-time 7200;
  ```
- Pada Subnet Switch3 menambahkan
  ```
  default-lease-time 720;
  max-lease-time 7200;
  ```
  
## Soal 7
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69 

**Pembahasan:**

## Soal 8
Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000 

**Pembahasan:**

## Soal 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy 

**Pembahasan:**

## Soal 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00) 

**Pembahasan:**

## Soal 11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

**Pembahasan:**

## Soal 12
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps 

**Pembahasan:**

## Soal 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya 

**Pembahasan:**


## Hambatan & Solusi
Tidak ada
