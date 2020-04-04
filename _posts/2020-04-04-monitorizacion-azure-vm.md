---
layout: post
title: "Opciones de monitorización de máquinas virtuales en Azure"
author: "José Ángel Fernández"
categories: blog
tags: [azure, vm, monitorización]
image: monitoring.jfif
---

Esta semana me preguntaron por las opciones existentes en Azure para monitorizar los diferentes aspectos que estén relacionados con una máquina virtual. La verdad, pensaba que era una respuesta fácil y sencilla, seguro que vosotros también; sin embargo, a la hora de ponerme a buscar los detalles me llevé una sorpresa. ¿Cuántas opciones pensáis que pude encontrar? ¡Yo encontré catorce! Sí, catorce.

Este es el listado de los diferentes servicios disponibles que van de los más genéricos a los más específicos.

* [Activity Logs](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/view-activity-logs#azure-portal): el más genérico de todos, disponible para cualquier recurso de Azure. En nuestro caso específico, permite monitorizar todas las operaciones realizadas con la máquina virtual desde el punto de vista de las APIs de Azure: encender, apagar, reiniciar, añadir discos, etc.
* [Metrics](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/metrics-supported#microsoftcomputevirtualmachines): nos permite visualizar las métricas de rendimiento de la máquina desde el punto de vista del hipervisor. Podremos consultar datos sobre el uso de la CPU, el consumo de memoria RAM o las operaciones sobre el disco entre otros.
* [Boot Diagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/troubleshooting/boot-diagnostics): si algo falla en el arranque, tener activado los logs de diagnóstico puede ayudarnos a entender qué ha sucedido. Veremos la información del inicio de la máquina virtual así como una captura de pantalla de cómo lo ve el hipervisor.
* [Diagnostic settings](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/vminsights-enable-overview): nos permite extender las capacidades de captura de información básica de “Metrics” a través de un agente dentro de la máquina virtual. Podemos recoger logs o contadores de rendimiento por ejemplo.
* [Performance Diagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/troubleshooting/performance-diagnostics): si detectamos problemas de rendimiento en nuestra máquina virtual, esta característica está enfocada a hacer troubleshooting permitiendo descubrir problemas alto uso de disco, lentitud en la VM o fallos de configuración entre otros.
* [Azure Monitor for VMs](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/vminsights-overview): si queremos conocer aún más lo que sucede dentro de la máquina virtual y cómo ésta se relaciona con su entorno. La extensión de Azure Monitor para máquinas virtuales será de mucha utilidad, especialmente su opción de representación de dependencias.
* [Network Watcher Connection Monitor](https://docs.microsoft.com/en-us/azure/network-watcher/connection-monitor): monitoriza la conexión de red entre diferentes máquinas. Por ejemplo, de un servidor de aplicaciones a uno de base de datos. Especialmente útil para entornos críticos donde cambios en la latencia o un fallo de la red pueda requerir una acción inmediata.
* [Update Management](https://docs.microsoft.com/en-us/azure/automation/automation-tutorial-update-management): mantiene al día el control de las actualizaciones disponibles para el sistema operativo de nuestra máquina virtual. El objetivo es proteger la máquina virtual de despistes a la hora de aplicar los parches necesarios de seguridad.
* [Inventory](https://docs.microsoft.com/en-us/azure/automation/automation-vm-inventory): una forma sencilla de tener disponible un inventario del software que se encuentra instalado dentro de las máquinas virtuales.
* [Change tracking](https://docs.microsoft.com/en-us/azure/automation/change-tracking): gestiona los posibles cambios que se produzcan dentro de las máquinas virtuales como instalación de nuevo software, cambios en variables de registros o ficheros y estado de servicios o demonios. Útil para detectar posible intrusiones de seguridad o el origen de fallos.
* [Advisor recommendations](https://docs.microsoft.com/en-us/azure/advisor/advisor-overview): casi todos los clientes tienen alguna máquina sobredimensionada de recursos, no tiene configurado los elementos necesarios para una alta disponibilidad o podría reducir su factura aplicando instancias reservadas. Azure Advisor te ayuda a través de sus recomendaciones a ello entre otras cosas.
* [Configuration management](https://docs.microsoft.com/en-us/azure/automation/automation-dsc-overview): permite garantizar que los servidores mantienen la configuración deseada a partir de módulos de PowerShell DSC. Una alternativa a lo que pueden proporcionar herramientas como Chef, Puppet o Ansible.
* [Azure Service Health](https://docs.microsoft.com/en-us/azure/service-health/service-health-overview): monitorización general de posibles incidencias que afecten a la plataforma o de mantenimientos programados que pueden impactar en el servicio para tenerlo planificado el plan de acción si fuera necesario.
* [Azure Resource Health](https://docs.microsoft.com/en-us/azure/service-health/resource-health-overview): si está ocurriendo una incidencia, es la forma más rápida de saber si una incidencia a nivel de la plataforma puede afectar a tus recursos concretos. Por ejemplo, una incidencia en un cluster de almacenamiento puede estar afectando a los discos de tu máquina virtual.

Si necesitáis tener controlado todo lo que sucede con vuestra máquina virtual, recordad echar un ojo a todas las opciones disponibles para que no se os quede ninguna fuera del radar.


*Photo by form [PxHere](https://pxhere.com/en/photo/893743)*