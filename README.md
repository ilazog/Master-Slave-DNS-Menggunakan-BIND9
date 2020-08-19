# Master Slave DNS Menggunakan BIND9

Repository ini berisikan instalasi dan konfigurasi Master-Slave DNS pada Bind9

## Task
Instalasi dan Konfigurasi
* BIND9
* Glue Record Domain

Ketentuan pengerjaan:
* Menggunakan 2 VPS dengan OS centos7
* Menggunakan domain utama (domain.tld)
* Menggunakan DNS Server BIND9
* Menggunakan Master Slave Bind9

## Instalasi dan Konfigurasi BIND9 (Master)
#### Step 1: Install BIND9
Untuk melakukan instalasi BIND9 yang pertama adalah melakukan remote ke IP VPS dengan menggunakan SSH

> ```$ ssh root@ipaddress```

Setelah berhasil login ke VPS, lakukan pembaharuan paket/repository dari system operasi Centos7 dengan perintah sebagai berikut:

> ```# yum -y update```

Setelah melakukan update system selanjutnya lakukan install epel-release

> ```# yum install epel-release -y```

Setelah melakukan install epel-release selanjutnya kita install bind9

> ```# yum install bind bind-utils -y```

#### Step 2: Konfigurasi BIND9

Kemudian lakukan backup file konfigurasi bind dengan melakukan copy file tersebut

> ```# cp /etc/named.conf /etc/named.conf.ori```

Lakukan perubahan konfigurasi named.conf dari sisi Master

> ```# vi /etc/named.conf```

Rubah isi dari file tersebut.

```
listen-on port 53 { 127.0.0.1; 103.23.20.70;}; ( masukan ip server bind )
listen-on-v6 port 53 { ::1; };
allow-query     { 127.0.0.1; 103.23.20.70; any; }; ( masukan ip server bind )
allow-query-cache { 127.0.0.1; 103.23.20.70; any ;}; ( masukan ip server bind )
```

Kemudian masukan konfigurasi Zone Master pada file konfigurasi named.conf

```
zone "padiakse.my.id" {
                type master; (masukan type disini sebagai master)
                file "/var/named/for.dns"; ( Penempatan file zona )
                allow-update { 117.53.47.189; }; ( Mengizinkan Update zone ke ip bind slave )
                allow-transfer { 117.53.47.189; }; ( Mengizinkan mentransfer zone ke ip bind slave )
                also-notify { 117.53.47.189; }; (Melakukan notify setiap kali ada peruabahan zone dari sisi master ke slave)
                notify yes;
                };
```

Setelah itu buat File Zone sesuai dengan penempatan file Zona pada file konfigurasi named.conf

> ```# vi /var/named/for.dns```

Berikut isi dari file zona Tersebut.

```
$TTL    86400
@       IN      SOA     padiakse.my.id. root.padiakse.my.id. (
                 2403202148      ;Serial yyMMddhhmm
                 3600            ;Refresh
                 1800            ;Retry
                 604800          ;Expire
                 86400           ;Minimum TTL
)
@               IN      NS      binds1.padiakse.my.id.
@               IN      NS      binds2.padiakse.my.id.
@               IN      A       103.23.20.70
binds1          IN      A       103.23.20.70
binds2          IN      A       117.53.47.189
bind            IN      A       117.53.47.189
```

Setelah file Zone selesai kita enable konfigurasi Bind9 Server dan menjalankan servicenya

```
# systemctl enable named
# systemctl start named
```

Pastikan service berjalan dengan normal.

```
# systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-03-24 21:50:22 WIB; 15h ago
  Process: 8204 ExecStop=/bin/sh -c /usr/sbin/rndc stop > /dev/null 2>&1 || /bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
  Process: 7386 ExecReload=/bin/sh -c /usr/sbin/rndc reload > /dev/null 2>&1 || /bin/kill -HUP $MAINPID (code=exited, status=0/SUCCESS)
  Process: 8215 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 8213 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
 Main PID: 8216 (named)
   CGroup: /system.slice/named.service
           └─8216 /usr/sbin/named -u named -c /etc/named.conf
```


## Instalasi dan Konfigurasi BIND9 (Slave)
#### Step 1: Install BIND9
Untuk melakukan instalasi BIND9 yang pertama adalah melakukan remote ke IP VPS dengan menggunakan SSH

> ```$ ssh root@ipaddress```

Setelah berhasil login ke VPS, lakukan pembaharuan paket/repository dari system operasi Centos7 dengan perintah sebagai berikut:

> ```# yum -y update```

Setelah melakukan update system selanjutnya lakukan install epel-release

> ```# yum install epel-release -y```

Setelah melakukan install epel-release selanjutnya kita install bind9

> ```# yum install bind bind-utils -y```

#### Step 2: Konfigurasi BIND9

Kemudian lakukan backup file konfigurasi bind dengan melakukan copy file tersebut

> ```# cp /etc/named.conf /etc/named.conf.ori```

Lakukan perubahan konfigurasi named.conf dari sisi Slave

> ```# vi /etc/named.conf```

Rubah isi dari file tersebut.

```
listen-on port 53 { 127.0.0.1; 117.53.47.189; }; ( masukan ip server bind )
listen-on-v6 port 53 { none; };
allow-query     { 127.0.0.1; 117.53.47.189; any; }; ( masukan ip server bind )
allow-query-cache { 127.0.0.1; 117.53.47.189; any ;}; ( masukan ip server bind )
```

Kemudian masukan konfigurasi Zone Slave pada file konfigurasi named.conf

```
zone "padiakse.my.id" {
                type slave; (masukan type disini sebagai Slave)
                allow-transfer { 103.23.20.70; }; ( Mengizinkan Update zone dari Ip master )
                allow-notify { 103.23.20.70; }; ( Mengizinkan mentransfer zone dari Ip master)
                masters { 103.23.20.70; }; (Merupakan IP master dari konfigurasi slave)
                file "slaves/for.dns"; ( Penempatan file zona )
                masterfile-format text; ( Berguna untuk menerjemahkan zona dari master ke dalam format text)
                notify yes;
                };
```

Setelah file Zone selesai kita enable konfigurasi Bind9 Server dan menjalankan servicenya

```
# systemctl enable named
# systemctl start named
```

Pastikan service berjalan dengan normal.

```
# systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-03-24 21:50:42 WIB; 15h ago
  Process: 16837 ExecStop=/bin/sh -c /usr/sbin/rndc stop > /dev/null 2>&1 || /bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
  Process: 16849 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 16847 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
 Main PID: 16851 (named)
   CGroup: /system.slice/named.service
           └─16851 /usr/sbin/named -u named -c /etc/named.conf
```

## Memastikan Konfigurasi Master-Slave Bind9
#### Step 1: Lakukan Pengecekan pada file Zone dari sisi SLave
Sesuai dengan file konfigurasi zone named.conf bahwa file zone akan diletakan pada folder /var/named/slaves/ maka kita harus masuk ke folder tersebut untuk memastikan

```# cd /var/named/slaves/```

Setelah itu lihat isi dari folder slaves

```
[slaves]# ls
for.dns
```

Apabila file tersebut tidak ada mohon pastikan lagi penempatan file zone pada sisi slave.

Jika sudah ditemukan kemudian kita lihat isi konfigurasi record DNS pada file Zone tersebut.

```
# cat for.dns
$ORIGIN .
$TTL 86400	; 1 day
padiakse.my.id		IN SOA	padiakse.my.id. root.padiakse.my.id. (
				2403202148 ; serial
				3600       ; refresh (1 hour)
				1800       ; retry (30 minutes)
				604800     ; expire (1 week)
				86400      ; minimum (1 day)
				)
			NS	binds1.padiakse.my.id.
			NS	binds2.padiakse.my.id.
			A	103.23.20.70=
$ORIGIN padiakse.my.id.
bind			A	117.53.47.189
binds1			A	103.23.20.70
binds2			A	117.53.47.189
```

#### Step 2: Lakukan Pengecekan dengan melakukan penambahan record dari sisi master
Lakukan penambahan record pada file zona sisi master

> ```# vi /var/named/for.dns```

Berikut isi dari file zona Tersebut.

```
$TTL    86400
@       IN      SOA     padiakse.my.id. root.padiakse.my.id. (
                 2403202149      ;Serial yyMMddhhmm
                 3600            ;Refresh
                 1800            ;Retry
                 604800          ;Expire
                 86400           ;Minimum TTL
)
@               IN      NS      binds1.padiakse.my.id.
@               IN      NS      binds2.padiakse.my.id.
@               IN      A       103.23.20.70
binds1          IN      A       103.23.20.70
binds2          IN      A       117.53.47.189
bind            IN      A       117.53.47.189
www             IN      CNAME   padiakse.my.id.
@               IN      MX      10 mail.padiakse.my.id.
@               IN      TXT     "v=spf1 a mx -all"
mail            IN      A       117.53.47.189
ftp             IN      CNAME   padiakse.my.id.
```

Setelah file Zone selesai kita enable konfigurasi Bind9 Server dan menjalankan servicenya

```
# systemctl restart named
```

Jalankan perintah berikut untuk melakukan pengecekan zona

```
# named-checkzone padiakse.my.id /var/named/for.dns
zone padiakse.my.id/IN: loaded serial 2403202149
OK
```

Lakukan pengecekan status pada sisi master.

```
# systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-03-25 13:43:41 WIB; 1min 41s ago
  Process: 8440 ExecStop=/bin/sh -c /usr/sbin/rndc stop > /dev/null 2>&1 || /bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
  Process: 7386 ExecReload=/bin/sh -c /usr/sbin/rndc reload > /dev/null 2>&1 || /bin/kill -HUP $MAINPID (code=exited, status=0/SUCCESS)
  Process: 8451 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 8449 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
 Main PID: 8452 (named)
   CGroup: /system.slice/named.service
           └─8452 /usr/sbin/named -u named -c /etc/named.conf
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: network unreachable resolving './NS/IN': 2001:503:ba3e::2:30#53
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: network unreachable resolving './DNSKEY/IN': 2001:500:a8::e#53
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: network unreachable resolving './NS/IN': 2001:500:a8::e#53
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: network unreachable resolving './DNSKEY/IN': 2001:500:2::c#53
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: network unreachable resolving './NS/IN': 2001:500:2::c#53
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: client @0x7f62c40b7800 117.53.47.189#34914 (padiakse.my.id): transfer of 'padiakse.my.id/IN': AXFR-style IXF...03202149)
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: client @0x7f62c40b7800 117.53.47.189#34914 (padiakse.my.id): transfer of 'padiakse.my.id/IN': AXFR-style IXFR ended
Mar 25 13:43:41 pdns.padiakse.my.id named[8452]: client @0x7f62c40a9060 117.53.47.189#41904: received notify for zone 'padiakse.my.id'
Mar 25 13:43:42 pdns.padiakse.my.id named[8452]: managed-keys-zone: Key 20326 for zone . acceptance timer complete: key now trusted
Mar 25 13:43:42 pdns.padiakse.my.id named[8452]: resolver priming query complete
Hint: Some lines were ellipsized, use -l to show in full.
```

Setelah itu lakukan restart service dari sisi slave

```
# systemctl restart named
```

Kemudian cek file zone pada sisi slave.

```
# cat for.dns
$ORIGIN .
$TTL 86400	; 1 day
padiakse.my.id		IN SOA	padiakse.my.id. root.padiakse.my.id. (
				2403202149 ; serial
				3600       ; refresh (1 hour)
				1800       ; retry (30 minutes)
				604800     ; expire (1 week)
				86400      ; minimum (1 day)
				)
			NS	binds1.padiakse.my.id.
			NS	binds2.padiakse.my.id.
			A	103.23.20.70
			MX	10 mail.padiakse.my.id.
			TXT	"v=spf1 a mx -all"
$ORIGIN padiakse.my.id.
bind			A	117.53.47.189
binds1			A	103.23.20.70
binds2			A	117.53.47.189
ftp			CNAME	padiakse.my.id.
mail			A	117.53.47.189
www			CNAME	padiakse.my.id.
```

Cek status dari bind slave

```
# systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-03-25 13:48:05 WIB; 7s ago
  Process: 17340 ExecStop=/bin/sh -c /usr/sbin/rndc stop > /dev/null 2>&1 || /bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
  Process: 17352 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 17350 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
 Main PID: 17354 (named)
   CGroup: /system.slice/named.service
           └─17354 /usr/sbin/named -u named -c /etc/named.conf
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './DNSKEY/IN': 2001:500:2::c#53
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './NS/IN': 2001:500:2::c#53
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './DNSKEY/IN': 2001:500:9f::42#53
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './NS/IN': 2001:500:9f::42#53
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './DNSKEY/IN': 2001:500:12::d0d#53
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './NS/IN': 2001:500:12::d0d#53
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './DNSKEY/IN': 2001:500:2f::f#53
Mar 25 13:48:05 imam.localhost named[17354]: network unreachable resolving './NS/IN': 2001:500:2f::f#53
Mar 25 13:48:06 imam.localhost named[17354]: managed-keys-zone: Key 20326 for zone . acceptance timer complete: key now trusted
Mar 25 13:48:06 imam.localhost named[17354]: resolver priming query complete
```

**Add Glue Record Pada Portal Domain**

Untuk menambahkan glue record pada domain.tld, silakan menghubungi pihak registrar domain tersebut dan dalam case ini kami menggunakan domain dari registrar Domain Cloud.

* Cara lihat registrar domain

```
$ whois domain.tld
Sponsoring Registrar PANDI ID:garuda
Sponsoring Registrar Organization:Domain Cloud
Sponsoring Registrar City:Jakarta Selatan
Sponsoring Registrar State/Province:Jakarta
Sponsoring Registrar Postal Code:12870
Sponsoring Registrar Country:ID
Sponsoring Registrar Phone:02129682828
Sponsoring Registrar Contact Email:registrar@isi.co.id
```

* Masuk pada portal domain dan pilih bagian name server masukan nama name server dan Ip Address kemudian save changes.


<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cak2.png" width="500">
<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cak3.png" width="500">

**Pengecekan untuk Master Slave BIND9**

* Cek Glue record domain

<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cak1.png" width="500">

* Cek Record Domain

```
$  dig padiakse.my.id ns @b.dns.id +short
binds1.padiakse.my.id.
binds2.padiakse.my.id.
```

```
$ dig binds1.padiakse.my.id +short
103.23.20.70
```

```
$ dig binds2.padiakse.my.id +short
117.53.47.189
```

```
$ dig padiakse.my.id @binds1.padiakse.my.id +short
103.23.20.70
```

```
$ dig padiakse.my.id @binds2.padiakse.my.id +short
103.23.20.70
```

```
$ dig bind.padiakse.my.id +short
117.53.47.189
```

* Cek Record A Domain

```
$ dig mail.padiakse.my.id +short
117.53.47.189
```

* Cek Record MX Domain

```
$ dig padiakse.my.id mx +short
10 mail.padiakse.my.id.
```

* Cek Record NS Domain

```
$ dig padiakse.my.id ns +short
binds1.padiakse.my.id.
binds2.padiakse.my.id.
```

* Cek Record TXT Domain

```
$ dig padiakse.my.id txt +short
"v=spf1 a mx -all"
```

* Cek Record CNAME Domain

```
$ dig www.padiakse.my.id cname +short
padiakse.my.id.
$ dig ftp.padiakse.my.id cname +short
padiakse.my.id.
```

Apabila dengan tools dig belum muncul untuk record tersebut kemungkinan besar record tersebut masih dalam proses *Propagasi* atau *kita salah memaasukan record tersebut* untuk itu kita dapat mengecek kembali pada table record dan apabila tidak ada masalah pada table record kita dapat melakukan pengecekan proses propagasi record domain dan berikut kami lampirkan:

<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cak4.png" width="500">
<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cak5.png" width="500">

Berikut dokumentasi dari report record DNS yang telah ditambahkan tadi.

<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cak6.png" width="500">


**Sekian dan Terima Kasih**
