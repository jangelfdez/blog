---
layout: post
title: "Consulta de las métricas de uso en Windows Virtual Desktop"
author: "José Ángel Fernández"
categories: blog
tags: [azure, wvd, monitor]
image: keyboardblack.jfif
---


El uso de *Windows Virtual Desktop (WVD)* como herramienta para desplegar entornos de trabajo remoto en estos tiempos de pandemia se ha incrementado considerablemente. De forma global, [se había visto un crecimiento de un 300% en las últimas semanas](https://azure.microsoft.com/en-us/blog/update-2-on-microsoft-cloud-services-continuity/). A día de hoy, es la carga de trabajo que nos encontramos prácticamente en todos nuestros clientes independiente del sector industrial al que pertenezcan. 

Una vez que hemos trabajado para tener el entorno inicial en marcha y los usuarios están haciendo uso del servicio, surge la necesidad de conocer mejor qué es lo que está sucediendo y empezar a sacar estádisticas y realizar análisis del mismo. A día de hoy, WVD no tiene una integración directa con el portal de Azure que nos permita verlo; sin embargo, eso no quiere decir que no podamos tener esos datos.

El primer paso necesario es activar el [envío de los logs de actividad a Azure Monitor Logs](https://docs.microsoft.com/en-us/azure/virtual-desktop/diagnostics-log-analytics). Esto nos permitirá monitorizar las siguientes actividades en nuestro *tenant*:

* **Subscription Feed**: cualquier interacción de los usuarios a través de las aplicaciones soportadas para acceder al servicio en las que intente obtener su *feed* para listar las aplicaciones o escritorios a los que tiene acceso. 
* **Connection**: cualquier intento de conexión a un escritorio o a una aplicación virtualizada a través de cualquiera de las aplicaciones soportadas. 
* **Management**: actividades relacionadas con las tareas de administración del sistema como crear nuevos *pools* de máquinas, la asignación de usuarios a grupos de aplicaciones o la asignación de nuevos roles. 

Los eventos quedarán registrados en la tabla ***WVDActivityV1_CL*** dentro de nuestro área de trabajo de Azure Monitor Logs. Junto a ella, también se crearán las tabla ***WVDCheckpointV1_CL*** y ***WVDErrorV1_CL*** que contienen eventos internos y los errores relacionados con el sistema respectivamente.

Por lo tanto, si necesitas recuperar información histórica de uso para saber quién se ha conectado, cuántas veces lo ha hecho, cuánto tiempo han estado conectados o qué aplicaciones o nodos han utilizado, este es el camino. La información está disponible en bruto por lo que será necesario que [construyas las consultas de Kusto](https://docs.microsoft.com/en-us/azure/data-explorer/write-queries) específicas para extraer los datos que te interesen. En la propia documentación vienen [algunos ejemplos](https://docs.microsoft.com/en-us/azure/virtual-desktop/diagnostics-log-analytics#example-queries) para empezar. 

Os recomendaría empezar filtrando por las actividades cuyo *Type_s* sea igual a *Connection*; si queréis filtrar por tipo de recurso utilizado, emplear el parámetro *ResourceType* igual a *DESKTOP*, para sesiones de escritorio remoto, o a *RAIL*, para sesiones de aplicaciones virtualizadas. En la siguiente tabla os dejo un ejemplo de cada uno de estos dos eventos para que os sirva de referencia para vuestras consultas: 

#### Registro inicio de sesión en un escritorio virtual

Párametro | Valor 
:-----:|:-----:|
SourceSystem|RestAPI
TimeGenerated [UTC]|2020-04-01T12:11:52.776Z
PredecessorConnectionId\_s|<>
ResourceAlias\_s|SessionDesktop
ResourceType|DESKTOP
SessionHostPoolName\_s|xxx-Pool-01
SessionHostName\_s|RDSH-Pool-0.aadds.xxx.es
SessionHostIPAddress\_s|13.80.xx.xx
AgentOSVersion\_s|10.0.18363
AgentOSDescription\_s|Windows 10 Enterprise for Virtual Desktops
AgentSxsStackVersion\_s|rdp-sxs200326004
ClientOS\_s|Windows 10 Chrome 80.0.3987.149
ClientVersion\_s|1.0.21.1
ClientType\_s|HTML
ClientIPAddress\_s|<
Id\_g|a68a5265-3e2b-xxxx-xxxx-aa06e9510000
Type\_s|Connection
StartTime\_t [UTC]|2020-04-01T11:30:03.59Z
EndTime\_t [UTC]|2020-04-01T12:11:22.502Z
UserName\_s| xxx@xxx.es
Outcome\_s|Success
Status\_s|Completed|
TenantId\_s|xxxWVD
Type|WVDActivityV1\_CL

#### Registro inicio de sesión en una aplicación virtualizada

Párametro | Valor 
:-----:|:-----:
SourceSystem|RestAPI
TimeGenerated [UTC]|2020-03-30T16:57:19.957Z
PredecessorConnectionId\_s|<>
ResourceAlias\_s|powerpoint
ResourceType|RAIL
SessionHostPoolName\_s|xxx-Pool-01
SessionHostName\_s|RDSH-Pool-0.aadds.xxx.es
SessionHostIPAddress\_s|13.80.xx.xx
AgentOSVersion\_s|10.0.18363
AgentOSDescription\_s|Windows 10 Enterprise for Virtual Desktops
AgentSxsStackVersion\_s|rdp-sxs200326004
ClientOS\_s|Windows 10 Chrome 83.0.4086.0
ClientVersion\_s|1.0.21.1
ClientType\_s|HTML
ClientIPAddress\_s|<>
Id\_g|7f9b814c-8d9f-xxxx-xxxx-c7446f290000
Type\_s|Connection
StartTime\_t [UTC]|2020-03-30T16:50:39.229Z
EndTime\_t [UTC]|2020-03-30T16:54:47.111Z
UserName\_s|xxx@xxx.es
Outcome\_s|Failure
Status\_s|Completed
TenantId\_s|VirtualdomWVD
Type|WVDActivityV1\_CL

Si tenéis algunas consultas ya creada que sean útiles, no dudéis en compartirlas. De esta manera, será más fácil reutilizar el conocimiento entre todos.

*Photo by form [PxHere](https://pxhere.com/en/photo/935315)*

