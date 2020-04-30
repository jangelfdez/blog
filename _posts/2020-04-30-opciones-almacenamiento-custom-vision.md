---
layout: post
title: "Opciones de almacenamiento y recuperación de imágenes en Custom Vision"
author: "José Ángel Fernández"
categories: blog
tags: [azure, custom vision, cognitive services]
image: listeria.jfif
---

Los [servicio cognitivos de Microsoft](https://azure.microsoft.com/es-es/services/cognitive-services/) facilitan la incorporación de inteligencia artificial de forma sencilla en nuestros proyectos. Uno de estos servicios es el de [*Custom Vision*](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/), con él es posible construir clasificadores o detectores de objetos a partir de un número reducido de imágenes para su entrenamiento inicial. Este artículo surge tras las dudas recogidas en una conversación con un cliente en cuanto al modelo de costes y a las opciones de guardar y recuperar las imágenes. 

El servicio de *Custom Vision* ofrece dos opciones a la hora de realizar una nueva predicción:

* **Almacenar la información**: este es el comportamiento por defecto que se obtiene al emplear la dirección URL que proporciona el portal de Custom Vision una vez que el modelo ha sido publicado. En este caso se emplea el método [*ClassifyImage*](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c15) o [*DetectImage*](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c19) dependiendo de si has construido un clasificador o un detector de objetos.

    En este escenario, por cada petición que se realiza al servicio, se almacena la nueva imagen junto con los resultados del modelo. Esto permite a posteriori poder revisarlas en [la sección de *Prediction* del portal](https://www.customvision.ai/), comprobar los resultados del modelo y añadirlas al conjunto de datos existente para utilizarlas en la siguiente interacción y mejorar los resultados.

    [A la hora de calcular los costes de uso](https://azure.microsoft.com/es-es/pricing/details/cognitive-services/custom-vision-service/) del servicio, es necesario tener en cuenta tanto el cargo por número de peticiones realizadas, como el número de imágenes inferiores a 6MB que se procesen y se almacen. 

* **No almacenar la información**: si no necesitas guardar todas las imágenes que llegan a tu modelo, es posible que te interese esta opción. En este caso será necesario que utilices los métodos [*ClassifyImageWithNoStore*](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c17) y [*DetectImageWithNoStore*](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.0/operations/5c82db60bf6a2b11a8247c1b). La dirección es similar a la del caso anterior pero añadiendo al final el segmento */nostore*. 

    En este escenario, por cada petición que se realiza al servicio, la respuesta será la misma que en el caso anterior pero ninguna de las imágenes quedarán registradas en la sección de *Prediction*. 

    [A la hora de calcular los costes de uso](https://azure.microsoft.com/es-es/pricing/details/cognitive-services/custom-vision-service/) del servicio, únicamente es necesario tener en cuenta tanto el cargo por número de peticiones realizadas. 


### ¿Cómo puedo recuperar las imágenes guardadas?

Si nos decantamos por la opción de guardar todas las imágenes, es posible que luego nos interese poder descargarlas para usarlas en otras aplicaciones, enlazarlas con algún sistema externo o cualquier otro motivo. El servicio de *Custom Vision* permite recuperar esa información a través de la API de dos formas:

* **En bruto**: en este caso únicamente nos proporcionará las URLs para descargarnos las imágenes del sistema interno de almacenamiento que emplea el servicio.

    Para ello es posible utilizar el método [*GetUntaggedImages*](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.2/operations/5dddfe4dc8d30b100855c610) 

* **Etiquetadas**: en este caso, no solo tendremos los datos del punto anterior, sino que también nos incoporará la información al respecto de las etiquetas u objetos que ha reconocido en las imágenes almacenadas.

    Para ello es posible utilizar el método [*GetTaggedImages*](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.2/operations/5dddfe4dc8d30b100855c60c)

Por defecto únicamente se devuelven 50 imágenes pero es posible modificar el número hasta 250 y paginar a través de los resultados para extraer toda la información que necesitemos. La URL no incluye la extensión de la imagen, almacenada en JPG, por lo que es recomendable que se la añadas manualmente para que si va a ser utilizada en otras aplicaciones no tengas problemas al abrirlas y manipularlas.

A modo de referencia, os dejo el resultado de una llamada para recuperar una de las imágenes etiquetadas. En este caso, de un modelo que detecta diferentes tipos de bacterias. 

```Batchfile
PS C:\Users\me> $result = Invoke-WebRequest -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v3.2/training/projects/ac69eb4a-3c8a-xxxx-xxxx-c4f4a1b10ea8/images/tagged?take=1" -Headers @{'Training-Key'=''} -Method GET

PS C:\Users\me> $result.Content
[
  {
    "id": "fe98b5e8-1765-4592-906f-25907dad1081",
    "created": "2020-04-29T18:51:50.2721924",
    "width": 1125,
    "height": 900,
    "resizedImageUri": "https://irisscuprodstore.blob.core.windows.net/i-ac69eb4a3c8axxxxxxxxc4f4a1b10ea8/i-fe98b5e817654592906xxxxxxdad1081?sv=2017-04-17&sr=b&sig=6RiqMMvEDuS3%2FtrPmiwC5WJbrpalCX3hE6kF5LNU4v0%3D&se=2020-04-30T19%3A00%3A29Z&sp=r",
    "thumbnailUri": "https://irisscuprodstore.blob.core.windows.net/i-ac69eb4a3c8axxxxxxxxc4f4a1b10ea8/t-fe98b5e817654592906xxxxxxdad1081?sv=2017-04-17&sr=b&sig=h5vH3jzVSIXgplHCqPbBVN45H5kKSVWGanJIx6ZaLpw%3D&se=2020-04-30T19%3A00%3A29Z&sp=r",
    "originalImageUri": "https://irisscuprodstore.blob.core.windows.net/i-ac69eb4a3c8axxxxxxxxc4f4a1b10ea8/i-fe98b5e817654592906xxxxxxdad1081?sv=2017-04-17&sr=b&sig=6RiqMMvEDuS3%2FtrPmiwC5WJbrpalCX3hE6kF5LNU4v0%3D&se=2020-04-30T19%3A00%3A29Z&sp=r",
    "tags": [
      {
        "tagId": "9c4c94e8-a6e2-4f5e-b0e1-fa342cec6b60",
        "tagName": "listeria",
        "created": "2020-04-29T18:51:50.2856923"
      }
    ]
  }
]
```
Si tienes curiosidad, la imagen analizada es la que se encuentra en la cabecera del artículo.