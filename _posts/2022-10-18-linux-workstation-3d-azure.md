---
layout: post
title: "Despliega una workstation Linux para visualización 3D en Azure"
author: "José Ángel Fernández"
categories: blog
tags: [azure, compute, 3d, visualization]
image: sourcecode.jfif
---

Azure dispone de varias opciones a la hora de desplegar una máquina virtual con soporte de GPUs para aceleración gráfica en entornos de visualización remota. Desde la serie original NV, con las NVIDIA Tesla M60, hasta la quinta generación de con la serie NVadsA10 basda en las NVIDIA A10. Esta serie es laprimera que introduce el soporte al uso de GPUs particionadas con un mínimo de 1/6 de los recursos de la GPU en la versión Standard_NV6ads_A10_v5, hasta un máximo de 2 GPUs completas por máquina virtual en las Standard_NV72ads_A10_v5. Además, esta nueva generación está basada en los últimos procesadores AMD EPYC 74F3V (Milan) con una frecuencia base de 3.2 GHz y una pico de 4.0 GHz. 

Todo ello hace de esta serie una de las más interesantes a día de hoy para cubrir tanto las necesidades más básicas de visualización hasta las más demandantes. Si necesitas configurar un entorno Linux para ello, este artículo te guía paso a paso. La configuración está basada en CentOS 7.9 como sistema operativo, emplea la versión de los drivers 510.73 debido a los requisitos impuestos porla versión GRID 14.1, y proporciona acceso remoto a través de TurboVNC junto con VirtualGL para la aceleración 3D.

El URN de la imagen exacta empleada es *"OpenLogic:CentOS:7_9-gen2:latest"*. Es importante tenerlo en cuenta ya que existen múltiples variantes tanto en la versión del sistema operativo, como de la generación, como el software que lleva instalado por defecto (i.e. OpenLogic:CentOS-HPC:7_9-gen2:latest)

El proceso se basa en los scripts de configuración en las imágenes usadas por [Azure HPC On-Demand Platform](https://github.com/Azure/az-hop) con drivers y versiones del software actualizadas.

## Preparación del sistema operativo

En este primer paso actualizaremos la imagen base disponible en Azur. También será necesario instalar las cabeceras del kernel de Linux y el soporte a Dynamic Kernel Module (DKMS). Estos dos últimos son empleados por los drivers de NVIDIA para generar el módulo necesario y cargarlo sin necesidad de modificar el kernel de forma completa. 

La versión del kernel empleada es la 3.10.0-1160.76.1

```bash
sudo yum update -y 
sudo yum install -y kernel-devel
# DKMS únicamente está disponible en los repos EPEL de Fedora.
sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum install -y dkms

sudo reboot
```

Este reinicio permite que el sistema operativo coja los cambios tras la actualización y evitar errores más adelante. Por ejemplo, el instalador de NVIDIA no encontrará las cabeceras del kernel correctamente de forma automática.

## Instalación de los drivers NVIDIA GRID

Dado que vamos a utilizar los drivers propietarios de NVIDIA, el primer paso es evitar que el kernel cargue los drivers Nouveau de código abierto. Es posible ejecutar lo siguiente como root o editar el fichero directamente con tu editor de texto preferido (i.e. nano, vim, etc.).

```bash
cat <<EOF >/etc/modprobe.d/nouveau.conf
blacklist nouveau
blacklist lbm-nouveau
EOF
```

Tras ello, instalamos los drivers de NVIDIA GRID. Es muy importante hacer uso del instalador proporcionado directamente por Microsoft en lugar de los disponibles en la página de NVIDIA. Esta versión incluye ya el licenciamiento GRID para ser utilizado en Azure configurado. Si utilizar los drivers propios de NVIDIA tendrás que configurar un servidor de licenciamiento y adquirir las licencias correspondientes, algo que no tiene sentido al estar incluidas ya en el precio de la máquina virtual.


```bash
wget -O NVIDIA-Linux-x86_64-grid.run https://download.microsoft.com/download/6/2/5/625e22a0-34ea-4d03-8738-a639acebc15e/NVIDIA-Linux-x86_64-510.73.08-grid-azure.run 
chmod +x NVIDIA-Linux-x86_64-grid.run
sudo ./NVIDIA-Linux-x86_64-grid.run -s 
```` 

Una vez instalado con éxito, es necesario modificar la configuración de NVIDIA GRID. Para ello utilizaremos el fichero de ejemplo proporcionado por NVIDIA:

```bash
sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
```

Será necesario realizar los siguientes cambios:

- Comentar la sección de FeatureType ya que no es necesario en esta versión personalizada de los drivers en Azure
- Deshabilitar la interfaz de licenciamiento en nvidia-settings con EnableUI=FALSE ya que es gestionado automáticamente en Azure.
- Añadir IgnoreSP=FALSE, este último no he sido capaz de encontrar el porqué más allá de que la documentación lo pide.

```bash
sudo su -
cat <<EOF >>/etc/nvidia/gridd.conf
IgnoreSP=FALSE
EnableUI=FALSE 
EOF
sed -i '/FeatureType=0/d' /etc/nvidia/gridd.conf

reboot
````

Tras reiniciar, para permitir que el kernel emplee los nuevos drivers recién instalados, podremos ver que la tarjeta está correctamente configurada.

```bash
nvidia-smi

+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.73.08    Driver Version: 510.73.08    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA A10-4Q       On   | 0000E7AB:00:00.0 Off |                    0 |
| N/A   N/A    P8    N/A /  N/A |      0MiB /  4096MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

## Instalación del acceso remoto por VNC con TurboVNC y VirtualGL

Las imágenes de Linux del marketplace de Azure no vienen por defecto con un entorno gráfico. Es por ello que necesitaremos instalar tanto el gestor de ventanas de X.org como un entorno de escritorio. En ese caso, utilizaremos [Xfce](https://xfce.org/) debido a su bajo consumo de recursos, ideal para un entorno de trabajo remoto en la nube.

```bash
sudo yum groupinstall -y "X Window system"
sudo yum groupinstall -y xfce
```

Una vez instalado el entorno gráfico, lo siguiente será configurar el acceso por VNC. Emplearemos [TurboVNC](https://turbovnc.org/About/Introduction) ya que se encuentra optimizado para entornos de trabajo de vídeo y 3D. Su integración con [VirtualGL](https://virtualgl.org/About/Background) permite disponer de una solución robusta y de alto rendimiento para este tipo de aplicaciones sobre cualquier tipo de red. 

```bash
sudo yum install -y https://jztkft.dl.sourceforge.net/project/turbovnc/3.0.1/turbovnc-3.0.1.x86_64.rpm

sudo wget --no-check-certificate "https://virtualgl.com/pmwiki/uploads/Downloads/VirtualGL.repo" -O /etc/yum.repos.d/VirtualGL.repo

sudo yum install -y VirtualGL turbojpeg xorg-x11-apps
```

A la hora de configurar VirtualGL, para que los cambios de permisos aplicados sean efectivos, es necesario parar el gestor de ventanas y descargar los módulos del kernel. Si no, el asistente de configuración te indicará que los cambios no serán efectivos hasta que lo hagas.

```bash
sudo service gdm stop
sudo rmmod nvidia_drm nvidia_modeset nvidia
sudo /usr/bin/vglserver_config -config +s +f -t
sudo service gdm start
```

Tras ello, configuramos que por defecto systemd arranque en modo gráfico y, para evitar un reinicio, lo arrancamos directamente en la sesión actual.

```bash
sudo systemctl set-default graphical.target
sudo systemctl isolate graphical.target
``` 

El último paso es indicar qué queremos ejecutar cuando accedamos por TurboVNC y genere un nuevo display en el servidor de las X. En nuestro caso, queremos una nueva sesión de Xfce para poder trabajar.

```bash
cd $HOME
echo "xfce4-session" > ~/.Xclients
chmod a+x ~/.Xclients
```

Tras ello, solo tienes que instalar el cliente de TurboVNC en tu máquina local y conectarte a la IP o DNS asociado a tu máquina virtual desplegada en Azure. El resultado será este:

[![](/assets/img/turbovncxfce.png)](/assets/img/turbovncxfce.png)

## Configuraciones extra recomendadas

### Actualización PCI Bus

Si reiniciamos nuestra máquina virtual o esta es redesplegada en otro host por un fallo de hardware, el identificador del bus PCI puede variar. Esto provocará que nuestro entorno gráfico no funcione correctamente al no ser posible encontrar la tarjeta gráfica.

Para evitarlo, es recomendable configurar este script que ajusta la configuración del BusPCI cada vez que se inicia la máquina virtual para asegurarnos de que se mantiene sincronizado.

```bash
sudo su -
cat <<EOF >/etc/rc.d/rc3.d/busidupdate.sh
#!/bin/bash
BUSID=\$(nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print \$4}')
nvidia-xconfig --enable-all-gpus --allow-empty-initial-configuration -c /etc/X11/xorg.conf --virtual=1920x1200 --busid \$BUSID -s
# https://virtualgl.org/Documentation/HeadlessNV
sed -i '/BusID/a\    Option         "HardDPMS" "false"' /etc/X11/xorg.conf
EOF
chmod +x /etc/rc.d/rc3.d/busidupdate.sh
/etc/rc.d/rc3.d/busidupdate.sh
 ```


### Create a vglrun alias

A la hora de configurar la acelaración de nuestro entorno gráfico lo podemos hacer a nivel de toda la sesión o a nivel de aplicación. Empleando Xfce como entorno de escritorio no es necesario lo primero y podemos dedicar todos los recursos de la GPU para nuestras aplicaciones. 

Para asegurarnos de que las aplicaciones hacen uso de la acelaración, es necesario ejecutarlas a través del comando *vglrun*. Para hacer el proceso más sencillo y asegurarnos de utilizar todas las GPUs disponibles en el nodo, este script genera un alias con la configuración necesaria.

```bash
sudo su -
cat <<EOF >/etc/profile.d/vglrun.sh 
#!/bin/bash
ngpu=\$(/usr/sbin/lspci | grep NVIDIA | wc -l)
alias vglrun='/usr/bin/vglrun -d :0.\$(( \${port:-0} % \${ngpu:-1}))'
EOF
```


### Incrementar el tamaño de los buffers de red

Es posible que la configuración predeterminada de red y dispositivo de red de Linux no proporcione un rendimiento (ancho de banda) y latencia óptimos para escenarios de trabajo en paralelo. Es por ello que es recomendable incrementar el tamaño de los buffers de escritura y lectura a nivel del sistema operativo.

```bash
cat << EOF >>/etc/sysctl.conf
net.core.rmem_max=2097152
net.core.wmem_max=2097152
EOF
```

Si has llegado aquí, ¡felicidades!, ya tienes disponible tu workstation Linux para visualización 3D en Azure. El siguiente paso será instalar las aplicaciones necesarias para tu caso de uso concreto.




