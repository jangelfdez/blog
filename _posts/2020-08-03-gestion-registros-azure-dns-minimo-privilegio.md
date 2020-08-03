---
layout: post
title: "Gestión de registros A en Azure DNS con el mínimo privilegio posible"
author: "José Ángel Fernández"
categories: blog
tags: [azure, dns]
image: lock.jfif
---

*Azure Role Based Access Control* proporciona un gran número de roles por defecto para gestionar nuestros recursos en Azure. Sin embargo, en algunas ocasiones es posible que éstos sean demasiado amplios para lo que nos interesa y necesesitemos limitar su alcance. Para ello, será necesario que hagamos uso de los roles personalizados. 

En esta situación me he encontrado hoy cuando únicamente se deseaba permitir la gestión de los registros de tipo A dentro de una zona de Azure DNS y nada más. Como estoy seguro que en el futuro lo volveré a necesitar, qué mejor que dejarlo registrado aquí:

``` json
  "permissions": [
            {
                "actions": [
                    "Microsoft.Network/privateDnsZones/read",
                    "Microsoft.Network/privateDnsZones/write",
                    "Microsoft.Network/privateDnsZones/A/read",
                    "Microsoft.Network/privateDnsZones/A/write"
                ],
                "notActions": [
                    "Microsoft.Network/privateDnsZones/A/delete"
                ],
                "dataActions": [],
                "notDataActions": []
            }
        ]

```

El proceso de saber qué acciones son las que necesitamos activar o cuáles bloquear, no era una tarea fácil. Sin embargo, si aún no lo habéis probado os recomiendo que le echés un ojo al [editor de roles personalizados disponible en el portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles-portal#step-3-basics). 

Si aún así preferís la experiencia dura, ya no es necesario consultar a través del CLI las operaciones que un proveedor de recursos tiene habilitadas, ahora se encuentran [incluídas en la documentación](https://docs.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations). ¡Ojo! Creo que probablemente sea la página más larga de las que he visto en la documentación oficial :) 

*Photo by form [PxHere](https://pxhere.com/en/photo/816636)*

