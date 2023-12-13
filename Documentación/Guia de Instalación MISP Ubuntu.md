# Guía de instalación de MISP en Sistemas Operativos Linux/Ubuntu  

[![N|Solid](https://upload.wikimedia.org/wikipedia/commons/9/91/Misp-logo.png)](https://www.misp-project.org/)

> Guía de instalación de MISP en Sistemas Operativos Linux/Ubuntu
> Autor: MISP Federal  
> Versión 1.2 – 20/10/2023  
> Este tutorial cubre los pasos básicos para la instalación de una instancia MISP en Sistemas Operativos Linux Ubuntu 18.04 LTS y Ubuntu 20.04 LTS.   
> Incluye configuración de paquetes del Sistema, instalación de MISP y configuraciones para sincronización con otras instancias.

**Atención:** Se recomienda configurar teclado en inglés para mayor facilidad en el traslado de comandos y símbolos a la consola.

# 1. Actualización de Ubuntu 

1.1. Actualizar Ubuntu:
```sh
# apt-get update && apt-get dist-upgrade -y
```
Reiniciar el servidor de ser necesario.  


# 2. Instalación de MISP y sus dependencias principales
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
Recordar el nombre obtenido, se solicitará en el paso siguiente.


2.2.2. Ingresar el siguiente comando:
```sh
sudo apt-get install postfix dialog -qy
```
Se presentarán una serie de configuraciones en formato visual. Siga los pasos indicados a continuación.

- Cuando se consulte por el tipo general de configuración de mail seleccione "Internet Site".

- El paso siguiente será ingresar el **FQDN** que se obtuvo en el paso 2.2.1.


**Recomendación:** Ingresar con cautela este dato. Es de gran importancia y se volverá a utilizar en pasos posteriores.  
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
# Forzar valores predeterminados para reducir los problemas "Críticos" (red) de la configuración del servidor MISP

# $SUDO_WWW -- $CAKE Admin setSetting "MISP.language" "eng"
# $SUDO_WWW -- $CAKE Admin setSetting "MISP.proposals_block_attributes" false

# Forzar valores predeterminados para reducir las configuraciones "recomendadas" (yellow) en el servidor MISP

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

# Forzar valores predeterminados para reducir las configuraciones "opcionales" (green) en el servidor MISP

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

