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
	![2-1](/screenshots/2-1.png)

  Restart DHCP Relay

  ```
  service isc-dhcp-relay restart
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
untuk mendapatkan IP yang tetap untuk node Skyipe, digunakan fitur fixed address yang dimiliki DHCP Server, caranya: 
buka dan edit file `/etc/dhcp/dhcpd.conf` pada Jipangu, kemudian tambahkan:
```
host Skypie {
  hardware ethernet 7a:20:d2:34:16:22;
  fixed-address 10.8.3.69;
}
```
- hardware ethernet bisa kita dapatkan dengan command `ip a` pada node Skypie 
![7-1](/screenshots/7-1.png)
dan untuk fixed addressnya kita masukan sesuai yang diperintahkan soal, jika sudah lakukan restart pada dhcp server.
```
service isc-dhcp-server restart
```
untuk mengecek apa Skypie sudah memiliki IP `10.8.3.69` kita cek dengan `ip a` pada node Skypie  

![7-2](/screenshots/7-2.png)

## Soal 8
Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000 

**Pembahasan:**  
> EniesLobby  

Pertama kita buat dulu domain jualbelikapal.b02.com di EniesLobby, buka file `/etc/bind/named.conf.local` dan tambahkan script di bawah ini di dalamnya:
```
zone "jualbelikapal.b02.com" {
        type master;
        file "/etc/bind/jarkom/jualbelikapal.b02.com";
};
```
selanjutnya buat folder jarkom di dalam `/etc/bind/jarkom`
```
mkdir /etc/bind/jarkom
```
kemudian Copykan file db.local pada path /etc/bind ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi `jualbelikapal.b02.com`
```
cp /etc/bind/db.local /etc/bind/jarkom/jualbelikapalb02.com
```
Kemudian buka file `jualbelikapal.b02.com` dan masukkan script di bawah ini:
```
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     jualbelikapal.b02.com. root.jualbelikapal.b02.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      jualbelikapal.b02.com.
@       IN      A       10.8.2.3 ;IP Water7
```
IP yang digunakan merupakan IP Water7 yang merupakan proxy server. Selanjutnya restart bind9
```
service bind9 restart
```
**Testing**  

![8-1](/screenshots/8-1.png)    

IP `jualbelikapal.b02.com` sudah mengarah ke IP Water7  
> Water7  

jika sudah lakukan konfigurasi squid di node Water7. Backup terlebih dahulu file konfigurasi default yang disediakan Squid.  
```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
```
kemudain buka file `/etc/squid/squid.conf ` dan masukan 
```
http_port 5000 
visible_hostname jualbelikapal.b02.com
```
lalu save dan restart squid
```
service squid restart
```
> Loguetown  

Karena Loguetown digunakan sebagai proxy client, kita set proxy di node Loguetown
```
export http_proxy="http://jualbelikapal.b02.com:5000"
```
**Testing** 
akses its.ac.id menggunakan lynx dengan proxy
![8-2](/screenshots/8-2.png)
![8-3](/screenshots/8-3.png)

## Soal 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy 

**Pembahasan:**  
> Water7  

Di node Water7 install `apache2-utils` bila belum
```
apt-get update
apt-get install apache2-utils -y
```
Kemudian ketikkan
```
htpasswd -cm /etc/squid/passwd luffybelikapalb02
```
klik enter, kemudian ketikkan password `luffy_b02` karena password diminta dienkripsi menggunakan MD5 setelah huruf `-c` kita tambahkan `m` selanjutnya ketikkan lagi yang sebelumnya namun untuk user zoro hanya perlu `-m` untuk mengenkripsi password  
```
htpasswd -c /etc/squid/passwd zorobelikapalb02
```
klik enter lagi dan ketikkan password `zoro_b02`. isi file `etc/squid/passwd`:  

![9-1](/screenshots/9-1.png)  

selanjutanya edit `/etc/squid/squid.conf` dengan menambahkan
```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
```
terakhir, lakukan restart pada squid.  

**Testing**
> Loguetown  

akses its.ac.id menggunakan lynx dengan proxy  

![9-2](/screenshots/9-2.png)
![9-3](/screenshots/9-3.png)
![9-4](/screenshots/9-4.png)

## Soal 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00) 

**Pembahasan:**  
> Water7  

buat file `acl.conf` didalam folder squid
```
nano /etc/squid/acl.conf
```
dan tambahkan:

```
acl AVAILABLE_WORKING time MTWH 07:00-11:00
acl AVAILABLE_WORKING_2 time TWHF 17:00-23:59
acl AVAILABLE_WORKING_3 time WHFA 00:00-03:00
```
save file tersebut, kemudian edit file `squid.conf` dengan menambahkan:
```
include /etc/squid/acl.conf
http_access allow USERS AVAILABLE_WORKING
http_access allow USERS AVAILABLE_WORKING_2
http_access allow USERS AVAILABLE_WORKING_3
```
save file tersebut dan lakukan restart pada squid.

**Testing**
> Loguetown  

```
date -s '21-11-12 10:00:00'
```  

![10-1](/screenshots/10-1.png)  

```
date -s '21-11-12 22:00:00'
```  

![10-2](/screenshots/10-2.png)


## Soal 11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

**Pembahasan:**
> EniesLobby  

Konfigurasi domain duper.franky.b02.com. ketikkan:
```
nano /etc/bind/named.conf.local
```
dan tambahkan dibawah zone `jualbelikapal.com`
```
zone "super.franky.b02.com" {
        type master;
        file "/etc/bind/kaizoku/super.franky.b02.com";
};
```
lakukan step yang sama seperti di soal 8, untuk file `/etc/bind/kaizoku/super.franky.b02.com` isi dengan:
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     super.franky.b02.com. root.super.franky.b02.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      super.franky.b02.com.
@       IN      A       10.8.3.69 ;IP skypie
www     IN      CNAME   super.franky.b02.com.
```
save file dan lakukan restart pada bind.
> Skypie  

Lakukan konfigurasi web server `super.franky.b02.com` di node Skypie, pertama ketikkan:
```
apt-get update
apt get install apache2 -y
service apache2 start
```
selanjutnya, ketikkan 
```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/super.franky.b02.com.conf
```
scipt diatas akan mengcopy isi file `000-default.conf` ke dalam file konfigurasi `super.franky.b02.com.conf` kemudian buka dan edit file `super.franky.b02.com.conf` dengan menambahkan:
```
<VirtualHost *:80>
	ServerName super.franky.b02.com
        ServerAlias www.super.franky.b02.com
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.b02.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
karena directory `/var/www/super.franky.b02.com` belum ada, kita buat dahulu
```
mkdir /var/www/super.franky.b02.com
```
selanjutnya, mengikuti perintah soal kita download dan unzip file dari link
yang telah diberikan menggunakan `wget` dan `unzip`. Instalasi dapat dilakukan langsung di `/var/www/super.franky.b02.com` atau bisa juga di root, apabila dilakukan di root maka pindahkan isi directory `super.franky` ke dalam directory `/var/www/super.franky.b02.com`
```
mv super.franky/* /var/www/super.franky.a04.com
```
selanjutnya aktifkan konfigurasi
```
a2ensite super.franky.a04.com.conf
```
kemudian restart apache
```
service apache2 restart
```
> Water7
Pada node Water7, kita edit file konfigurasi squid `/etc/squid/squid.conf`
dengan menambahkan:
```
acl google dstdomain google.com
http_access deny google
deny_info http://super.franky.b02.com/ google
```
- `dstdomain` diisi `google.com` 
- `deny_info` diisi `http://super.franky.b02.com/`, situs yang akan dibuka apabila
mengakses google.
save file tersebut, kemudian ketikkan
```
echo nameserver 10.8.2.2 > /etc/resolv.conf
```
IP merupakan IP EniesLobby supaya kita dapat mengakses `http://super.franky.b02.com` 

**Testing**  

![11-1](/screenshots/11-1.png)  

langsung diredirect menuju `http://super.franky.b02.com/`  

![11-2](/screenshots/11-2.png)  

## Soal 12
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps 

**Pembahasan:**  
> Water7  

untuk mengatur bandwith buat file `acl-bandwith.conf` pada folder squid
```
nano /etc/squid/acl-bandwith.conf
```
dan ketikkan
```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd

acl luffy proxy_auth luffybelikapalb02
acl zoro proxy_auth zorobelikapalb02
acl download  url_regex -i \.jpg$ \.png$

delay_pools 2
delay_class 1 1
delay_parameters 1 1250/1250
delay_access 1 allow luffy download
delay_access 1 deny all

http_access allow luffy download
```
- `delay_pools 2` dua karena untuk luffy dan zoro
- `delay_parameters 1 1250/1250` set bandwith. karena squid menggunakan byte
maka 10 kbps di konversi terlebih dahulu menjadi byte.
- `delay_access 1 allow luffy download` izinkan luffy untuk mendownload img  

save file dan edit file konfigurasi squid dengan menambahkan
```
include /etc/squid/acl-bandwidth.conf
```
save file dan lakukan restart pada squid

**Testing**
> Loguetown  

download img:  
![12-1](/screenshots/12-1.png)  

donwload non-img (tidak ada limit, langsung terdownload):  
![12-2](/screenshots/12-2.png)

## Soal 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya 

**Pembahasan:**  
edit kembali file `acl-bandwith.conf` pada folder squid dengan menambahkan
```
delay_class 2 1
delay_parameters 2 -1/-1
delay_access 2 allow zoro
delay_access 2 deny zoro download
delay_access 2 deny all
```
- `delay_parameters 2 -1/-1` tidak ada limit pada bandwith
- `delay_access 2 deny zoro download` jangan izinkan zoro untuk mendownload file img
jika sudah, save file dan lakukan restart pada squid.

**Testing**  
> Loguetown  

download img (access denied):  
![13-1](/screenshots/13-1.png)  

donwload non-img (tidak ada limit, langsung terdownload):  
![13-2](/screenshots/13-2.png)


## Hambatan & Solusi
- Masih kesulitan dalam menulis script, jadi proses konfigurasi masih ada yang perlu diketik diulang setiap GNS3 di close
- Proxy client tidak dapat mengaksess `super.franky.b02.com`, perlu mengganti nameserver pada file `/etc/resolv.conf` dedngan ip EniesLobby di node Skypie
