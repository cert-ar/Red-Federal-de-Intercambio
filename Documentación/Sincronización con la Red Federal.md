# Proceso de Sincronización con el CERT AR

Para poder llevar a cabo la sincronización con el CERT AR se deben llevar a cabo los siguientes procedimientos:

## Parte A

Pasos para establecer la sincronización de eventos:

1. Agregar la organización "Red Federal - CERT.AR" utilizando el UUID brindado por el CERT-AR, sin tildar la opción de Local organisation

2. Crear un Sync User en la organización mencionada en el paso 1.  
	2.1. Ingresar a la pestaña "Administration" y seleccionar la opción "Add User".  
	2.2. Completar los datos según corresponda y en Rol seleccionar "Sync User".  
	**Nota:** Tildar el campo "Set password" y asignar una.   

3. Brindarle al CERT-AR la Auth key generada por el Sync User.

4. Esperar confirmación de parte del CERT-AR luego de que haya levantado el servidor con la información de **URL Base** requerida, **AuthKey** brindada, nombre de organización, certificados que correspondan y toda info relevante

5. Realizar pruebas de comunicación publicando y recibiendo eventos.
Nota: Los eventos deben ser originados por usuarios que no tengan perfil de administrador

## Parte B

El CERT AR requiere los siguientes elementos (siguiendo lineamientos de [Sharing / Synchronisation]):

  - **Nombre de la organización** que se quiere sincronizar junto con su **UUID** (los cuales se obtienen de la instancia propia de MISP)
  - Correo para poder establecer un Sync User
  - Correo para la creación de usuarios

Posteriormente se proporcionará la **Authkey** del **Sync User**, establecido en la organización que se encuentra en el servidor del CERT AR, para poder configurar el **Servidor de Sincronización**, así como también demás datos pertenecientes a la instancia (**URL Base**, **Nombre de Instancia**, certificados, etc).
Para la configuración del **Servidor** se debe: 
1. Colocar la URL Base del CERT AR
2. Colocar el nombre de la Instancia a la cual se hace referencia
3. Indicar tipo de organización local
4. Seleccionar la organización de la instancia propia a la cual se hace referencia (la que posea el **UUID** que fue compartido previamente)
5. Colocar la **Authkey** recibida
6. Seleccionar los métodos de sincronización a utilizar
7. Tildar la opción de **Self Signed**

[Sharing / Synchronisation]: <https://www.circl.lu/doc/misp/sharing/>

