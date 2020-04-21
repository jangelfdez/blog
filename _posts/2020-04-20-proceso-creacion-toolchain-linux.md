---
layout: post
title: "El proceso inicial para la generación de una toolchain en Linux"
author: "José Ángel Fernández"
categories: blog
tags: [programming, linux, toolchain]
image: toolchain.jfif
---

La curiosidad por saber cómo funcionan las cosas es algo que siempre he tenido. Esta curiosidad a veces es buena aunque otras me ha dado alguna sopresa como dejar sin luz a algún edificio entero o a nuestras propias oficinas en Madrid. Sin embargo, eso son otras historias.

Una de esas cosas que desde hace tiempo me generaba curiosidad era saber cómo realmente un sistema basado en GNU/Linux funcionaba, cómo las diferentes piezas que lo forman hacen un todo para que arranque y puedas disponer de un sistema operativo funcional y multipropósito.

Gerard Beekmans tuvo esa misma curiosidad y escribió el libro [*"Linux From Scratch"*](http://www.linuxfromscratch.org/), una guía paso a paso que proporciona las instrucciones para construir un sistema GNU/Linux funcional desde cero. Para ello, se parte del propio código fuente de todos los elementos que lo integran, se compila y se aplica la configuración necesaria a cada elemento. Hace ya varios años que lo hice por primera vez con el objetivo de generar la imagen base de un contenedor que funcionara sobre Docker utilizando [*scratch*](https://hub.docker.com/_/scratch/#!) únicamente como base. Durante esta larga cuarentena volví a construir uno nuevo, esta vez utilizando *systemd* como sistema de arranque en lugar de *SysVinit*, ¿por qué no?.

Uno de los pasos que más me rechinó la primera vez que lo hice fue la creación del sistema temporal y la doble compilación de algunas de las herramientas base. La primera vez me limité a seguir los pasos del libro sin prestar demasiada atención a lo que hacía. El objetivo de tener un sistema Linux desde cero era más emocionante que entender los pasos para llegar a él. Sin embargo, en esta segunda pasada me ha dado por investigar un poco más del porqué y del cómo de este proceso. 

En el paso de crear el sistema de herramientas temporal, lo que se está realizando es la generación de un compilador cruzado *(cross-compiler)*. Cuando hablamos de una compilación cruzada, nos referimos al proceso por el cuál necesitamos transformar nuestro código escrito en nuestra máquina de desarrollo *(host)* a otra plataforma donde se va a ejecutar *(target)*. Sin embargo, mi duda estaba ahí, si estoy compilando sobre Linux en x86_64 y mi objetivo es ejecutar sobre la misma platforma, ¿por qué necesito un compilador cruzado en lugar de utilizar directamente el compilador nativo del sistema operativo? 

Para ello, el primer paso era entender cuáles son los elementos mínimos que forman una *toolchain* básica para un sistema de compilación cruzada basada en Linux y la [colección de compiladores *GCC*](https://gcc.gnu.org/). Éstos son:

* **binutils**: grupo de utilidades para la manipulación de archivos binarios. Las más importantes son el *linker (ld)* y el *assembler (as)*
* **gcc**: el compilador en sí y los runtime de ejecucción
* **Linux kernel headers**: las definiciones de las llamadas al sistema o estructuras de datos entre otros.
* **C library**: implementación de las funciones del estándar POSIX junto con otras funciones. Se basa en las llamadas al sistema del núcleo de Linux.

Para poder incluir estos elementos en nuestra distribución de Linux, el código fuente se ha compilado con configuración específica y unas suposiciones que son válidas para el sistema de destino que se tenía en mente. En el caso de que el sistema destino fuese diferente, como es el caso de una proceso de compilación cruzada, es posible que se produzca lo que se denomina una [contaminación de las herramientas *("toolchain leak")*](https://landley.net/writing/docs/cross-compiling.html) al usar esas herramientas de forma directa. 

Esto implica que información del sistema *host* se introduzcan en las aplicaciones compiladas provocando fallos. Algunos de estos errores pueden ser la inclusión de cabeceras incorrectas o el uso de librerías erróneas por un fallo en las rutas.

Si cogemos el [propio código de muestra](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/gcc.html) que aparece en el libro para verificar que nuestra *toolchain* está correctamente configurada podemos ver las cosas que un compilador mal configurado puede provocar. El código es bastante sencillo y se almacena en el fichero *dummy.c*

```bash
echo 'int main(){}' > dummy.c
``` 
Tras ello, se compila de forma verbosa para obtener todos los detalles que el compilador genera durante el proceso.

```bash
cc dummy.c -v -Wl,--verbose &> dummy.log
``` 

En mi caso, estoy trabajando sobre una CentOS Stream 8, por lo que si lo pruebas en una distribución diferente, los resultados podrán variar. Dicho esto, podemos revisar cuál es el [*dynamic linker*](https://en.wikipedia.org/wiki/Dynamic_linker) que está empleando:

```bash
$ readelf -l a.out | grep ': /lib'
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
``` 

Esta pieza de código es la responsable de cargar las librerías dinámicas en tiempo de ejecucción que un programa necesita para funcionar correctamente. En el caso de Linux, el responsable habitual es [ld-linux](http://man7.org/linux/man-pages/man8/ld.so.8.html) para los binarios en formato ELF. En nuestro caso, se encuentra en la ruta */lib64/ld-linux-x86-64.so.2*. 

Esta ruta queda embebida en el ejecutable como se ha visto al leer el fichero con *readelf*. Aunque esta ruta suele ser un estándar, uno de los problemas que puede suceder si nuestro sistema destino por algún motivo no lo sigue, es que no sea posible encontrar en él, el enlazador dinámico y nuestra aplicación no pueda ni ejecutarse.

Otro punto de contaminación que se puede dar es a la hora de cargar los entornos de ejecucción del lenguaje C. Estos ficheros, generados por el compilador, contienen [el código previo necesario para la inicialización de los programas](https://wiki.osdev.org/Creating_a_C_Library) antes de ejecutar el método *main*. 

```bash
$ grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log
/usr/lib/gcc/x86_64-redhat-linux/8/../../../../lib64/crt1.o succeeded
/usr/lib/gcc/x86_64-redhat-linux/8/../../../../lib64/crti.o succeeded
/usr/lib/gcc/x86_64-redhat-linux/8/../../../../lib64/crtn.o succeeded
``` 

Un runtime incorrecto a la hora de ser enlazado puede provocar problemas también en el momento de la ejecucción de nuestra aplicación. 

Lo mismo con la búsqueda de las cabeceras, las librerías o el *dynamic linker*

```bash
$ grep -B4 '^ /usr/include' dummy.log
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-redhat-linux/8/include
 /usr/local/include
 /usr/include

$ grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'
SEARCH_DIR("=/usr/x86_64-redhat-linux/lib64")
SEARCH_DIR("=/usr/lib64")
SEARCH_DIR("=/usr/local/lib64")
SEARCH_DIR("=/lib64")
SEARCH_DIR("=/usr/x86_64-redhat-linux/lib")
SEARCH_DIR("=/usr/local/lib")
SEARCH_DIR("=/lib")
SEARCH_DIR("=/usr/lib");

$ grep "/lib.*/libc.so.6 " dummy.log
attempt to open /lib64/libc.so.6 succeeded

$ grep found dummy.log
found ld-linux-x86-64.so.2 at /lib64/ld-linux-x86-64.so.2
```

Por todos estos puntos, es necesario realizar una compilación cruzada inicial de los cuatro elementos que comentábamos anteriormente. Una vez que se obtiene ese sistema inicial del host, ya es posible realizar la segunda compilación de todas las herramientas para tener un conjunto de herramientas auto-contenido e independiente del sistema anfitrión. 

Si tenéis interés en conocer más detalles, os recomiendo esta sesión de Thomas Petazzoni en la "Embedded Linux Conference" de 2016.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/Pbt330zuNPc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

*Photo by form [PxHere](https://pxhere.com/en/photo/1062912)*