---
layout: post
title: "El coste del ancho de banda en Azure, un pequeño desconocido"
author: "José Ángel Fernández"
categories: blog
tags: [azure, bandwidth, costes]
image: servers.jfif
---

El otro día, en una conversación interna, saltaron unas cuántas dudas de qué tráfico se cobraba o no en Azure según el origen, el destino y el tipo de conectividad de la que se disponía. Inicialmente pensé que era algo que tenía bastante claro pero tras ver el resumen que hizo mi compañero José Guardia ([@msjosegm](https://twitter.com/msjosegm)) creo que tiene mucho más detrás de lo que inicialmente pensaba.

A continuación, tenéis un listado de todos los puntos que tenéis que tener en cuenta cuando tengáis que realizar una estimación de los costes de ancho de banda en Azure:

* Tráfico que sea de ingreso a Azure desde Internet, una conexión VPN, una conexión ExpressRoute u otro servicio de Azure: gratuito.
* Tráfico que sea entre recursos conectados dentro de la misma red virtual: gratuito
* Tráfico que fluya entre un VNet Peering ya sea regional o entre diferentes regiones: [tiene coste tanto de salida como de entrada](https://azure.microsoft.com/en-us/pricing/details/virtual-network/).
* Tráfico que sea saliente de la red virtual hacia Internet: [tiene coste de salida](https://azure.microsoft.com/en-us/pricing/details/bandwidth/).
* Tráfico que sea saliente de la red virtual hacia una IP pública de la misma región de Azure: gratuito.
* Tráfico que sea saliente de la red virtual hacia una IP pública de otra región de Azure: [tiene coste de salida](https://azure.microsoft.com/en-us/pricing/details/bandwidth/).
* Tráfico que sea saliente de la red virtual a través de una conexión VPN: [tiene coste de salida](https://azure.microsoft.com/en-us/pricing/details/bandwidth/)..
* Tráfico que sea saliente de la red virtual a través de una conexión de ExpressRoute en su categoría metered: [tiene coste de salida](https://azure.microsoft.com/en-us/pricing/details/expressroute/).
* Tráfico que sea saliente de la red virtual a través de una conexión de ExpressRoute en su categoría unmetered: gratuito.
* Tráfico entre servicios de Azure dentro de la misma región: gratuito.
* Tráfico entre servicios de Azure en diferentes regiones: [tiene coste de salida](https://azure.microsoft.com/en-us/pricing/details/bandwidth/).
* Tráfico entre servicios de Azure desplegados en Availability Zone en una red virtual: dentro de la misma AZ es gratuito pero entre AZ [tiene coste de entrada y salida](https://azure.microsoft.com/en-us/pricing/details/bandwidth/).

Como veis, no tan sencillo como inicialmente parecía. Si seguía la máxima de todo lo entrante es gratuito y solo se factura lo saliente, recordad las excepciones a la regla que en este caso son varias.

*Photo by form [PxHere](https://pxhere.com/en/photo/1136169)*
