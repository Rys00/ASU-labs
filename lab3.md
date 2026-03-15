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

| pole        | wartość |
| ----------- | ------- |
| Folder Path | ~/cups  |
| Folder Name | cups    |

```sh
mkdir /CUPS
mount -t vboxsf cups /CUPS
```

Weryfikacja:

```sh
su anonymous
touch /CUPS/test
```

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

Zamieniamy `Out /var/spool/cups-pdf` na:

```
Out /CUPS
```

Aktywacja:

```sh
service cups restart
```

Następnie otwieramy:

```sh
elinks http://localhost:631
```

W menu `Administration -> Add Printer` wybieramy:

```
PDF Printer (CUPS-PDF)
```

Weryfikacja:

Klikamy w panelu `Printers -> Print Test Page`

Na PC powinien pojawić się nowy plik pdf w `~/cups`

### Etap 3 - Udostępnianie drukarki

##### Host 1

Wchodzimy do panelu:

```sh
elinks http://localhost:631
```

Włączamy:

- `Administration -> Share printers connected to this system`
- `Printer -> Modify Printer -> Share This Printer`

Aktywacja:

```sh
service cups restart
```

##### Host 2

Sprawdzamy widoczność drukarki:

```sh
lpstat -p
```

Test wydruku:

```sh
echo test | lp
```

Na PC powinien pojawić się nowy plik w `~/cups`
