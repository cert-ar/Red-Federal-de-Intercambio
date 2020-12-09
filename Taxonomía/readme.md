[En desarrollo]

Las taxonomías permiten clasificar, etiquetar y organizar la información. De esta manera es posible intercambiar información manejando un mismo vocabulario.

### Agregado de taxonomía privada

Se debe ir al directorio /var/www/MISP/app/files/taxonomies/, crear una carpeta para guardar la taxonomia privada, cambiar a esta última y crear el archivo machinetag.json para colocar la taxonomia deseada   [(ver contenido del .json)](/Taxonomía/CERT-AR-Taxonomía.json)  adicionalmente se detallan los códigos de colores a fin de poder customizar la clasificación 
[Criterio de colores]):

```sh
$ cd /var/www/MISP/app/files/taxonomies/
$ mkdir privatetaxonomy
$ cd privatetaxonomy
$ vi machinetag.json
```
Una vez que se crea el archivo machinetag.json, se deben colocar valores como los que se ven en el ejemplo

```sh
{
  "namespace": "ejemplo",
  "description": "Algunas palabras descriptivas",
  "version": 1,
  "predicates": [
    {
      "value": "mi-predicado",
      "expanded": "mi-predicado"
    }
  ],
  "values": [
    {
      "predicate": "mi-predicado",
      "entry": [
        {
          "value": "un-valor",
          "expanded": "Un valor"
        }
      ]
    }
  ]
}
```

Finalmente se debe ir a la GUI Web de MISP, en la sección de taxonomías y actualizar las mismas una vez se esté conforme con el archivo generado. La taxonomía creada debería estar visible. El último paso es activar las distintas etiquetas propias de esta nueva taxonomía

### Taxonomía CERT Nacional de la República Argentina

El CERT Nacional de la República Argentina define para su operación en la coordinación en la resolución de incidentes una clasificación de los diferentes tipos de incidentes.

[Criterio de colores]
>Bajo: Gris #808080  
>Medio: Verdoso #c7cd3f  
>Alto: Ambar #FFBF00  
>Crítico: Rojo #FF0000  
  
[![N|Solid](https://github.com/cert-ar/Red-Federal-de-Intercambio/blob/master/Taxonom%C3%ADa/Clasificacion.jpg)]

