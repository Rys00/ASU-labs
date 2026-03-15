# Lab 1 - Zarządzanie siecią

### Intro

##### Kasowanie starych maszyn i konfiguracja do ćwiczenia

```sh
/dyd/asu/net.pl
```

##### Konta na maszynach:

- Login: `root` Hasło: `root`
- Login: `user` Hasło: `user`

### Etap 1 - Interfejsy i routing

##### Host 1

```sh
ip addr add 192.168.1.10/24 dev eth0
ip link set eth0 up

dhclient eth1

ip route add 192.168.2.0/24 via 192.168.1.1
```

weryfikacja:

```sh
ip addr
ip route
```

##### Host 2

```sh
ip addr add 192.168.2.20/24 dev eth0
ip link set eth0 up

dhclient eth1

ip route add 192.168.1.0/24 via 192.168.2.1
```

weryfikacja:

```sh
ip addr
ip route
```

##### Router

```sh
ip addr add 192.168.1.1/24 dev eth0
ip link set eth0 up

ip addr add 192.168.2.1/24 dev eth1
ip link set eth1 up

dhclient eth2

echo 1 > /proc/sys/net/ipv4/ip_forward
```

weryfikacja:

```sh
cat /proc/sys/net/ipv4/ip_forward
ip addr
ip route
```

##### Weryfikacja połączeń

Host 1 do Host 2

```sh
ping 192.168.2.20
traceroute 192.168.2.20
```

Host 2 do Host 1

```sh
ping 192.168.1.10
traceroute 192.168.1.10
```

Host 1 do Internet

```sh
ping 194.29.160.35
traceroute 194.29.160.35
```

### Etap 2 - Automatyczna konfiguracja

##### Host 1

```sh
nano /etc/network/interfaces
```

Zawartość:

```
auto eth0
iface eth0 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    up ip route add 192.168.2.0/24 via 192.168.1.1

auto eth1
iface eth1 inet dhcp
```

Następnie:

```sh
nano /etc/hosts
```

Zawartość:

```
192.168.1.10 host1
192.168.2.20 host2
192.168.1.1 router
```

##### Host 2

```sh
nano /etc/network/interfaces
```

Zawartość:

```
auto eth0
iface eth0 inet static
    address 192.168.2.20
    netmask 255.255.255.0
    up ip route add 192.168.1.0/24 via 192.168.2.1

auto eth1
iface eth1 inet dhcp
```

Następnie:

```sh
nano /etc/hosts
```

Zawartość:

```
192.168.1.10 host1
192.168.2.20 host2
192.168.1.1 router
```

##### Router

```sh
nano /etc/network/interfaces
```

Zawartość:

```
auto eth0
iface eth0 inet static
    address 192.168.1.1
    netmask 255.255.255.0

auto eth1
iface eth1 inet static
    address 192.168.2.1
    netmask 255.255.255.0

auto eth2
iface eth2 inet dhcp
```

Następnie:

```sh
nano /etc/sysctl.conf
```

Dodaj / odkomentuj:

```
net.ipv4.ip_forward=1
```

Następnie:

```sh
nano /etc/hosts
```

Zawartość:

```
192.168.1.10 host1
192.168.2.20 host2
192.168.1.1 router
```

### Etap 3 - Serwer FTP

##### Host 1

```sh
apt update
apt install vsftpd
```

Następnie:

```sh
nano /etc/vsftpd.conf
```

Dodaj / odkomentuj:

```
write_enable=YES
local_enable=YES
```

Aktywuj:

```sh
systemctl restart vsftpd
```

##### Host 2

```sh
apt update
apt install ftp
```

##### Test połączenia:

```sh
ftp 192.168.1.10
```

Login: `user` Hasło: `user`

```sh
put test.txt
ls
```
