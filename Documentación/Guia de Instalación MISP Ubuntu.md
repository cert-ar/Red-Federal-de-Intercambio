# Guía de instalación de MISP en Sistemas Operativos Linux/Ubuntu y configuración de hardening básicos  

[![N|Solid](https://upload.wikimedia.org/wikipedia/commons/9/91/Misp-logo.png)](https://www.misp-project.org/)

> Guía de instalación de MISP en Sistemas Operativos Linux/Ubuntu y configuración de hardening básicos 
> Autor: MISP Federal  
> Versión 1.1 – 20/10/2020  
> Este tutorial cubre los pasos básicos para la instalación de una instancia MISP en Sistemas Operativos Linux Ubuntu 18.04 LTS y Ubuntu 20.04 LTS.   
> Incluye hardering del Sistema Operativo, configuración de paquetes del Sistema, instalación de MISP y configuraciones para sincronización con otras instancias.

# 0. Instalación de sistema operativo
Para el cálculo y estimación de espacio requerido se recomienda utilizar la siguiente herramienta:   
https://misp-project.org/MISP-sizer/ 

**Atención:** Se recomienda configurar teclado en inglés para mayor facilidad en el traslado de comandos y símbolos a la consola.

# 1. Configuración de hardening básicos
El primer paso luego de una instalación básica del Sistema Operativo Ubuntu es configurar un firewall para proteger la instalación y evitar ataques provenientes de internet, principalmente de fuerza bruta de diccionario.

## 1.1. Configuración de Firewall
**Aclaración:** a analizar en base a implementación propia.

1.1.1. Crear el archivo /root/rules.v4
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

1.1.2. Crear el archivo /root/rules.v6
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

1.1.3.  Cargar las reglas del firewall con los comandos:
```sh
# iptables-restore /root/rules.v4
# ip6tables-restore /root/rules.v6
```

1.1.4. Verificar si las reglas se cargaron con los comandos:
```sh
# iptables -nL
# ip6tables -nL
```

1.1.5. Instalar el paquete iptables-persistent para mantener las reglas cargadas de forma permanente con los comandos:
```sh
# apt-get update
# apt-get install iptables-persistent -qy
```

1.1.6. Luego de la instalación iptables-persistent guardará las reglas del firewall que están en la memoria, en archivos de reglas en el directorio /etc/iptables. 
Para esto, confirme con “YES” en las siguientes dos preguntas sobre IPv4 e IPv6.


1.1.7. Verificar si los archivos con las reglas fueron creados en /etc/iptables  


## 1.2. Configuración y hardening de SSH

1.2.1. Generar el par de claves con el siguiente comando:
```sh
$ ssh-keygen -t ed25519 -q -f /path/de/clave/misp_ed25519 -C 'MISP'
```
1.2.2. Verificar si se generó el par de claves:
```sh
$ ls -la /path/de/clave/misp_ed25519
```
1.2.3. Copiar el contenido de la clave pública en el archivo /root/.ssh/authorized_keys en el servidor MISP:

```sh
$ cat /path/da/chave/misp_ed25519.pub
```
## 1.3. Configurar sshd para aceptar solo inicios de sesión usando un par de claves
1.3.1. Si el servidor no tiene sshd instalado, instalarlo con el comando:

```sh
# apt-get install openssh-server -qy

```
1.3.2. Editar el archivo / etc / ssh / sshd_config y cambie los siguientes valores:

```sh
PermitRootLogin prohibit-password
PubkeyAuthentication yes
PasswordAuthentication no
```
1.3.3. Reiniciar el servicio ssh con el comando:
```sh
# service sshd restart
```

## 1.4. Instalación y configuración de unbound 
1.4.1. Instalar Unbound:
```sh
	# apt-get install unbound -qy
```
1.4.2. Desactivar el resolutor del sistema con los comandos:
```sh
	# systemctl disable systemd-resolved
	# systemctl stop systemd-resolved
	# rm /etc/resolv.conf
```
1.4.3. Crear nuevamente el archivo /etc/resolv.conf con el siguiente contenido:
```sh
nameserver ::1
nameserver 127.0.0.1
```
1.4.4. Reiniciar el servicio de Unbound:
```sh
	# service unbound restart
```
1.4.5. Probar si Unbound está resolviendo nombres y validando DNSSEC: 
```sh
	### RESULTADO con validación de DNSSEC
	# dig www.dnssec-failed.org @127.0.0.1
	
	; <<>> DiG 9.11.3-1ubuntu1.13-Ubuntu <<>> www.dnssec-failed.org
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 6943
	;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 4096
	;; QUESTION SECTION:
	;www.dnssec-failed.org.		IN	A

	;; Query time: 502 msec
	;; SERVER: 127.0.0.1#53(127.0.0.1)
	;; WHEN: Thu Sep 10 02:08:51 UTC 2020
	;; MSG SIZE  rcvd: 50
```

## 1.5 Actualización de Ubuntu 

1.5.1. Actualizar Ubuntu:
```sh
# apt-get update && apt-get dist-upgrade -y
```
Reiniciar el servidor de ser necesario.  


# 2. Instalación de MISP y sus dependencias
Esta sección abarca la instalación de MISP para Ubuntu 20.04 

## 2.1. Instalación de MISP y sus dependencias principales
 A continuación se instalará el Core de MISP junto con MariaDB, Apache, PHP, CakePHP y otras dependencias.

```sh
wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh

bash /tmp/INSTALL.sh
```

```sh
wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh

bash /tmp/INSTALL.sh -c
```
**Nota:** este último comando demora bastante tiempo. Se recomienda no interrumpir el proceso hasta que finalice por completo.

## 2.2. Instalación de Postfix

2.2.1. Para averiguar su FQDN, también conocido como Nombre de Dominio Completo, ingrese el siguiente comando:
```sh
hostname --fqdn
```
Recuerde el nombre obtenido, se solicitará en el paso siguiente.


2.2.2. Ingrese el siguiente comando:
```sh
sudo apt-get install postfix dialog -qy
```
Se presentarán una serie de configuraciones en formato visual. Siga los pasos indicados a continuación.

- Cuando se consulte por el tipo general de configuración de mail seleccione "Internet Site".

- El paso siguiente será ingresar el **FQDN** que se obtuvo en el paso 2.2.1.


**Recomendacion:** Ingresar con cautela. Este dato es de gran importancia y se volverá a utilizar en pasos posteriores.  
En caso de haber ingresado un FQDN erróneo se recomienda reconfigurar Postfix con el siguiente comando:
dpkg-reconfigure postfix  



# 3. Configurar el sitio web MISP 
  
## 3.1. Ingresar al sitio web MISP

Desde el navegador ingrese el IP de su servidor para poder visualizar el sitio de bienvenida.

Las credenciales iniciales son:

- User: admin@admin.test

- Password: admin

**Nota:** No olvidar cambiar el correo electrónico, la contraseña y la clave de autenticación después de la instalación.

# 4. Configuración de MISP

## 4.1 Configuración inicial de MISP

Una vez cambiadas las credenciales, acceder a la sección "Administration" - "Server Settings & Maintenance" - "MISP settings".

Allí se solicitará atención a algunas configuraciones. Se podrán resolver tanto en el panel visual como por consola.

A continuación se detalla la configuración recomendada para realizar si decide utilizar consola. También será util para tener un ejemplo de cada campo si se está configurando de manera visual.



4.1.1. Definir la URL de MISP:
```sh
# $SUDO_WWW -- $CAKE Baseurl https://<tu FQDN>
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.external_baseurl" https://<tu FQDN>
```
**Nota:** si la configuración actual de Baseurl figura como https://< tu IP > es correcto dejarlo de esa forma.


4.1.2. Habilitar la organización predeterminada y configurar algunas variables:

```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.host_org_id" 1
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.email" "<tu_email>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.disable_emailing" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.contact" "<tu_email>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.disablerestalert" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.showCorrelationsOnIndex" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.default_event_tag_collection" 0
```

4.1.3. Configurar plugins
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

4.1.4. Configurar redis:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_host" "127.0.0.1" 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_port" 6379 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_database" 13 
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.redis_password" ""
```
4.1.5. Configurar opciones por defecto:
```sh
# Forzar valores predeterminados para hacer que la configuración del servidor MISP sea menos ROJA

# $SUDO_WWW -- $CAKE Admin setSetting "MISP.language" "eng"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.proposals_block_attributes" false

# Forzar valores predeterminados para hacer que la configuración del servidor MISP sea menos AMARILLA

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

# Forzar valores predeterminados para hacer que la configuración del servidor MISP sea menos VERDE

# $SUDO_WWW -- $CAKE Admin setSetting "Security.password_policy_length" 12
# $SUDO_WWW -- $CAKE Admin setSetting "Security.password_policy_complexity" '/^((?=.*\d)|(?=.*\W+))(?![\n])(?=.*[A-Z])(?=.*[a-z]).*$|.{16,}/'
# $SUDO_WWW -- $CAKE Admin setSetting "Security.self_registration_message" "If you would like to send us a registration request, please fill out the form below. Make sure you fill out as much information as possible in order to ease the task of the administrators."
```
4.1.6. Agregar la dirección IP en los registros:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.log_client_ip" true
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.log_auth" true
```
4.1.7. Personalizar MISP:
```sh
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.footermidleft" ""
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.footermidright" "Operated by <SU_ORG>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.welcome_text_top" "<SU_ORG>"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.welcome_text_bottom" ""
```

4.1.8. Actualizar galaxias, taxonomías entre otros objetos:
Estas actualizaciones demoran unos minutos.
```sh
# $SUDO_WWW -- $CAKE Admin updateGalaxies
# $SUDO_WWW -- $CAKE Admin updateTaxonomies
# $SUDO_WWW -- $CAKE Admin updateWarningLists
# $SUDO_WWW -- $CAKE Admin updateNoticeLists
# $SUDO_WWW -- $CAKE Admin updateObjectTemplates "1337"
```

# Bibliografía

- https://misp.github.io/MISP/INSTALL.ubuntu2004/

- https://cert.br/misp/tutorial-ubuntu/

