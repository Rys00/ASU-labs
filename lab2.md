# Lab 2 - DNS

### Intro

##### Kasowanie starych maszyn i konfiguracja do ćwiczenia

```sh
/dyd/asu/dns.pl
```

##### Konta na maszynach:

- Login: `root` Hasło: `root`
- Login: `user` Hasło: `user`

### Etap 1 - Serwer DNS

##### Host 1

```sh
apt update
apt install bind9 dnsutils
```

Następnie:

```sh
nano /etc/bind/named.conf.local
```

Dodaj:

```
zone "asu.ia.pw.edu.pl" in {
    type master;
    file "/etc/bind/asu.dns";
};

zone "4.168.192.in-addr.arpa" in {
    type master;
    file "/etc/bind/asu.rdns";
};
```

| element          | znaczenie                              |
| ---------------- | -------------------------------------- |
| zone             | definicja strefy DNS                   |
| asu.ia.pw.edu.pl | nazwa domeny obsługiwanej przez serwer |
| type master      | serwer jest głównym źródłem danych     |
| file "asu.dns"   | plik zawierający rekordy domeny        |

Następnie:

```sh
nano /etc/bind/asu.dns
```

Zawartość:

```
$TTL 604800
@   IN  SOA host1.asu.ia.pw.edu.pl. root.asu.ia.pw.edu.pl. (
        2
        604800
        86400
        2419200
        604800 )

@       IN  NS      host1.asu.ia.pw.edu.pl.
@       IN  NS      host2.asu.ia.pw.edu.pl.

host1   IN  A       192.168.4.10
host2   IN  A       192.168.4.20
```

| pole                   | znaczenie             |
| ---------------------- | --------------------- |
| @                      | bieżąca domena        |
| SOA                    | początek opisu domeny |
| host1.asu.ia.pw.edu.pl | główny serwer DNS     |
| root.asu.ia.pw.edu.pl  | email administratora  |

| parametr SOA | znaczenie                           |
| ------------ | ----------------------------------- |
| Serial       | numer wersji strefy                 |
| Refresh      | co ile sekund slave sprawdza zmiany |
| Retry        | po jakim czasie powtórzyć próbę     |
| Expire       | po jakim czasie dane są nieważne    |
| Minimum      | minimalny TTL                       |

`IN NS host1.asu.ia.pw.edu.pl.` znaczy, że serwer DNS tej domeny = host1

Ostatnie linijki mapują nazwy hostów do ich adresów

Następnie:

```sh
nano /etc/bind/asu.rdns
```

Zawartość:

```
$TTL 604800
@   IN  SOA host1.asu.ia.pw.edu.pl. root.asu.ia.pw.edu.pl. (
        2
        604800
        86400
        2419200
        604800 )

@       IN  NS      host1.asu.ia.pw.edu.pl.
@       IN  NS      host2.asu.ia.pw.edu.pl.

10   IN  PTR host1.asu.ia.pw.edu.pl.
20   IN  PTR host2.asu.ia.pw.edu.pl.
```

Weryfikacja:

```sh
named-checkconf /etc/bind/named.conf.local
named-checkzone asu.ia.pw.edu.pl /etc/bind/asu.dns
named-checkzone 4.168.192.in-addr.arpa /etc/bind/asu.rdns
```

Aktywacja:

```sh
service bind9 restart
```

##### Host 2

```sh
apt update
apt install bind9 dnsutils
```

Następnie

```sh
nano /etc/resolv.conf
```

Zawartość:

**Komentujemy istniejące tam nameserver-y!**
Dodajemy własne:

```
domain asu.ia.pw.edu.pl
nameserver 192.168.4.10
```

| linia      | znaczenie         |
| ---------- | ----------------- |
| domain     | domyślna domena   |
| nameserver | adres serwera DNS |

##### Weryfikacja połączeń

Z host 2:

```sh
nslookup host1.asu.ia.pw.edu.pl
nslookup 192.168.4.10
ping host1.asu.ia.pw.edu.pl
```

### Etap 2 - DNS slave

##### Host 2

```sh
nano /etc/bind/named.conf.local
```

Zawartość:

```
zone "asu.ia.pw.edu.pl" in {
    type slave;
    file "/etc/bind/asu.dns";
    masters { 192.168.4.10; };
};

zone "4.168.192.in-addr.arpa" in {
    type slave;
    file "/etc/bind/asu.rdns";
    masters { 192.168.4.10; };
};
```

Aktywacja:

```sh
service restart bind9
```

##### Test serwera slave

Z host 2:

```sh
nslookup host1.asu.ia.pw.edu.pl 192.168.4.20
```

### Etap 3 - NFS

##### Host 1

```sh
apt install nfs-kernel-server
mkdir /pub
chmod 777 /pub
```

Następnie:

```sh
nano /etc/exports
```

Dodaj:

```
/pub 192.168.4.20(rw,sync,no_subtree_check)
```

| element          | znaczenie                 |
| ---------------- | ------------------------- |
| /pub             | katalog udostępniony      |
| 192.168.4.20     | klient NFS                |
| rw               | odczyt i zapis            |
| sync             | zapis synchroniczny       |
| no_subtree_check | brak sprawdzania poddrzew |

Aktywacja:

```sh
exportfs -a
service nfs-kernel-server restart
```

Weryfikacja:

```sh
exportfs
```

##### Host 2

```sh
apt install nfs-common
mkdir /host1
mount 192.168.4.10:/pub /host1
```

Następnie:

```sh
nano /etc/fstab
```

Dodajemy:

```
host1:/pub /host1 nfs defaults 0 0
```

| pole       | znaczenie          |
| ---------- | ------------------ |
| host1:/pub | zasób NFS          |
| /host1     | katalog montowania |
| nfs        | typ systemu plików |
| defaults   | domyślne opcje     |
| 0          | brak dump          |
| 0          | brak fsck          |

##### Test NFS

Z hosta 2:

```sh
touch /host1/test
```

Z hosta 1:

```sh
ls /pub
touch /pub/test2
```

Z hosta 2:

```sh
ls /host1
```
