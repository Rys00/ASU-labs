# Lab 4 - LDAP

### Intro

##### Kasowanie starych maszyn i konfiguracja do ćwiczenia

```sh
/dyd/asu/ldap.pl
```

##### Konta na maszynach:

- Login: `root` Hasło: `root`
- Login: `user` Hasło: `user`

### Etap 1 - Tworzenie użytkownika

##### Host 2

```sh
apt update
apt install libnss-ldap libpam-ldap ldap-utils nscd
```

##### Host 1

```sh
apt update
apt install slapd libnss-ldap libpam-ldap ldap-utils nscd
```

Następnie:

```sh
nano jan.ldif
```

Zawartość:

```ldif
dn: ou=people,dc=asu,dc=ia,dc=pw,dc=edu,dc=pl
objectClass: organizationalUnit
ou: people

dn: ou=groups,dc=asu,dc=ia,dc=pw,dc=edu,dc=pl
objectClass: organizationalUnit
ou: groups

dn: cn=users,ou=groups,dc=asu,dc=ia,dc=pw,dc=edu,dc=pl
objectClass: posixGroup
cn: users
gidNumber: 5000

dn: uid=jan,ou=people,dc=asu,dc=ia,dc=pw,dc=edu,dc=pl
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Jan
sn: Kowalski
uid: jan
uidNumber: 5000
gidNumber: 5000
homeDirectory: /home/users/jan
loginShell: /bin/bash
userPassword: jan
```

| pole        | znaczenie                                  |
| ----------- | ------------------------------------------ |
| dn          | Distinguished Name – pełna ścieżka obiektu |
| objectClass | typ obiektu LDAP                           |
| ou          | nazwa jednostki organizacyjnej             |
| cn          | common name (nazwa obiektu)                |
| posixGroup  | grupa systemu UNIX                         |
| gidNumber   | identyfikator grupy                        |
| sn          | surname (nazwisko)                         |
| uid         | login                                      |
| uidNumber   | UID użytkownika                            |
| gidNumber   | GID grupy                                  |

Aktywacja:

```sh
ldapadd -x -D "cn=admin,dc=asu,dc=ia,dc=pw,dc=edu,dc=pl" -W -f jan.ldif
```

hasło: `admin`

| opcja | znaczenie             |
| ----- | --------------------- |
| -x    | simple authentication |
| -D    | DN użytkownika        |
| -W    | zapytaj o hasło       |
| -f    | plik LDIF             |

##### Weryfikacja:

Szukamy użytkownika jan:

```sh
ldapsearch -x uid=jan
```

### Etap 2 - Konfiguracja logowania LDAP

##### Host 1 i Host 2

```sh
nano /etc/nsswitch.conf
```

Szukamy i zamieniamy pola aby określić skąd system pobiera użytkowników.:

```
passwd: files ldap
group: files ldap
shadow: files ldap
```

Następnie dodajemy LDAP jako źródło użytkowników:

```sh
auth-client-config -t nss -p lac_ldap
```

Następnie konfigurujemy uwierzytelnianie poprzez włączenie `LDAP Authentication` w:

```sh
pam-auth-update
```

Aktywacja:

```sh
service nscd restart
```

##### Weryfikacja logowania

na host 2:

Sprawdzamy czy system widzi użytkownika LDAP

```sh
getent passwd jan
```

powinno pokazać `jan:x:5000:5000:Jan:/home/users/jan:/bin/bash`

Logowanie:

```sh
su jan
```

Hasło: `jan`

Sprawdzenie katalogu domowego

```sh
ls /home/users/jan
```
