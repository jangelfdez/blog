---
layout: post
title: "Automatiza el despliegue de tu servidor CycleCloud con Bicep"
author: "José Ángel Fernández"
categories: blog
tags: [azure, compute, hpc]
image: circuits.jfif
---

[CycleCloud](https://learn.microsoft.com/en-us/azure/cyclecloud/overview?view=cyclecloud-8) proporciona la forma más sencilla de aprovisionar en Azure clústeres de computación de alto rendimiento *(High Performance Computing, HPC)* basados en los planificadores más utilizados en el mercado como por ejemplo: Slurm, PBS o LSF. 

De forma general, CycleCloud se despliega como una máquina virtual a partir de la [imagen base disponible en el Marketplace de Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/azurecyclecloud.azure-cyclecloud?tab=overview). Tras ello, para completar la instalación, es necesario seguir un asistente web de tres sencillos pasos.

En la mayor parte de los casos, este despliegue manual es suficiente ya que es un proceso que se realiza una única vez. Sin embargo, en algunos casos es posible que nos interese automatizar el proceso de instalación sin necesidad de acceder a la interfaz gráfica. Especialmente aquellos clientes que emplean la estrategia de *Infraestructura como Código (IaC)* para desplegar múltiples entornos u entornos efímeros.

Este artículo se basa en los scripts de configuración de Ansible empleados por [Azure HPC On-Demand Platform](https://github.com/Azure/az-hop) y permite desplegar tu servidor de CycleCloud de forma automática. En primer lugar, se explica paso a paso la configuración necesaria para a continuación proporcionar una implementación mínima en Bicep lista para ser desplegada.

## Información necesaria para la configuración inicial

Para tener una instalación funcional de CycleCloud es necesario proporcionar los siguientes datos:

- Nombre del usuario y contraseña de la cuenta de usuario inicial que tendrá el rol administrador del servidor.
- Clave SSH pública asociada a dicho usuario que se empleará para configurar el acceso remoto a los nodos de los clústeres.
- Datos de la subscripción de Azure que se utilizará para desplegar los recursos asociados a los clústeres.

## Configurar la cuenta de administrador

CycleCloud permite modificar la configuración del servidor de forma dinámica a través de ficheros JSON. Estos ficheros, al ser colocados en la carpeta */data/config* dentro del directorio de instalación de CycleCloud son automáticamente importados. Si la configuración se ha importado correctamente veremos que el nombre del fichero se habrá modificado con la extensión *.json.imported*. Este funcionalidad permite  proporcionar los datos necesarios para automatizar la instalación.

En primer lugar, para configurar la cuenta de administador crearemos un nuevo fichero que llamaremos account_data.json. El nombre es indiferente por lo que es posible escoger cualquier otro nombre. El fichero será necesario crearlo en */opt/cycle_server/config/data/*. 

El contenido será el siguiente:

```bash
[
  {
    "AdType": "Application.Setting",
    "Name": "cycleserver.installation.initial_user",
    "Value": <YourUserName>
  },
  {
    "AdType": "AuthenticatedUser",
    "Name": <YourUserName>
    "RawPassword": <YourUserPassword>,
    "Superuser": true
  },
  {
    "AdType": "Credential",
    "CredentialType": "PublicKey",
    "Name": "<YouUserName>/public"
    "PublicKey": <YourPublicKeyInformation>,
  },
  {
    "AdType": "Application.Setting",
    "Name": "cycleserver.installation.complete",
    "Value": true
  }
]
```

En primer lugar se esvoge el nombre de usuario administrador de CycleCloud, su contraseña y la clave de acceso pública asociada. La última propiedad indica a CycleCloud que no muestre la pantalla de configuración inicial ya que la configuración se ha realizado correctamente.

Una vez guardado el fichero, comprobaremos que su extensión es modificada a *.imported* indicando que los cambios se han aplicado.

## Configurar la subscripción de Azure

Tras configurar el administrador de CycleCloud, el siguiente paso es proporcionar los datos de la suscripción de Azure que usaremos para desplegar los clústeres. Crearemos un fichero JSON que llamaremos azure_data.json. Esta vez, sin embargo, no lo haremos en dicho directorio sino en nuestra $HOME. 

El contenido del fichero será el siguiente:

```bash
{
    "Environment": "public",
    "AzureRMUseManagedIdentity": true,
    "AzureResourceGroup": <ResourceGroupWhereForCycleCloudResources>,
    "AzureRMApplicationId": " ",
    "AzureRMApplicationSecret": " ",
    "AzureRMSubscriptionId": <AzureSubscriptionId>,
    "AzureRMTenantId": <AzureTenantId>,
    "DefaultAccount": true,
    "Location": <DefaultLocation>,
    "Name": "azure",
    "Provider": "azure",
    "ProviderId": "fd6abe95-c55e-44c8-9085-68002a27c1bb",
    "RMStorageAccount": <ExistinStorageAccountName>
    "RMStorageContainer": <CycleCloudLockerContainerName>
  }

```

Es importante mencionar que CycleCloud soporta dos formas de autenticarse contra Azure:
- Utilizando un Service Principal (Application) y su contraseña (Secret)
- Utilizando una identidad gestionada por Azure.

La segunda opción es la recomendada ya que toda la gestión del ciclo de vida de la identidad y de los secretos es gestionada de forma automática por Azure. En este ejemplo utilizaremos esta opción. Si por algún motivo es necesario utilizar un *Service Principal*, únicamente tendrás que proporcionar sus datos en las propiedades *AzureRMApplicationId* y *AzureRMApplicationSecret*, y modificar a *false* la propiedad *AzureRMUseManagedIdentity*

No obstante, en cualquiera de las dos opciones es importante acordarse de asignar los permisos de RBAC dentro de nuestra suscripción para desplegar los recursos. En este ejemplo utilizaremos el rol de *Contributor* a nivel de la suscripción.

Tras asegurarnos que los permisos están correctamente configurados será necesario ejecutar el siguiente comando. 

```bash
/usr/local/bin/cyclecloud account create -f $HOME/azure_data.json
```

Tras ello, si accedemos al portal de CycleCloud con los datos del usuario administrador, podremos ver que funcionan y que en la opción de Configuración (1) > Suscripciones (2), la suscripción se ha configurado correctamente (3) y está recuperando la información necesaria (4).

[![](/assets/img/cyclecloudinstallation.png)](/assets/img/cyclecloudinstallation.png)

## Un ejemplo mínimo de automatización con Bicep

Todo el proceso anterior ha servido para entender cuáles son los pasos necesarios para automatizar la instalación sin hacer uso de la interfaz de usuario. Sin embargo, hemos seguido realizando de forma manual cada uno de los pasos. Esto son:

1. Crear la máquina virtual desde el MarketPlace con la imagen de referencia de CycleCloud
2. Configurar la máquina virtual para que haga uso de las identidades gestionadas
3. Asignar los permisos RBAC a la identidad gestionada para que tenga acceso a la suscripción de Azure
4. Generar el fichero de configuración del usuario administrador de CycleCloud en la carpeta */data/config*.
5. Importar la configuración de Azure con el comando *cyclecloud account*

Los tres primeros puntos es posible automatizarlos con una plantilla de Bicep. Los dos últimos podemos escribir nuestros propios scripts, o, como comentábamos al principio, aprovechar el trabajo existente de az-hop.

No queremos reinventar la rueda por lo que usaremos el fichero [configure.py](https://github.com/Azure/az-hop/blob/main/playbooks/roles/cyclecloud/files/configure.py) que incorpora además validaciones extras ante posibles fallos y la inicialización de la CLI de CycleCloud. Para nuestro caso particular, únicamente será necesario modificar la línea 230 para reemplazar:

```bash
--url=https://localhost/cyclecloud"
```
por 

```bash
--url=https://localhost/"
```

ya que la instalación por defecto es a nivel raíz del servidor.

Si quieres ver el código, está disponible en [jangelfdez/cyclecloud-bicep](https://github.com/jangelfdez/cyclecloud-bicep). Para desplegarlo en tu suscripción, únicamente necesitarás ejecutar el siguiente comando reemplazando los valores de los siguiente parámetros:

- **location**: región de Azure donde se realizará el despliegue.
- **resourceGroupname**: grupo de recursos donde se creará la máquina virtual de CycleCloud.
- **vnetName** y **subnetName**: datos de la red donde la máquina virtual se desplegará.
- **vnetResourceGroupName**: si vuestra red virtual está en otro grupo de recursos diferente, es necesario indicarlo con este parámetro. Si está en el mismo lo puedes omitir.
- **storageAccountName**: nombre de la cuenta de almacenamiento que CycleCloud utilizará como *locker*.
- **tenantId**: identificador de vuestro tenant **.onmicrosoft.com*.
- **adminUsername**, **adminPassword**, **publicKey**: los datos de acceso del usuario administrador de CycleCloud que coincidirán con los de la VM en este caso.

```bicep

az deployment sub create --location <location> --template-file .\main.bicep --parameters resourceGroupName=<rgName> vnetName=<vnetName> subnetName=<subnetName> vnetResourceGroupName=<netRgName> storageAccountName=<saname> tenantId=<tenantId> adminUsername=<username> publicKey='<publicKey>'

```

Si quieres entender lo que sucede, la estructura es la siguiente:

[![](/assets/img/cycleclouddeployment.png)](/assets/img/cycleclouddeployment.png)

El fichero *main.bicep* orquesta el resto del despliegue. Esto es así ya que necesitamos desplegar recursos tanto a nivel de suscripción como a nivel de grupo de recursos. Bicep únicamente lo permite configurando el *targetscope* a nivel de suscripción del fichero principal y luego usando módulos con  *scopes* personalizados.

En primer lugar, se despliegua la máquina virtual junto con sus discos, tarjeta de red e IP a nivel del grupo de recursos pasado por parámetro. Una vez que ha terminado, se asigna a nivel de subscripción el rol de *Contributor* a la identidad gestionada asociada a la VM. Finalmente, se termina la instalación desplegando de nuevo a nivel del grupo de recursos de la extensión *CustomScript* que ejecuta el script de Python para inicializar CycleCloud.

Si en lugar de asignar los permisos a nivel de suscripción únicamente fuera necesario asignarlo a nivel del grupo de recursos, sería posible simplificar el despliegue en un único fichero con todos los recursos en él sin necesidad de usar múltiples módulos y *scopes*.

Si has llegado hasta aquí, ¡felicidades!, ya conoces los principios básicos de cómo automatizar el despliegue de CycleCloud para integrarlo en tus propios scripts.