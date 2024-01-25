# servidor.LDAP-FTP

## Índice

1. [Introducción](#1-Introducción)
2. [Instalación de paquetes](#2-Instalación-de-los-paquetes-necesarios)
3. [Configuraciones](#3-Configuraciones)
4. [Prueba de conexión](#4-Crear-estructura-Ldap-y-añadir-usuarios)
5. [Referencias](#5-Prueba-de-conexión)
6. [Licencia](#6-licencia)

---

## 1. Introducción

En esta práctica vamos a realizar las configuraciones necesarias para que algunos usuarios ldap puedan autenticarse a través del servicio y tengan acceso a nuestro servidor proFTP.
Esta práctica será realizada en una instancia de AWS por lo que debemos abrir los puertos 20 y 21 en la configuración de la instancia.

## 2. Instalación de los paquetes necesarios

Para poder realizar esta práctica vamos a necesitar los siguientes paquetes:

```bash
sudo apt install proftpd
sudo apt install slapd
sudo apt install proftpd-mod-ldap
```

## 3. Configuraciones

Editamos el fichero /etc/proftpd/modules.conf y descomentamos las líneas:

```bash
LoadModule mod_ldap.c
```

Editamos el fichero /etc/proftpd/proftpd.conf y descomentamos la línea:
```bash
Include /etc/proftpd/ldap.conf
DefaultRoot ~  #Para enjaular a los usuarios en sus directorios
RequireValidShell off
```

Editamos el fichero /etc/proftpd/ldap.conf, en el que dejamos la siguiente línea:
```bash
LDAPServer ldap://localhost/??sub LDAPBindDN "cn=admin,dc=rodrigocaro,dc=net" "peque2024" LDAPDoAuth on "ou=people,dc=rodrigocaro,dc=net"
```
Además debemos añadir las siguientes dos líneas con los datos de nuestro LDAP:
```bash
LDAPUsers ou=people,dc=rodrigocaro,dc=net (uid=%u) (uidNumber=%u)
LDAPGroups ou=group,dc=rodrigocaro,dc=net uniqueMember=%u
```

## 4. Crear estructura Ldap y añadir usuarios
Creamos los archivos ldif correspondientes. Yo lo voy a realizar en dos archivos, en el primero inserto la base y en el segundo los grupos y usuarios:

base.ldif:
```bash
dn: ou=group,dc=rodrigocaro,dc=net
objectClass: organizationalUnit
objectClass: top
ou: group

dn: ou=people,dc=rodrigocaro,dc=net
objectClass: organizationalUnit
objectClass: top
ou: people
```

usuarios.ldif:
```bash
dn: cn=ftpusers,ou=group,dc=rodrigocaro,dc=net
objectClass: posixGroup
objectClass: top
cn: ftpusers
gidNumber: 2002

dn: uid=rigoberta,ou=people,dc=rodrigocaro,dc=net
uid: rigoberta
cn: Rigoberta
objectclass: account
objectclass: posixAccount
objectclass: top
uidNumber: 2002
gidNumber: 2002
homeDirectory: /srv/ftp/rigoberta
userPassword: {MD5}r0hXtCWl+3Cck4ChQ2zI+g==
loginShell: /bin/false
```

A continuación, incorporamos la estructura y la comprobamos con los siguientes comando:
```bash
ldapadd -x -D "cn=admin,dc=rodrigocaro,dc=net" -W -f base.ldif
ldapadd -x -D "cn=admin,dc=rodrigocaro,dc=net" -W -f usuarios.ldif
```
Solo nos faltaría crear el directorio de trabajo de rigoberta y darle los permisos correspondientes que sería:
```bash
mkdir -p /srv/ftpusers/rigoberta 
chgrp 2002 /srv/ftpusers 
chown 2002 /srv/ftpusers/rigoberta 
chmod 755 /srv/ftpusers/rigoberta
```

Podríamos hacerlo de forma automática con la activación del siguiente módulo en el proftpd.conf
```bash
<IfModule mod_ifsession.c>
  <IfGroup special>
    CreateHome on 755 dirmode 755
  </IfGroup>

  <IfGroup !special>
    CreateHome on 711 dirmode 711
  </IfGroup>
</IfModule>
```


## 5. Prueba de conexión

![image](https://github.com/camposchaconjosemaria/servidor.LDAP-FTP/assets/114906855/2fb3f5c7-b7ee-406b-9438-40353ba0ba43)








