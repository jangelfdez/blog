---
layout: post
title: "Opciones para clonar un proyecto de Custom Vision"
author: "José Ángel Fernández"
categories: blog
tags: [azure, custom vision, cognitive services]
image: listeria.jfif
---

Como comentaba en un [artículo previo sobre Custom Vision](http://joseangelfernandez.es/blog/opciones-almacenamiento-custom-vision.html): *"Los [servicio cognitivos de Microsoft](https://azure.microsoft.com/es-es/services/cognitive-services/) facilitan la incorporación de inteligencia artificial de forma sencilla en nuestros proyectos. Uno de estos servicios es el de [*Custom Vision*](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/), con él es posible construir clasificadores o detectores de objetos a partir de un número reducido de imágenes para su entrenamiento inicial.* 

Sin embargo, podemos encontrarnos situaciones en las que no queramos tener únicamente un modelo de Custom Vision sino que nos interese a partir de un conjunto de imágenes base poder generar más de un modelo. De esta manera, se puede afinar cada uno ellos para un aspecto concreto empleando como base de entrenamiento el mismo conjunto de datos etiquetados. 

Es decir, imaginemos que tenemos un proyecto A con varios cientos de imágenes clasificadas con varias decenas de objetos ya etiquetados, ¿qué puedo hacer si tengo un proyecto B en el que necesitamos un subconjunto de estos objetos ya clasificados en esos cientos de imágenes?

Lo ideal sería que existiera un botón que permitiera clonar automáticamente un proyecto de *Custom Vision* en uno nuevo manteniendo todo el conjunto de datos y listo para empezar a ser entrenado; sin embargo, a día de hoy no es una opción que se encuentre disponible ni en el portal ni a través de los SDKs. Para habilitar esta opción de clonación sería necesario emplear las APIs REST del servicio de Custom Vision para extraer esa información y volver a cargarla de nuevo. 

El [artículo anterior](http://joseangelfernandez.es/blog/opciones-almacenamiento-custom-vision.html) proporcionaba una guía sobre los métodos concretos de la API a usar y cómo invocarlos. Sin embargo, implica realizar un desarrollo a medida para el proceso iterativo de descargar todos los datos y volver a ingestarlos. No os preocupéis, alguien pasó por lo mismo anteriormente a vosotros y en el [repositorio de GitHub de ejemplos de Azure](https://github.com/Azure-Samples) podéis encontrar ["Custom Vision Move Project"](https://github.com/Azure-Samples/custom-vision-move-project), una solución automatizada basada en el SDK de Python para exportar toda la información de un proyecto existente y generar una copia exacta en otro nuevo listo para empezar a entrenarlo.

El código es bastante claro y sencillo de leer por si tenéis algún miedo de que borre los datos o modifique la información. Básicamente es [un único fichero](https://github.com/Azure-Samples/custom-vision-move-project/blob/master/migrate_project.py) que genera dos clientes de *Custom Vision*, origen y destino, y se encarga de leer del primero los datos para escribirlos en el segundo. Algo sencillo de implementar por uno mismo pero que siempre viene bien que alguien lo haya hecho con anterioridad. Así puedes centrarte en lo de verdad importante para tu proyecto y no perder tiempo en la preparación de la infraestructura o elementos necesarios ;) 


