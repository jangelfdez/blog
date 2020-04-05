---
layout: post
title: "Ejecutando múltiples veces CustomScriptExtension en una máquina Windows en Azure"
author: "José Ángel Fernández"
categories: blog
tags: [azure, extensions]
image: sourcecode.jfif
---

Es posible que en alguna ocasión hayáis necesitado ejecutar más de un script de configuración en una máquina virtual con Windows Server 2012 R2 en Azure en diferentes momentos de su aprovisionamiento. Si habéis intentado realizarlo desplegando la extensión Custom Script lo más probable es que recordéis un mensaje como el siguiente:

```bash
me@Azure:~$ az group deployment create --name deploy --resource-group jangelfdez-blog --template-file /home/me/clouddrive/main.simplevm.json --verbose
[...]
Failed: secondInstallation (Microsoft.Compute/virtualMachines/extensions)
Failed: deploy (Microsoft.Resources/deployments)
Succeeded: firstInstallation (Microsoft.Compute/virtualMachines/extensions)
Deployment failed. {
  "error": {
    "code": "BadRequest",
    "message": "Multiple VMExtensions per handler not supported for OS type 'Windows'. VMExtension 'secondInstallation' with handler 'Microsoft.Compute.CustomScriptExtension' already added or specified in input."
  }
}
```

Esta salida se corresponde con la siguiente plantilla de ARM. Está basada en [la disponible en el repositorio de Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-simple-windows/azuredeploy.json) y únicamente le he añadido la parte de las extensiones. En ambas intento ejecutar un comando en línea de PowerShell que escriba algo a la salida estándar.

```json
[
{
  "type":"Microsoft.Compute/virtualMachines/extensions",
  "name":"[concat(variables('vmName'), '/firstInstallation')]",
  "apiVersion":"2016-03-30",
  "location":"[resourceGroup().location]",
  "dependsOn":[
  "[variables('vmName')]"
  ],
  "properties":{
    "publisher":"Microsoft.Compute",
    "type":"CustomScriptExtension",
    "typeHandlerVersion":"1.8",
    "autoUpgradeMinorVersion":true,
    "settings":{
      "fileUris":{
      },
      "commandToExecute":"powershell.exe -Command 'Write-Output First"
      },
      "protectedSettings":{
      }
  }
},
{
  "type":"Microsoft.Compute/virtualMachines/extensions",
  "name":"[concat(variables('vmName'), '/secondInstallation')]",
  "apiVersion":"2016-03-30",
  "location":"[resourceGroup().location]",
  "dependsOn":[
    "[variables('vmName')]"
  ],
  "properties":{
    "publisher":"Microsoft.Compute",
    "type":"CustomScriptExtension",
    "typeHandlerVersion":"1.8",
    "autoUpgradeMinorVersion":true,
    "settings":{
      "fileUris":{
      },
      "commandToExecute":"powershell.exe -Command 'Write-Output Second'"
      },
      "protectedSettings":{
    }
  }
}
]
``` 

Si revisamos el log disponible en C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension\1.9 veremos que se interpreta correctamente nuestro comando de la primera extensión. Sin embargo, como indicaba el log de despliegue, de la segunda no hay rastro.

```bash
[INFO] Waiting for all async file download tasks to complete...
[INFO] Files downloaded. Asynchronously executing command: 'powershell.exe -Command 'Write-Output First''
[INFO] Command execution task started. Awaiting completion...
[INFO] Command execution finished. Command exited with code: 0
```

Esto se debe a que no es posible instalar esta extensión dos veces sobre una máquina virtual con Windows Server 2012 R2. Si probáis con Windows Server 2016 observaréis que no sucede lo mismo, en esta versión sí que permite instalar dos veces una extensión.

Antes de probar con otras alternativas como dejar una tarea de Windows programada o buscar algún sistema de gestión de la configuración externo como PowerShell DSC, Puppet o Chef estuve rebuscando en la documentación y encontré lo siguiente.

Es posible ejecutar la instalación de la misma extensión varias veces con diferentes comandos empleando el parámetro timestamp. Sin embargo, no es algo evidente. [La documentación de la extensión para Windows no está actualizada](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/extensions-customscript) a la última versión; tienes que descubrirlo en [la documentación para la extensión sobre Linux](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/extensions-customscript) o revisando [el repositorio de GitHub](https://github.com/Azure/custom-script-extension-linux).

Para ello será necesario definir la primera ejecución de la extensión en la plantilla inicial y ,posteriormente, definir la segunda ejecución en otra nueva plantilla en la que se debe añadir la variable de timestamp en el apartado de settings. La propiedad admite un valor numérico en formato entero que no tiene por qué corresponderse con el momento en el tiempo actual. Finalmente será necesario actualizar el comando que se desea ejecutar.

Al tener una segunda plantilla que desplegar esto se puede hacer de dos formas: hacer dos despliegues o emplear los despliegues anidados. Asumiremos la primera, [la segunda podéis encontrar más información en la documentación](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-linked-templates).

Nuestra segunda plantilla contendrá lo siguiente únicamente:

 
```json
[
{
  "type":"Microsoft.Compute/virtualMachines/extensions",
  "name":"[concat(variables('vmName'), '/firstInstallation')]",
  "apiVersion":"2016-03-30",
  "location":"[resourceGroup().location]",
  "dependsOn":[
    "[variables('vmName')]"
  ],
  "properties":{
    "publisher":"Microsoft.Compute",
    "type":"CustomScriptExtension",
    "typeHandlerVersion":"1.8",
    "autoUpgradeMinorVersion":true,
    "settings":{
      "fileUris":{
      },
      "commandToExecute":"powershell.exe -Command 'Write-Output Second'",
      "timestamp": 10000
      },
      "protectedSettings":{
      }
    }
}
]
```
Ya solo queda desplegarla y volver a mirar el log de la extensión; en él veremos que se han añadido nuevas filas que contienen la recepción de nuestro nuevo comando modificado.

```bash
[INFO] Waiting for all async file download tasks to complete...
[INFO] Files downloaded. Asynchronously executing command: 'powershell.exe -Command 'Write-Output Second''
[INFO] Command execution task started. Awaiting completion...
[INFO] Command execution finished. Command exited with code: 0
```

Esta opción simplifica el proceso pudiendo trabajar directamente con plantillas de ARM y evitando incluir así otra alternativa externa.

*Photo by form [PxHere](https://pxhere.com/en/photo/593221)*