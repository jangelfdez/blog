---
layout: post
title: "Evitar el throttling listando ficheros de Azure File Shares desde la CLI"
author: "José Ángel Fernández"
categories: blog
tags: [azure, storage, files]
image: storagefiles.jfif
---

A la hora de trabajar con Azure Files es posible que tengamos la necesidad de listar los ficheros que se encuentran alojados dentro del servicio. Desde la CLI podemos realizarlo de forma sencilla a través de un comando similar al siguiente:

```bash
az storage file list --share-name MyShare --account-name MyStorageAccountName 
```

Si es algo puntual, es posible que no te encuentres ningún problema; sin embargo, si lo tenéis integrado en algún script que hace varias consultas en un corto periodo de tiempo os podéis encontrar con la limitación de ARM de realizar un [máximo de operaciones de tipo *List* de 100 cada 5 minutos](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits#storage-resource-provider-limits).

¿Por qué salta esta limitación si únicamente tendría que estar leyendo datos de la API de Azure a través del método GET correspondiente? Como siempre, todo tiene su motivo.

La razón es que si no se incluyen los detalles de autenticación de la cuenta de almacenamiento a la que estamos accediendo, el comando automáticamente lanza una operación para listar sus claves de acceso y autenticar la petición.

Si queremos evitar este problema, únicamente será necesario utilizar alguna de las opciones alternativas disponibles con el mismo comando para proporcionarle esos datos:

```bash
# Empleando la clave de la cuenta
az storage file list --share-name MyShare --account-name MyStorageAccountName  –account-key [your-key]

# Empleando la cadena de conexión completa
az storage file list --share-name MyShare –connection-string [your-connection-string]

# Empleando un token SAS específico
az storage file list --share-name MyShare --account-name MyStorageAccountName f –sas-token [your-sas]
```

De esta manera, la CLI no necesitará ejecutar esas consultas extras y se autenticará directamente evitando así la limitación de la API.

*Photo by form [PxHere](https://pxhere.com/en/photo/536212)*