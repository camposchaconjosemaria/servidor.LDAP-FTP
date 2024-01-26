# Servidor FTP con LDAP

![image](https://github.com/camposchaconjosemaria/servidor.LDAP-FTP/assets/114906855/7e7a9d7e-7fcf-41b9-b98f-b8b143eb1ffd)

## Índice

1. [Introducción](#1-Introducción)
2. [Instalación de paquetes](#2-Instalación-de-los-paquetes-necesarios)
3. [Configuraciones](#3-Configuraciones)
4. [Crear estructura Ldap y añadir usuarios](#4-Crear-estructura-Ldap-y-añadir-usuarios)
5. [Prueba de conexión](#5-Prueba-de-conexión)
6. [Referencias](#6-Referencias)
7. [Licencia](#7-licencia)


---

## 1. Introducción

En esta práctica vamos a realizar las configuraciones necesarias para que algunos usuarios ldap puedan autenticarse a través del servicio y tengan acceso a nuestro servidor proFTP.
Esta práctica será realizada en una instancia de AWS por lo que debemos abrir los puertos 20 y 21 en la configuración de la instancia.

## 2. Instalación de los paquetes necesarios

Para poder realizar esta práctica vamos a necesitar los siguientes paquetes:

```bash
sudo apt install proftpd
sudo apt install slapd ldap-utils
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
Reiniciamos el servicio y comprobamos la configuración con el comando ***slapcat***. En el caso que no se haya creado igual, con el comando ***#dpkg-reconfigure slapd*** podríamos solucionarlo.

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
userPassword: {MD5}r0hXtCWl+3Cck4ChQ2zI+g==  #Creada a partir del comando slappasswd -h {MD5} -s contraseña
loginShell: /bin/false
```

A continuación, incorporamos la estructura y la comprobamos con los siguientes comando:
```bash
ldapadd -x -D "cn=admin,dc=rodrigocaro,dc=net" -W -f base.ldif
ldapadd -x -D "cn=admin,dc=rodrigocaro,dc=net" -W -f usuarios.ldif
```
Solo nos faltaría crear el directorio de trabajo de rigoberta y darle los permisos correspondientes que sería:
```bash
mkdir -p /srv/ftp/rigoberta 
chgrp 2002 /srv/ftp
chown 2002 /srv/ftp/rigoberta 
chmod 755 /srv/ftp/rigoberta
```


## 5. Prueba de conexión

![image](https://github.com/camposchaconjosemaria/servidor.LDAP-FTP/assets/114906855/2fb3f5c7-b7ee-406b-9438-40353ba0ba43)

## 6. Referencias

- [Servidor proFTPd: Usuarios virtuales con LDAP](https://plataforma.josedomingo.org/pledin/cursos/servicios2008/doc/Servidor_proFTPd_Usuarios_virtuales_con_LDAP)

## 7. Licencia

<p align="center">
  <img src="https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/a4f36118-06cf-4a79-8eda-e0e029c21ff2" alt="licencia">
</p>





