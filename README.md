# Guía de instalación de MISP en Sistemas Operativos Linux/Ubuntu y configuración de hardening básicos  

[![N|Solid](https://upload.wikimedia.org/wikipedia/commons/9/91/Misp-logo.png)](https://www.misp-project.org/)

> Guía de instalación de MISP en Sistemas Operativos Linux/Ubuntu y configuración de hardening básicos 
> Autor: MISP Federal  
> Versión 1.1 – 20/10/2020  
> Este tutorial cubre los pasos básicos para la instalación de una instancia MISP en Sistemas Operativos Linux Ubuntu 18.04 LTS y Ubuntu 20.04 LTS.   
> Incluye hardering del Sistema Operativo, configuración de paquetes del Sistema, instalación de MISP y configuraciones para sincronización con otras instancias.

# 0. Instalación de sistema operativo
Para el cálculo y estimación de espacio requerido se recomienda utilizar la siguiente herramienta: https://misp-project.org/MISP-sizer/ 

Atención: Configurar teclado en ingles para mayor facilidad en el traslado de comandos y símbolos a la consola.

# 1. Instalación de hardening básicos
El primer paso luego de una instalación básica del Sistema Operativo Ubuntu es configurar un firewall para proteger la instalación y evitar ataques provenientes de internet, principalmente de fuerza bruta de diccionario.

## 1.1. Configuración de Firewall
1. 1.1. Crear el archivo /root/rules.v4
```sh
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]
:IN-NEW - [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m tcp ! --tcp-flags FIN,SYN,RST,ACK SYN -m state --state NEW -j DROP
-A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP
-A INPUT -p tcp -m tcp --tcp-flags FIN,SYN FIN,SYN -j DROP
-A INPUT -p tcp -m tcp --tcp-flags SYN,RST SYN,RST -j DROP
-A INPUT -p tcp -m tcp --tcp-flags FIN,RST FIN,RST -j DROP
-A INPUT -p tcp -m tcp --tcp-flags FIN,ACK FIN -j DROP
-A INPUT -p tcp -m tcp --tcp-flags ACK,URG URG -j DROP
-A INPUT -m state --state INVALID -j DROP
-A INPUT -d 224.0.0.0/32 -j REJECT --reject-with icmp-port-unreachable
-A INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
-A INPUT -p udp -m state --state ESTABLISHED -j ACCEPT
-A INPUT -p icmp -m state --state ESTABLISHED -j ACCEPT
-A INPUT -p icmp -m state --state NEW,ESTABLISHED -m icmp --icmp-type 8 -j ACCEPT
-A INPUT -m state --state NEW -j IN-NEW
-A INPUT -j LOG --log-prefix "IPT_INPUT: " --log-level 6
-A INPUT -j DROP
-A FORWARD -j LOG --log-prefix "IPT_FORWARD: " --log-level 6
-A FORWARD -j DROP
-A OUTPUT -o lo -j ACCEPT
-A OUTPUT -p tcp -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -p udp -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -p icmp -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -j LOG --log-prefix "IPT_OUTPUT: " --log-level 6
-A OUTPUT -j DROP

#Permitir acceso SSH desde la red interna de la Organización
-A IN-NEW -s <Tu-IPv4-O-Red> -p tcp -m tcp --dport 22 --tcp-flags FIN,SYN,RST,ACK SYN -j ACCEPT

#Permitir acceso a MISP desde la red interna de la Organización
-A IN-NEW -s <Tu-IPv4-O-Red> -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -m multiport --dports 80,443 -j ACCEPT

#Permitir acceso externo a MISP a terceros
-A IN-NEW -s <IPv4-MISP-tercero> -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -m multiport --dports 80,443 -j ACCEPT

#IPs let's encrypt (opcional)
-A IN-NEW -s 66.133.109.36/32 -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN --dport 80 -j ACCEPT

#IPs ssllabs (opcional)
-A IN-NEW -s 64.41.200.0/24 -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN --dport 443 -j ACCEPT
COMMIT
```

1. 1.2. Crear el archivo /root/rules.v6
```sh
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -m rt --rt-type 0 -j DROP
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -m conntrack --ctstate INVALID -j DROP
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 1 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 2 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 3 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 4 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 128 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 129 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 133 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 134 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 135 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 136 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 141 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 142 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 148 -j ACCEPT
-A INPUT -p ipv6-icmp -m icmp6 --icmpv6-type 149 -j ACCEPT

#Permitir acceso SSH desde la red interna de la Organización
-A INPUT -s <Tu-IPv6-O-Red> -p tcp -m tcp --dport 22 -j ACCEPT

#Permitir acceso a MISP desde la red interna de la Organización
-A INPUT -s <Tu-IPv4-O-Red> -p tcp -m tcp -m multiport --dports 80,443 -j ACCEPT

# Permitir acceso externo a MISP a terceros
-A INPUT -s <IPv6-MISP-tercero> -p tcp -m tcp -m multiport --dports 80,443 -j ACCEPT

#IPs let's encrypt (opcional)
-A INPUT -s 2600:3000::/29 -p tcp -m tcp --dport 80 --tcp-flags FIN,SYN,RST,ACK SYN -j ACCEPT
-A INPUT -s 2600:1f00::/24 -p tcp -m tcp --dport 80 --tcp-flags FIN,SYN,RST,ACK SYN -j ACCEPT
-A INPUT -s 2a05:d000::/25 -p tcp -m tcp --dport 80 --tcp-flags FIN,SYN,RST,ACK SYN -j ACCEPT

#IPs ssllabs (opcional)
-A INPUT -s 2600:C02:1020:4202::/64 -p tcp -m tcp --dport 443 -j ACCEPT

-A INPUT -j LOG --log-prefix "IPT_INPUT6: " --log-level 6
-A INPUT -j REJECT --reject-with icmp6-port-unreachable
-A FORWARD -j REJECT --reject-with icmp6-port-unreachable
-A OUTPUT -j ACCEPT
COMMIT
```

1. 1.3.  Cargar las reglas del firewall con los comandos:
```sh
# iptables-restore /root/rules.v4
# ip6tables-restore /root/rules.v6
```

1. 1.4. Verificar si las reglas se cargaron con los comandos:
```sh
# iptables -nL
# ip6tables -nL
```

1. 1.5. Instalar el paquete iptables-persistent para mantener las reglas cargadas de forma permanente con los comandos:
```sh
# apt-get update
# apt-get install iptables-persistent -qy
```

1. 1.6. Luego de la instalación iptables-persistent guardará las reglas del firewall que están en la memoria, en archivos de reglas en el directorio /etc/iptables. 
Para esto, confirme con “YES” en las siguientes dos preguntas sobre IPv4 e IPv6.



   1.1.7. Verificar si los archivos con las reglas fueron creados en /etc/iptables  
.


## 1.2. Configuración y hardening de SSH
.
## 1.3. Configuración de SSHD para aceptar solo inicios de sesión usando un par de claves
.
## 1.4. Configuración zona horaria 
por ahora falla

## 1.5. Instalación y configuración de unbound 
1. 5.1. Instalar Unbound:
```sh
	# apt-get install unbound -qy
```
1. 5.2. Desactivar el resolutor del sistema con los comandos:
```sh
	# systemctl disable systemd-resolved
	# systemctl stop systemd-resolved
	# rm /etc/resolv.conf
```
1. 5.3. Crear nuevamente el archivo /etc/resolv.conf con el siguiente contenido:
```sh
nameserver ::1
nameserver 127.0.0.1
```
1. 5.4. Reiniciar el servicio de Unbound:
```sh
	# service unbound restart
```
1. 5.5. Probar si Unbound está resolviendo nombres y validando DNSSEC con consultas: 
```sh
	# dig www.dnssec-failed.org @127.0.0.1
```

## 1.6 Actualización de Ubuntu 

1. 6.1. Actualizar Ubuntu:
```sh
# apt-get update && apt-get dist-upgrade -y
```
Reiniciar el servidor de ser necesario.

## 1.7 Instalación Postfix 
**Aclaración: revisar funcionamiento. Paso de verificación de envío de mail intentado pero no comprobado** 

1. 7.1. Instalar Postfix:
```sh
# apt-get install postfix mailutils -qy
```
1. 7.2. Responder "Sitio de Internet" en la ventana "Configuración de Postfix"
 
1.7.3. Ingrese su FQDN en la siguiente pantalla  

**Recomendacion: Dejarlo como está por defecto.**  
Este dato se volverá a usar en múltiples pasos posteriores.  

1. 7.4. Comprobar si en el archivo /etc/postfix/main.cf la directiva mynetworks contiene solo direcciones de host local. El archivo debería verse así:  

```sh
# See /usr/share/postfix/main.cf.dist for a commented, more complete version


# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# See http://www.postfix.org/COMPATIBILITY_README.html -- default to 2 on
# fresh installs.
compatibility_level = 2

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
# information on enabling SSL in the smtp client.

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = <FQDN>
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = $myhostname, <FQDN>, localhost.<domínio>, , localhost
relayhost = 
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all
```

1. 7.5. Editar el archivo /etc/aliases y agregar la siguiente línea:
```sh
root: <E-MAIL-PARA-RECEBER-ALERTAS-DESSE-SERVIDOR>
```
1. 7.6. Generar una nueva base de datos de alias con el comando:

```sh
# newaliases
```
1. 7.7. Probar el envio de e-mail:
```sh
# date | mail -s "Teste de envio do `hostname`" root
```
*Este último paso no funcionó por el momento*  

## 1.8. Instalación de Cron-apt
.

# 2. Instalación MariaDB, Apache, PHP y otras dependencias de MISP
## 2.1 Instalación y hardening de mariadb

2. 1.1 Instalar mariadb con el siguiente comando:
```sh
# apt-get install mariadb-client mariadb-server -qy
```

2. 1.2 Hardening de mariadb ejecutando el comando mysql_secure_instalation:
```sh
# mysql_secure_installation
```
A continuación se solicitará la contraseña de mariadb. Como aún no hay una clave, presionar enter.

Luego preguntará si se quiere agregar una contraseña raíz: [Y/n] Y

Ingrese la nueva contraseña.

Desea remover usuarios anónimos?: [Y/n] Y

¿Deshabilitar el inicio de sesión de root de forma remota?: [Y/n] Y

¿Eliminar la base de datos de prueba y acceder a ella?: [Y/n] Y

¿Recargar tablas de privilegios ahora?:  [Y/n] Y     
  

2. 1.3 Comprobar si mariadb se está ejecutando con el comando:
```sh
# systemctl status mariadb
```

### 2.2 Instalación apache

2. 1.1 Ingresar el siguiente comando para instalar:
```sh
# apt-get install apache2 apache2-doc apache2-utils -qy
```

2. 1.2 Editar el archivo /etc/apache2/sites-available/000-default.conf

Descomentar la línea ServerName y completar con su **FQDN**. 


2. 1.3 Volver a cargar el archivo de configuración de apache con el comando:
```sh
# service apache2 reload
```
Instalar y configurar certbot es opcional si su organización tiene su propio certificado.


### 2.3 Instalación de php 7.4
2. 3.1. Agregar el repositorio de PHP para garantizar la instalación de PHP 7.4 (necesario en Ubuntu 18.04):
```sh
# add-apt-repository ppa:ondrej/php
```
2. 3.2. Instalar PHP y los módulos utilizados por MISP:
```sh
# apt-get install libapache2-mod-php7.4 php php-cli php-dev \
php-json php-xml php-mysql php7.4-opcache php-readline \
php-mbstring php-redis php-gnupg php-gd -qy
```

2. 3.3. Configurar PHP editando el archivo /etc/php/7.4/apache2/php.ini con los siguientes valores:
```sh
upload_max_filesize=50M
post_max_size=50M
max_execution_time=300
memory_limit=2048M
```
Verificar los campos expose_php y display_errors del archivo anteriormente mencionado:
```sh
expose_php=Off
display_errors=Off
```
Luego de realizar las alteraciones, reiniciar apache:
```sh
# service apache2 reload
```

2. 3.4. Probar php creando el archivo /var/www/html/phpinfo.php con el siguiente contenido:
```sh
<?php
  phpinfo()
?>
```
2. 3.5. Ingresar en el navegador para verificar que php esta funcionando correctamente:  

```sh
http://<su FQDN>/phpinfo.php

o bien

http://<su ip>/phpinfo.php
```

2. 3.6. Luego del testeo, remover el archivo phpinfo.php:
```sh
# rm /var/www/html/phpinfo.php
```


### 2.4 Instalación dependencias de MISP
2. 4.1. Instalar las dependencias de MISP:

```sh
# apt-get install curl gcc git gpg-agent make python python3 openssl \
redis-server sudo vim zip unzip virtualenv libfuzzy-dev sqlite3 moreutils \
python3-dev python3-pip libxml2-dev libxslt1-dev zlib1g-dev python-setuptools -qy
```
## 3. Instalación de MISP
Esta sección abarca la instalación de MISP, creación de la base de datos y configuración del sitio web.

### 3.1 Instalación del código de MISP

3. 1.1. - Crear las variables de ambiente con los comandos:

```sh
# export PATH_TO_MISP=/var/www/MISP
# export WWW_USER=www-data
# export SUDO_WWW='sudo -H -u www-data'
# export CAKE="/var/www/MISP/app/Console/cake"
```
3. 1.2. Verifique si las variables fueron creadas con el comando:
```sh
# export
```
3. 1.3. Crear el directorio donde se instalará MISP:
```sh
# mkdir ${PATH_TO_MISP}
# chown $WWW_USER:$WWW_USER ${PATH_TO_MISP}
```
3. 1.4. Descargar el código MISP con los comandos:
```sh
# cd ${PATH_TO_MISP}
# $SUDO_WWW git clone https://github.com/MISP/MISP.git ${PATH_TO_MISP}
# $SUDO_WWW git submodule update --init --recursive

Configuración para que GIT ignore la diferencia en los permisos en los directorios:

# $SUDO_WWW git submodule foreach --recursive git config core.filemode false
# $SUDO_WWW git config core.filemode false
```
3. 1.5. Crear el virtualenv de python:
```sh
# $SUDO_WWW virtualenv -p python3 ${PATH_TO_MISP}/venv
```
3. 1.6. Crear el directorio de caché de pip:
```sh
# mkdir /var/www/.cache/ 
# chown $WWW_USER:$WWW_USER /var/www/.cache
```
3. 1.7. Descargar e instalar los componentes de MISP:
```sh
# cd ${PATH_TO_MISP}/app/files/scripts
# $SUDO_WWW git clone https://github.com/CybOXProject/python-cybox.git
# $SUDO_WWW git clone https://github.com/STIXProject/python-stix.git
# $SUDO_WWW git clone https://github.com/MAECProject/python-maec.git

# $SUDO_WWW git clone https://github.com/CybOXProject/mixbox.git

# cd ${PATH_TO_MISP}/app/files/scripts/mixbox
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install .

# cd ${PATH_TO_MISP}/app/files/scripts/python-cybox
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install .

# cd ${PATH_TO_MISP}/app/files/scripts/python-stix
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install .

# cd $PATH_TO_MISP/app/files/scripts/python-maec
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install .

# cd ${PATH_TO_MISP}/cti-python-stix2
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install .

```
3. 1.8. Instalar PyMISP:
```sh
# cd ${PATH_TO_MISP}/PyMISP
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install .
```
3.1.9. Instalar otros paquetes MISP:

```sh
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install git+https://github.com/kbandla/pydeep.git
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install lief
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install zmq redis
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install python-magic
# $SUDO_WWW ${PATH_TO_MISP}/venv/bin/pip install plyara
```
### 3.2 Instalación  CackePHP

3. 2.1. Instalar CakePHP con los siguientes comandos:
```sh
# cd ${PATH_TO_MISP}/app
# mkdir /var/www/.composer ; sudo chown $WWW_USER:$WWW_USER /var/www/.composer
# $SUDO_WWW php composer.phar install
```
3. 2.2. Habilitar los módulos para el funcionamento de Cake:
```sh
# phpenmod redis
# phpenmod gnupg
```
3. 3.3. Habilitar usuarios de Cake:

```sh
# $SUDO_WWW cp -fa ${PATH_TO_MISP}/INSTALL/setup/config.php ${PATH_TO_MISP}/app/Plugin/CakeResque/Config/config.php
```
### 3.3 Crear base de datos de MISP

3. 3.1. Acceder a mariadb como root: 
```sh
# mysql -u root -p
```
3. 3.2. Ingresar los siguientes comandos para crear la base de datos MISP y el usuario que tendrá acceso a la base:
```sh
CREATE DATABASE misp;
CREATE USER 'misp_user'@'localhost' IDENTIFIED BY '<MISP_USER-PASSWORD>';
GRANT USAGE ON *.* to misp_user@localhost;
GRANT ALL PRIVILEGES on misp.* to 'misp_user'@'localhost';
FLUSH PRIVILEGES;
exit
```
Sugerencia para la contraseña de misp_user: Generar la siguiente cadena de caracteres, guardarla, y luego utilizarla como contraseña.
```sh
# openssl rand -base64 15
```
3. 3.2.Importar el esquema de la base de datos MISP:
```sh
# ${SUDO_WWW} cat ${PATH_TO_MISP}/INSTALL/MYSQL.sql | mysql -u misp_user misp -p
```
### 3.4 Corregir los permisos
3. 4.1.Para asegurarse de que ningún permiso se haya configurado incorrectamente, corregir todos los permisos en el directorio de instalación de MISP con los comandos:
```sh
# chown -R ${WWW_USER}:${WWW_USER} ${PATH_TO_MISP}
# chmod -R 750 ${PATH_TO_MISP}
# chmod -R g+ws ${PATH_TO_MISP}/app/tmp
# chmod -R g+ws ${PATH_TO_MISP}/app/files
# chmod -R g+ws $PATH_TO_MISP/app/files/scripts/tmp
```
### 3.5 Configurar el sitio web MISP 
  falló
### 3.6 Habilitar la rotación de registros de MISP

3. 6.1. Habilitar la rotación de registros de MISP:
```sh
# cp ${PATH_TO_MISP}/INSTALL/misp.logrotate /etc/logrotate.d/misp
# chmod 0640 /etc/logrotate.d/misp
```
## 4. Configuración de MISP
Esta sección cubre la configuración inicial de MISP.
### 4.1 Configuraciones de MISP

4. 1.1. Crear los archivos de configuración copiando los archivos predeterminados proporcionados por MISP:
```sh
# $SUDO_WWW cp -a ${PATH_TO_MISP}/app/Config/bootstrap.default.php ${PATH_TO_MISP}/app/Config/bootstrap.php
# $SUDO_WWW cp -a ${PATH_TO_MISP}/app/Config/database.default.php ${PATH_TO_MISP}/app/Config/database.php
# $SUDO_WWW cp -a ${PATH_TO_MISP}/app/Config/core.default.php ${PATH_TO_MISP}/app/Config/core.php
# $SUDO_WWW cp -a ${PATH_TO_MISP}/app/Config/config.default.php ${PATH_TO_MISP}/app/Config/config.php
```
4. 1.2. Editar la configuración de acceso a la base de datos de MISP en el archivo database.php:

```sh
# $SUDO_WWW vi $PATH_TO_MISP/app/Config/database.php

encontre as seguintes linhas:
'login' => 'misp_user',
'password' => '<MISP_USER-PASSWORD>',
'database' => 'misp',
```
**Importante:** Genere una nueva sal antes de cambiar la contraseña del usuario administrador del sistema. Se recomienda una contraseña segura generada con el comando:
```sh
# openssl rand -base64 24
```
4. 1.3. Editar el archivo config.php y configurar el nuevo salt
```sh
# $SUDO_WWW vi $PATH_TO_MISP/app/Config/config.php
Busque la línea 'salt' => '' y coloque la nueva cadena generada por el comando openssl
```
4. 1.4. Establecer los permisos correctos en los archivos de configuración con los comandos:
```sh
# chown -R $WWW_USER:$WWW_USER ${PATH_TO_MISP}/app/Config
# chmod -R 750 ${PATH_TO_MISP}/app/Config
```

### 4.2 Habilitar a los trabajadores 

4. 2.1. Habilitar el permiso de ejecución en el script que carga a los trabajadores:
```sh
# chmod +x $PATH_TO_MISP/app/Console/worker/start.sh

```
4. 2.2. Crear el archivo de configuración del servicio misp-workers editando el archivo /etc/systemd/system/misp-workers.service:

```sh
[Unit]
Description=MISP background workers
After=network.target

[Service]
Type=forking
User=www-data
Group=www-data
ExecStart=/var/www/MISP/app/Console/worker/start.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
4. 2.3. Iniciar y habilitar el daemon en el sistema:
```sh
# systemctl daemon-reload
# systemctl enable --now misp-workers
```

### 4.3 Configuración inicial de MISP
4. 3.1. Actualizar base de datos:
```sh
# $SUDO_WWW -- $CAKE userInit -q
# $SUDO_WWW -- $CAKE Admin runUpdates
```

4. 3.2. Definir el path para virtualenv:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.python_bin" "${PATH_TO_MISP}/venv/bin/python"
```

4. 3.3. Definir timeouts:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "Session.autoRegenerate" 0
# $SUDO_WWW -- $CAKE Admin setSetting "Session.timeout" 600
# $SUDO_WWW -- $CAKE Admin setSetting "Session.cookieTimeout" 3600
```

4. 3.4. Definir la URL de MISP:
```sh
# $SUDO_WWW -- $CAKE Baseurl https://<tu FQDN>
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.external_baseurl" https://<tu FQDN>
```

4. 3.5. Habilitar la organización predeterminada y configurar algunas variables:

```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.host_org_id" 1
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.email" "<seu_email>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.disable_emailing" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.contact" "<seu_email>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.disablerestalert" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.showCorrelationsOnIndex" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.default_event_tag_collection" 0
```

4. 3.6. Configurar plugins
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_services_enable" false
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_services_url" "http://127.0.0.1"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_services_port" 9000
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_timeout" 120
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_authkey" ""
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_ssl_verify_peer" false
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_ssl_verify_host" false
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Cortex_ssl_allow_self_signed" true

# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Sightings_policy" 0
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Sightings_anonymise" false
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Sightings_range" 365
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.Sightings_sighting_db_enable" false

# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.CustomAuth_disable_logout" false

# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_policy" "DROP"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_walled_garden" "127.0.0.1"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_serial" "\$date00"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_refresh" "2h"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_retry" "30m"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_expiry" "30d"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_minimum_ttl" "1h"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_ttl" "1w"
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_ns" "localhost."
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_ns_alt" ""
# $SUDO_WWW -- $CAKE Admin setSetting "Plugin.RPZ_email" "root.localhost"
```

4. 3.7. Configurar redis:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_host" "127.0.0.1" 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_port" 6379 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_database" 13 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_password" ""
```
4. 3.8. Configurar opciones por defecto:
```sh
# Force defaults to make MISP Server Settings less RED

# $SUDO_WWW -- $CAKE Admin setSetting "MISP.language" "eng"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.proposals_block_attributes" false

# Force defaults to make MISP Server Settings less YELLOW

# $SUDO_WWW -- $CAKE Admin setSetting "MISP.ssdeep_correlation_threshold" 40 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.extended_alert_subject" false 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.default_event_threat_level" 4 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.newUserText" "Dear new MISP user,\\n\\nWe would hereby like to welcome you to the \$org MISP community.\\n\\n Use the credentials below to log into MISP at \$misp, where you will be prompted to manually change your password to something of your own choice.\\n\\nUsername: \$username\\nPassword: \$password\\n\\nIf you have any questions, don't hesitate to contact us at: \$contact.\\n\\nBest regards,\\nYour \$org MISP support team" 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.passwordResetText" "Dear MISP user,\\n\\nA password reset has been triggered for your account. Use the below provided temporary password to log into MISP at \$misp, where you will be prompted to manually change your password to something of your own choice.\\n\\nUsername: \$username\\nYour temporary password: \$password\\n\\nIf you have any questions, don't hesitate to contact us at: \$contact.\\n\\nBest regards,\\nYour \$org MISP support team" 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.enableEventBlacklisting" true 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.enableOrgBlacklisting" true 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.disableUserSelfManagement" false 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.block_event_alert" false 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.block_event_alert_tag" "no-alerts=\"true\"" 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.block_old_event_alert" false 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.block_old_event_alert_age" "" 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.block_old_event_alert_by_date" "" 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.incoming_tags_disabled_by_default" false 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.maintenance_message" "Great things are happening! MISP is undergoing maintenance, but will return shortly. You can contact the administration at \$email." 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.attachments_dir" "$PATH_TO_MISP/app/files"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.download_attachments_on_load" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.title_text" "MISP"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.terms_download" false
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.showorgalternate" false
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.event_view_filter_fields" "id, uuid, value, comment, type, category, Tag.name"

# Force defaults to make MISP Server Settings less GREEN
# $SUDO_WWW -- $CAKE Admin setSetting "Security.password_policy_length" 12
# $SUDO_WWW -- $CAKE Admin setSetting "Security.password_policy_complexity" '/^((?=.*\d)|(?=.*\W+))(?![\n])(?=.*[A-Z])(?=.*[a-z]).*$|.{16,}/'
# $SUDO_WWW -- $CAKE Admin setSetting "Security.self_registration_message" "If you would like to send us a registration request, please fill out the form below. Make sure you fill out as much information as possible in order to ease the task of the administrators."
```
4. 3.9. Agregar la dirección IP en los registros:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.log_client_ip" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.log_auth" true
```
4. 3.10. Personalizar MISP:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.footermidleft" ""
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.footermidright" "Operated by <SUA_ORG>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.welcome_text_top" "<SUA_ORG>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.welcome_text_bottom" ""
```

4. 3.11. Actualizar galaxias, taxonomías entre otros objetos:
Estas actualizaciones demoran unos minutos.
```sh
# $SUDO_WWW -- $CAKE Admin updateGalaxies
# $SUDO_WWW -- $CAKE Admin updateTaxonomies
# $SUDO_WWW -- $CAKE Admin updateWarningLists
# $SUDO_WWW -- $CAKE Admin updateNoticeLists
# $SUDO_WWW -- $CAKE Admin updateObjectTemplates "1337"
```





