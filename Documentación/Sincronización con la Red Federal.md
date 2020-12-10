# Proceso de Sincronización con el CERT AR

El CERT AR requiere para la sincronización de eventos e indicadores de compromiso de los siguientes elementos (siguiendo lineamientos de [https://www.circl.lu/doc/misp/sharing/]):

  - **Nombre de la organización** que se quiere sincronizar junto con su **UUID** (los cuales se obtienen de la instancia propia de MISP)
  - Correo para poder establecer un Sync User
  - Correo para la creación de usuarios

Posteriormente se proporcionará la **Authkey** del **Sync User**, establecido en la organización que se encuentra en el servidor del CERT AR, para poder configurar el servidor de sincronización, así como también demás datos pertenecientes a la instancia (**URL Base**, **Nombre de Instancia**, certificados, etc).


A continuación se detallan los pasos para establecer la sincronización de eventos:

1. Agregar a la organización "Red Federal - CERT.AR" utilizando el UUID brindado por el CERT-AR.

2. Crear un Sync User en la organización propia.  
	2.1. Ingresar a la pestaña "Administration" y seleccionar la opción "Add User".  
	2.2. Completar los datos según corresponda y en Rol seleccionar "Sync User".  
	**Nota:** Tildar el campo "Set password" y asignar una.   

3. Brindarle al CERT-AR la Auth key generada por el Sync User.

4. Esperar confirmación de parte del CERT-AR luego de que haya levantado el servidor con la información de **URL Base** requerida, **AuthKey** brindada, nombre de organización, certificados que correspondan y toda info relevante

5. Realizar pruebas de comunicación publicando y recibiendo eventos.
Nota: Los eventos deben ser originados por usuarios que no tengan perfil de administrador


