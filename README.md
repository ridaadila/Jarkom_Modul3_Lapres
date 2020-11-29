# Jarkom_Modul3_Lapres

# LAPRES MODUL 3 JARKOM

### KELOMPOK C13:
#### 1) Rida Adila 051111840000002
#### 2) Bayu Surya B. 05111840000114

********

##### SOAL 1 : Membuat topologi

![image](https://user-images.githubusercontent.com/71973415/100538753-3909e400-3264-11eb-89e0-12f5240e712e.png)

- Syntax topologi.sh :
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.76.57 eth1=daemon,,,switch1 eth2=daemon,,,switch3 eth3=daemon,,,switch2 mem=256M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=160M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T TUBAN -e linux ubd0=TUBAN,jarkom umid=TUBAN eth0=daemon,,,switch2 mem=128M &

# Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=64M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=64M &
xterm -T BANYUWANGI -e linux ubd0=BANYUWANGI,jarkom umid=BANYUWANGI eth0=daemon,,,switch3 mem=64M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch3 mem=64M &

```

- Setting pada ``` nano /etc/sysctl.conf ```
- hilangkan tanda # -> ``` net.ipv4.ip_forward=1 ```
- Jalankan ``` sysctl -p ```
- Atur interfaces pada UML Server , dengan ``` nano /etc/network/interfaces ```

##### SURABAYA

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.76.58
netmask 255.255.255.252
gateway 10.151.76.57

auto eth1
iface eth1 inet static
address 192.168.0.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 192.168.1.1
netmask 255.255.255.0

auto eth3
iface eth3 inet static
address 10.151.77.113
netmask 255.255.255.248


```

##### MALANG

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.77.114
netmask 255.255.255.248
gateway 10.151.77.113
```

##### MOJOKERTO

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.77.115
netmask 255.255.255.248
gateway 10.151.77.113
```

##### TUBAN

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.77.116
netmask 255.255.255.248
gateway 10.151.77.113
```

- Jalankan ``` service networking restart ``` pada setiap uml yang tlah diatur interfacesnya
- Jalankan ``` iptables –t nat –A POSTROUTING –o eth0 –j MASQUERADE –s 192.168.0.0/16 ``` pada UML Surabaya
- Untuk setiap UML yang akan menginstall sesuatu, jangan lupa untuk ``` apt-get update ``` dulu
- Jika ``` apt-get update ``` masih gagal, maka ubah repo nya. Dengan cara ``` nano /etc/apt/sources.list ```. Lalu komen (#) pada tiap baris yang terdapat tulisan "kambing". Dan tambahkan syntax ``` deb http://boyo.its.ac.id/debian stretch main contrib non-free ```
********

##### SOAL 2-6
- Pada UML Surabaya, jalankan ``` apt-get install isc-dhcp-relay ``` . Lalu atur seperti gambar dibawah ini
![image](https://user-images.githubusercontent.com/71973415/100539487-7c1a8600-3269-11eb-8e89-4aec063e65f8.png)
#
- Pada UML Tuban jalankan ``` apt-get install isc-dhcp-server ``` . Lalu ``` nano /etc/default/isc-dhcp-server ``` dan atur seperti dibawah ini, dimana ``` INTERFACESv4="eth0" ```

![image](https://user-images.githubusercontent.com/71973415/100539540-d6b3e200-3269-11eb-86bb-73f298efcbb3.png)
#
- Lalu pada UML Tuban, jalankan ``` nano /etc/dhcp/dhcpd.conf ```
- Tambahkan syntax berikut :
```
subnet 10.151.77.112 netmask 255.255.255.248 {
    option routers 10.151.77.113;
}

subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.10 192.168.0.100;
    range 192.168.0.110 192.168.0.200;
    option routers 192.168.0.1;
    option broadcast-address 192.168.0.255;
    option domain-name-servers 10.151.77.114, 202.46.129.2;
    default-lease-time 300;
    max-lease-time 7200;
}

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.50 192.168.1.70;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 10.151.77.114, 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}
```
![image](https://user-images.githubusercontent.com/71973415/100539585-1e3a6e00-326a-11eb-894b-a5c08b38c937.png)
#
- Jalankan ``` service isc-dhcp-server restart ```

- Kemudian pada 4 klien ubah interfaces nya dengan syntax berikut, lalu jalankan ``` service networking restart ```

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```

- Cek pada setiap UML klien dengan mengetikkan ``` ifconfig ``` dan ``` cat /etc/resolv.conf ```

![image](https://user-images.githubusercontent.com/71973415/100539695-ca7c5480-326a-11eb-87f0-74259b0c3f54.png)

![image](https://user-images.githubusercontent.com/71973415/100539711-eed83100-326a-11eb-806d-9e09a3cbb796.png)

![image](https://user-images.githubusercontent.com/71973415/100539739-13340d80-326b-11eb-88b2-d7e7dd77b7df.png)

![image](https://user-images.githubusercontent.com/71973415/100539760-2c3cbe80-326b-11eb-8341-edcabecf976b.png)


*******

##### SOAL 7-11

- Pada UML Mojokerto ``` apt-get install squid ```
- Lalu jalankan ``` mv /etc/squid/squid.conf /etc/squid/squid.conf.bak ```
- Kemudian ``` nano /etc/squid/squid.conf ``` . Dan isikan syntax berikut :

```
include /etc/squid/acl.conf

http_port 8080
visible_hostname mojokerto

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access deny !USERS

http_access allow AVAILABLE_WORKING
http_access allow AVAILABLE_WORKING2
http_access allow AVAILABLE_WORKING3

acl MyRedirects dstdomain .google.com
deny_info http://monta.if.its.ac.id MyRedirects
http_reply_access deny MyRedirects

http_access deny all
```
![image](https://user-images.githubusercontent.com/71973415/100539890-298e9900-326c-11eb-915b-a4f6fdb19934.png)
#
- Jalankan ``` apt-get install apache2-utils ``` . Kemudian ``` htpasswd -c /etc/squid/passwd userta_c13 ``` dan untuk passnya diiskan ``` inipassw0rdta_c13 ```
- Jalankan ``` nano /etc/squid/acl.conf ```
- Isikan pada ``` nano /etc/squid/acl.conf ``` dengan :
######
![image](https://user-images.githubusercontent.com/71973415/100539984-e8e34f80-326c-11eb-8498-e109adcdafcb.png)
#
- Download halaman error dengan perintah ``` wget 10.151.36.202/ERR_ACCESS_DENIED ```
- Kemudian copy kan hasil download an tadi dengan ``` cp -r ERR_ACCESS_DENIED /usr/share/squid/errors/English/ ```
- Terakhir, restart squid dengan ``` service squid restart ```
- Tes hasil nya dengan mengunjungi website its.ac.id dan google.com pada waktu yg telah ditentukan pada no 8-9. Pada saat mengakses google.com hasil halaman harus telah ter redirect ke monta.if.its.ac.id
######
![image](https://user-images.githubusercontent.com/71973415/100540136-f3521900-326d-11eb-844f-8337552c730a.png)
#
- Hasil error page

![image](https://user-images.githubusercontent.com/71973415/100540148-0664e900-326e-11eb-8982-2f6a39943d8d.png)

**********

##### SOAL 12
- Di UML Malang install bind9 dengan ``` apt-get install bind9 -y ```
- Jalankan ``` nano /etc/bind/named.conf.local ``` dan isikan seperti pada gambar dibawah ini :
######
![image](https://user-images.githubusercontent.com/71973415/100540272-e2ee6e00-326e-11eb-9d17-4aa94e87aad5.png)
#
- Buat folder jarkom dengan perintah ``` mkdir /etc/bind/jarkom ```
- Lalu jalankan ``` cp /etc/bind/db.local /etc/bind/jarkom/janganlupa-ta.c13.pw ```
- Kemudian ``` nano /etc/bind/jarkom/janganlupa-ta.c13.pw ``` dan isikan seperti pada gambar dibawah ini :
######
![image](https://user-images.githubusercontent.com/71973415/100540346-6f009580-326f-11eb-8715-b02d37662213.png)
#
- Jalankan ``` service bind9 restart ``` . Kemudian ubah proxy di browser
#####
![image](https://user-images.githubusercontent.com/71973415/100540390-ba1aa880-326f-11eb-8bad-7f7539346b36.png)


