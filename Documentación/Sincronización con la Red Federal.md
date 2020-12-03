# Proceso de Sincronización con el CERT AR

El CERT AR requiere para la sincronización de eventos e indicadores de compromiso de los siguientes elememtos (siguiendo lineamientos de [https://www.circl.lu/doc/misp/sharing/][df1]):

  - **Nombre de la organización** que se quiere sincronizar junto con su **UUID** (los cuales se obtienen de la instancia propia de MISP)
  - Correo para poder establecer un Sync User

Posteriormente se proporcionará la **Authkey** del **Sync User**, establecido en la organización que se encuentra en el servidor del CERT AR, para poder configurar el servidor de sincronización, así como también demás datos pertenecientes a la instancia (**URL Base**, **Nombre de Instancia**, certificados si es que se requieren, etc).
