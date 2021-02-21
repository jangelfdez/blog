---
layout: post
title: "Opciones de salida a Internet en Azure: ¿Load Balancer o NAT Gateway?"
author: "José Ángel Fernández"
categories: blog
tags: [azure, networking]
image: networkcable.jfif
---

Azure proporciona por defecto conectividad de salida a Internet a cualquier máquina virtual desplegada dentro de una subred. Esto simplifica la configuración inicial cuando necesitamos que nuestras máquinas virtuales accedan a contenido fuera de Azure. Si no necesitamos nada más, con el comportamiento por defecto sería más que suficiente. Sin embargo, ¿qué sucede cuándo necesitamos tener un mayor control de nuestros flujos de salida? Por ejemplo, ¿qué podemos hacer para asegurarnos que siempre sale por la misma IP? ¿cómo podemos controlar mejor el agotamiento de los puertos para SNAT?

En estos escenarios tenemos dos alternativas, desplegar un [balanceador de carga](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview) o un [NAT Gateway](https://docs.microsoft.com/en-us/azure/virtual-network/nat-overview) que nos proporcione  capacidades avanzadas. Sin embargo, ¿cuál de los dos debería escoger? 

Si este es tu caso y no sabes qué opción es la más recomendable, la siguiente tabla resume las principales diferencias entre ambos:

|                       | Load Balancer | NAT Gateway |
|---|---|---|
| **Configuración** | Detallada. Compleja. | Sencilla. |
| **Alcance**       | Recursos en un pool de balanceo. | Todos los recursos de la subred. |
| **Timeout**       | 4 minutos por defecto. <br> Configurable (4-30min). <br> Opción de TCP Reset en estado Idle. |  4 minutos por defecto. <br> Configurable (4-120min). <br> TCP Reset con paquetes inesperados. | 
| **Flujos de tráfico**     | Únicamente de salida.  <br> Compatibilidad con LB o PIP   | Entrada y salida |
| **Frontend**              | Public IP, Public IP Prefix. | Public IP, Public IP Prefix. | 
| **Zonas de disponibilidad** | Opcional. <br> Redudante de zona, zonal. <br> Stateless. | Opcional. <br> Zonal. <br> Stateful. |
| **Ancho de banda** | Sin limitaciones tráfico de salida. | 50Gbps. |
| **Nº IPs** | Nº máquinas x SKU. | Hasta 16 direcciones IP. |
| **Nº Flujos** | Límite de la máquina virtual. | Hasta 1M de flujos concurrentes. |
| **Coste** | Instancia, reglas de balanceo y datos procesados. | Instancia y datos procesados. |

Por regla general, si nuestras instancias van a publicar algún tipo de servicio hacia Internet la opción recomendada es utilizar un balanceador de carga; por el contrario, si únicamente son instancias internas, la solución sería un NAT Gateway. El principal problema es que los costes de la segunda solución puede ser mayores que la primera al tener un mayor coste por hora y por tráfico procesado. 

Es por eso necesario comparar las características de ambas soluciones y escoger aquella que mejor se ajusta a nuestro escenario, no solo en funcionalidad si no también en precio.

*Photo by form [PxHere](https://pxhere.com/es/photo/1616396)*