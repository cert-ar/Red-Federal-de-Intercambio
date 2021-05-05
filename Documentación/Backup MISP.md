# Backup de MISP

La estrategia de backup que se detalla a continuación consta de guardar todos los archivos de configuración de la plataforma en formato comprimido.

### 1. Crear carpeta backup y subcarpetas
```sh
# mkdir backup
# cd backup
# mkdir -p ./var/www/MISP/app/Config
# mkdir -p ./etc/apache2/sites-enabled
# mkdir -p ./etc/php/7.4/apache2/
# mkdir -p ./etc/ssl/private/
```


### 2. Copiar los archivos de configuración


```sh
# echo 'Copiando los archivos de configuracion de MISP'
# cp /var/www/MISP/app/Config/* ./var/www/MISP/app/Config/

# echo 'Copiando las credenciales de cifrado'
# cp /var/www/MISP/.gnupg ./var/www/MISP/.gnupg

# echo 'Copiando la configuracion de MISP Apache' 
# cp /etc/apache2/sites-enabled/misp-ssl.conf ./etc/apache2/sites-enabled/misp-ssl.conf

# echo 'Copiando la configuracion de Apache Php'
# cp /etc/php/7.4/apache2/php.ini ./etc/php/7.4/apache2/php.ini

# echo 'Copiando los archvios del certificado SSL para SSL'
# cp /etc/ssl/private/* ./etc/ssl/private/

# echo 'Generando el dump de la BD'
# mysqldump -u misp -p misp > ./misp_db_bkp.sql.dump

# echo 'Copiando informacion extra'
#cp /var/www/MISP/app/webroot/img/orgs ./var/www/MISP/app/webroot/img/orgs
#cp /var/www/MISP/app/webroot/img/custom ./var/www/MISP/app/webroot/img/custom
# cp /var/www/MISP/app/files ./var/www/MISP/app/files
```

### 3. Salir del directorio


```sh
# cd ..
```
### 4. Comprimir la información obtenida
```sh
# tar -zcvf backup-$(date "+%b_%d_%Y_%H_%M_%S").tar.gz backup
```

### 5. Exportar

Exporte el archivo comprimido fuera del servidor de MISP. La forma de realizar este paso dependerá del sistema operativo y la estrategia de almacenado de Backup que tenga su organización.