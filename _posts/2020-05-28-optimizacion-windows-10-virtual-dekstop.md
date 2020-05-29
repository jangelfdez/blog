---
layout: post
title: "Optimización de imágenes de Windows 10 para Virtual Desktop"
author: "José Ángel Fernández"
categories: blog
tags: [azure, wvd]
image: keyboardblack.jfif
---


Como ya comentaba en [un artículo previo](/blog/logs-actividad-windows-virtual-desktop.html), el uso de *Windows Virtual Desktop (WVD) como herramienta para desplegar entornos de trabajo remoto en estos tiempos de pandemia se ha incrementado considerablemente.* 

Entornos que inicialmente parecían desplegarse para unos pocos días o semanas, con el paso del tiempo han pasado a ser definitivos para cubrir las necesidades de trabajo remoto de los próximos meses en los que continúe este proceso de desescalada y vuelta a la nueva normalidad. Es por ello que una optimización de las imágenes que están desplegadas, especialmente las de Windows 10, nos puede ayudar a sacar más partido de los recursos en uso.

Uno de los puntos de referencia, que creo que no es del todo conocido, es [la guía de optimización de Windows 10 en entornos de infraestructura de escritorios virtuales](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/rds_vdi-recommendations-1909). Un compendio de optimizaciones de configuración enfocadas a minimizar, por un lado, la necesidad de redibujado de las aplicaciones, la ejecucción de actividades de fondo que no aportan un valor extra en este tipo de entornos y otros procesos del sistema al mínimo posible; y por otro, reducir el tamaño de nuestras imágenes. El objetivo de todo ello es lograr una imagen maestra con el menor tamaño y uso de recursos posible. Algo que se notará fundamentalmente en los costes de entornos a gran escala. 

Es posible seguir la guía paso a paso analizando las recomendaciones que se realizan o acceder [al repositorio de GitHub del equipo de *The VDI Guys*](https://github.com/TheVDIGuys/Windows_10_VDI_Optimize/tree/2004) donde es posible encontrar los *scripts* para facilitar la automatización de esta configuración. En él se encuentran los detalles para cada una de las versiones disponibles de Windows 10, siendo la 2004 la última y la que contiene mejoras para los escenarios basados en la imagen multisesión. 

Si quieres probarlo, los pasos son los siguientes. Es importante que tengas en cuenta que no es un recurso oficial de Microsoft que se encuentre soportado o respaldado por el equipo de producto de Windows o de Windows Virtual Desktop. Por lo tanto, a la hora de ejecutarlo no existen garantías, ¡úsalos con cabeza!.

1. Clona el repositorio o descárgalo en formato comprimido a tu sistema Windows 10.
    * La carpeta *2004* contiene las políticas locales en formato JSON. 
    * El directorio *LGPO* la herramienta para aplicar las políticas anteriores a nuestro sistema operativo. 
    * Y en la raíz está el *script* *Win10_VDI_Optimize.ps1* para la ejecucción. 
2. Inicia sesión en la máquina que se va a emplear como imagen maestra.
3. Abre una consola de PowerShell con permisos de administrador
4. Ejecuta los siguientes comandos:
    *	Set-ExecutionPolicy -ExecutionPolicy Bypass
    *	Accede al directorio donde se encuentran los ficheros.
    *	.\Win10_VDI_Optimize.ps1 -WindowsVersion 2004 -Verbose
    *	Set-ExecutionPolicy -ExecutionPolicy default
5. ¡Listo!

Cualquier sugerencia, problema o pregunta, no dudéis en abrir un incidente en GitHub al equipo. Están abiertos a ello. Todo esto lo descubrí gracias [a mi compañero Robert Smith](https://www.linkedin.com/in/robert-smith-9143a417b/) a través de las listas internas de Microsoft.

*Photo by form [PxHere](https://pxhere.com/en/photo/935315)*

