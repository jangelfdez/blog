---
layout: post
title: "Las opciones de almacenamiento en Azure de un vistazo"
author: "José Ángel Fernández"
categories: blog
tags: [azure, storage]
image: olddocuments.jfif
---

Empiezo a pensar que tengo algo de apego especial por los servicios de almacenamiento de Azure. Intentando refrescar en mi mente todos los servicios y funcionalidades nuevas que se han incluido en el último año relacionados con el almacenamiento, me he dado cuenta que esta idea era recurrente. Justo en [la Azure Bootcamp de 2018 presentamos una sesión](/sessions/2018-04-27-azure-bootcamp.html) [Iria](http://twitter.com/iriaq) y yo sobre el almacenamiento en Azure. 

Revisando los detalles de la misma veo que la descripción viene completamente a cuento de este artículo:

> Storage es uno de los primeros servicios disponibles en Azure desde su lanzamiento y muchas veces uno de los menos conocidos. En los últimos meses se ha incorporado cada vez más funcionalidades creando un cierto caos entre qué se puede hacer con cada tipo de almacenamiento y qué no. En esta sesión pondremos un poco de luz sobre lío y sentar las bases para optimizar tu consumo de almacenamiento en Azure.

Han pasado dos años y la sensación vuelve a ser la misma. Una serie de servicios que en un principio consideraríamos que son "simples" ya que deberían servir para almacenar nuestros datos; pero que sin embargo, con el paso del tiempo, esa simpleza se complica al incluirse nuevas características y funcionalidades que se entrecruzan. 

Tirando de la documentación de Azure junto con [un poco de papel y boli](https://www.linkedin.com/posts/jangelfdez_aunque-vivamos-en-un-mundo-digital-todav%C3%ADa-activity-6737445504919203840-gnQw) he tomado algunas notas y dibujado unos esquemas para que con un vistazo rápido pudiera refrescarlo en cualquier momento. Algo frecuente ya que cada vez resulta más complicado retener todo en la cabeza debido al ritmo de crecimiento de Azure.

Dado que es posible que a alguien más le pueda resultar útil, he consolidado esos esquemas en una única imagen que tenéis a continuación. Podéis abrirla a tamaño original si hacéis click en ella para ver mejor los detalles.

[![](/assets/img/azurestorageservices.png)](/assets/img/azurestorageservices.png)

El servicio más destacado, como era de esperar, es *Azure Storage* y la gran variedad de tipos de almacenamiento que incluye en él. Destaca especialmente *Azure Blob Storage* y , dentro de él, el servicio de *Azure Blob Block Storage* como base para otros servicios tan relevantes como *Azure Data Lake Gen 2*. 

Alrededor de de *Azure Storage*, podemos encontrar los servicios para ingesta y exportación de datos de la familia *Data Box*, los sistemas de caché de alto rendimiento para entornos HPC con la familia de servicios basados en Avere y los servicios de copia de seguridad y protección frente a desastres con los *Recovery Services*. 

Además de los propios servicios de Azure también he añadido los detalles de las capas de rendimiento y de acceso que soporta cada uno de ellos ya que dependiendo de lo que escojamos, quedarán definidos sus niveles de rendimiento y los límites máximos.

Espero que os sea útil.


*Photo by form [PxHere](https://pxhere.com/en/photo/640737)*