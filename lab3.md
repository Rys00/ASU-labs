# Lab 3 - Drukowanie

### Intro

##### Kasowanie starych maszyn i konfiguracja do ćwiczenia

```sh
/dyd/asu/cups.pl
```

##### Konta na maszynach:

- Login: `root` Hasło: `root`
- Login: `user` Hasło: `user`

### Etap 1 - Katalog współdzielony

##### PC

```
mkdir ~/cups
chmod 777 ~/cups
```

##### Host 1

`Ustawienia maszyny -> Shared Folders -> Add folder`

**Jeżeli nie pozwala kliknąć ok to wybierzmy Folder Path przy użyciu ikonki**

| pole        | wartość |
| ----------- | ------- |
| Folder Path | ~/cups  |
| Folder Name | cups    |

```sh
mkdir /CUPS
chmod 777 /CUPS
mount -t vboxsf cups /CUPS
```

Weryfikacja:

```sh
touch /CUPS/test
```

pliki powinien się pojawić na PC

### Etap 2 - Serwer CUPS

##### Host 1

```sh
apt update
apt install cups cups-pdf elinks

systemctl start cups
```

Następnie:

```sh
nano /etc/cups/cups-pdf.conf
```

Podmieniamy wartości:

```conf
# ...
Out /CUPS
# ...
UserUMask 0022
# ...
```

Aktywacja:

```sh
service cups restart
```

Następnie:

```sh
nano /etc/cups/printers.conf
```

Szukamy sekcji domyślnie zainstalowanej drukarki o nazwie `PDF` i zmieniamy pole:

**Jeżeli nie mamy drukarki możemy dodać ją przy użyciu:** `elinks http://localhost:631`

```conf
# ...
Shared Yes
# ...
```

Aktywacja:

```sh
service cups restart
```

Weryfikacja:

`lp /etc/hosts`

Na PC powinien pojawić się nowy plik pdf w `~/cups`

### Etap 3 - Udostępnianie drukarki

##### Host 1

Włączamy udostępnianie drukarek:

```sh
cupsctl --share-printers
```

Aktywacja:

```sh
service cups restart
```

##### Host 2

Sprawdzamy widoczność drukarki i ustawiamy ją jako domyślną:

```sh
lpstat -p
lpoptions -d PDF # lub inna nazwa naszej drukarki
```

Test wydruku:

```sh
echo "test" | lp
```

Na PC powinien pojawić się nowy plik (`_stdin_.pdf`) w `~/cups` o zawartości `test`
