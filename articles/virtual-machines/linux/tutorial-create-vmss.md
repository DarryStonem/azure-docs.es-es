---
title: "Creación de un conjunto de escalado de máquinas virtuales Linux en Azure | Microsoft Docs"
description: "Cree e implemente una aplicación de alta disponibilidad en máquinas virtuales Linux con un conjunto de escalado de máquinas virtuales."
services: virtual-machine-scale-sets
documentationcenter: 
author: iainfoulds
manager: timlt
editor: 
tags: 
ms.assetid: 
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: article
ms.date: 04/18/2017
ms.author: iainfou
ms.translationtype: Human Translation
ms.sourcegitcommit: be3ac7755934bca00190db6e21b6527c91a77ec2
ms.openlocfilehash: 6be49be9e4321075aa76b3abcf4695d0e7b45f6e
ms.contentlocale: es-es
ms.lasthandoff: 05/03/2017

---

# <a name="create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-linux"></a>Creación de un conjunto de escalado de máquinas virtuales e implementación de una aplicación de alta disponibilidad en Linux
En este tutorial, aprenderá cómo los conjuntos de escalado de máquinas virtuales de Azure le permiten escalar rápidamente el número de máquinas virtuales (VM) ejecutando la aplicación. El conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático idénticas. Puede escalar el número de máquinas virtuales del conjunto de escalado manualmente, o definir reglas de escalado automático basado en el uso de la CPU, la demanda de memoria o el tráfico de red. Para ver un conjunto de escalado de máquinas virtuales en acción, compile una aplicación de Node.js que se ejecute en varias máquinas virtuales Linux.

Se pueden completar los pasos de este tutorial con la versión más reciente de la [CLI de Azure 2.0](/cli/azure/install-azure-cli).


## <a name="scale-set-overview"></a>Introducción al conjunto de escalado
El conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático idénticas. Los conjuntos de escalado usan los mismos componentes que ha aprendido en el tutorial anterior sobre [Creación de máquinas virtuales de alta disponibilidad](tutorial-availability-sets.md). Las máquinas virtuales de un conjunto de escalado se crean en un conjunto de disponibilidad y se distribuyen entre los dominios de error lógico y de actualización.

Las máquinas virtuales se crean según sea necesario en un conjunto de escalado. Defina reglas de escalado automático para controlar cómo y cuándo se agregan o se quitan las máquinas virtuales del conjunto de escalado. Estas reglas se pueden desencadenar en función de métricas como la carga de la CPU, el uso de la memoria o el tráfico de red.

Los conjuntos de escalado admiten hasta 1000 máquinas virtuales cuando se usa una imagen de la plataforma de Azure. Para las cargas de trabajo de producción, puede que desee la [creación de una imagen de máquina virtual personalizada](tutorial-custom-images.md). Puede crear hasta 100 máquinas virtuales en un conjunto de escalado al usar una imagen personalizada.


## <a name="create-an-app-to-scale"></a>Creación de una aplicación para escalar
Para su uso en producción, puede que desee [crear una imagen de máquina virtual personalizada](tutorial-custom-images.md) que incluya la instalación y configuración de la aplicación. En este tutorial, vamos a personalizar las máquinas virtuales del primer arranque para ver rápidamente un conjunto de escalado en funcionamiento.

En un tutorial anterior, aprendió [cómo personalizar una máquina virtual Linux en el primer arranque](tutorial-automate-vm-deployment.md) con cloud-init. Pues bien, el mismo archivo de configuración cloud-init puede usarlo para instalar NGINX y ejecutar una aplicación sencilla Node.js "Hello World". Cree un archivo denominado *cloud-init.txt* y pegue la siguiente configuración:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Antes de poder crear un conjunto de escalado, cree un grupo de recursos con [az group create](/cli/azure/group#create). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupScaleSet* en la ubicación *westus*:

```azurecli
az group create --name myResourceGroupScaleSet --location westus
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [az vmss create](/cli/azure/vmss#create). En el ejemplo siguiente se crea un conjunto de escalado denominado *myScaleSet*, se usa el archivo cloud-init para personalizar la máquina virtual y se generan claves SSH si estas no existen aún:

```azurecli
az vmss create \
  --resource-group myResourceGroupScaleSet \
  --name myScaleSet \
  --image Canonical:UbuntuServer:14.04.4-LTS:latest \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.txt \
  --admin-username azureuser \
  --generate-ssh-keys      
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.


## <a name="allow-web-traffic"></a>Permitir tráfico web
Se creó automáticamente un equilibrador de carga como parte del conjunto de escalado de máquinas virtuales. El equilibrador de carga distribuye el tráfico a través de un conjunto de máquinas virtuales definidas usando reglas de equilibrador de carga. Puede aprender más información sobre los conceptos y la configuración del equilibrador de carga en el siguiente tutorial, [How to load balance virtual machines in Azure](tutorial-load-balancer.md) (Equilibrio de carga en máquinas virtuales de Azure).

Para permitir que el tráfico llegue a la aplicación web, cree una regla con [az network lb rule create](/cli/azure/network/lb/rule#create). En el ejemplo siguiente, se crea una regla denominada *myLoadBalancerRuleWeb*:

```azurecli
az network lb rule create \
  --resource-group myResourceGroupScaleSet \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

## <a name="test-your-app"></a>Prueba de la aplicación
Para ver la aplicación de Node.js en la web, obtenga la dirección IP pública del equilibrador de carga con [az network public-ip show](/cli/azure/network/public-ip#show). En el ejemplo siguiente se obtiene la dirección IP de *myLoadBalancerRuleWeb* que se ha creado como parte del conjunto de escalado:

```azurecli
az network public-ip show \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSetLBPublicIP \
    --query [ipAddress] \
    --output tsv
```

Escriba la dirección IP pública en un explorador web. Se muestra la aplicación, incluido el nombre de host de la máquina virtual a la que el equilibrador de carga distribuye el tráfico:

![Ejecución de la aplicación Node.js](./media/tutorial-create-vmss/running-nodejs-app.png)

Para ver el conjunto de escalado en funcionamiento, realice una actualización forzada del explorador web para ver cómo el equilibrador de carga distribuye el tráfico entre las máquinas virtuales del conjunto de escalado que ejecutan la aplicación.


## <a name="management-tasks"></a>Tareas de administración
Durante el ciclo de vida del conjunto de escalado, debe ejecutar una o varias tareas de administración. Además, puede crear scripts para automatizar varias tareas de ciclo de vida. La CLI de Azure 2.0 proporciona una forma rápida de realizar esas tareas. A continuación, presentamos algunas tareas comunes.

### <a name="view-vms-in-a-scale-set"></a>Visualización de máquinas virtuales en un conjunto de escalado
Para ver una lista de las máquinas virtuales en ejecución en el conjunto de escalado, use [az vmss list-instances](/cli/azure/vmss#list-instances) como se indica a continuación:

```azurecli
az vmss list-instances \
  --resource-group myResourceGroupScaleSet \
  --name myScaleSet \
  --output table
```

La salida es similar a la del ejemplo siguiente:

```azurecli
  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup            VmId
------------  --------------------  ----------  ------------  -------------------  -----------------------  ------------------------------------
           1  True                  westus      myScaleSet_1  Succeeded            MYRESOURCEGROUPSCALESET  c72ddc34-6c41-4a53-b89e-dd24f27b30ab
           3  True                  westus      myScaleSet_3  Succeeded            MYRESOURCEGROUPSCALESET  44266022-65c3-49c5-92dd-88ffa64f95da
```


### <a name="increase-or-decrease-vm-instances"></a>Aumento o disminución de instancias de máquina virtual
Para ver el número de instancias que tiene actualmente en un conjunto de escalado, use [az vmss show](/cli/azure/vmss#show) y realice consultas a *sku.capacity*:

```azurecli
az vmss show \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```

A continuación, puede aumentar o reducir manualmente el número de máquinas virtuales del conjunto de escalado con [az vmss scale](/cli/azure/vmss#scale). En el ejemplo siguiente se establece el número de máquinas virtuales en el conjunto de escalado en *5*:

```azurecli
az vmss scale \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --new-capacity 5
```

Las reglas de escalado automático le permiten definir cómo escalar o reducir verticalmente el número de máquinas virtuales en el conjunto de escalado en respuesta a la demanda, como el tráfico de red o el uso de CPU. Actualmente, no se pueden establecer estas reglas en la CLI de Azure 2.0. Use [Azure Portal](https://portal.azure.com) para configurar el escalado automático.

### <a name="get-connection-info"></a>Obtención de información de conexión
Para obtener información de conexión sobre las máquinas virtuales de los conjuntos de escalado, use [az vmss list-instance-connection-info](/cli/azure/vmss#list-instance-connection-info). Este comando da como resultado la dirección IP pública y los puertos de cada máquina virtual que le permiten conectarse con SSH:

```azurecli
az vmss list-instance-connection-info \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet
```


## <a name="use-data-disks-with-scale-sets"></a>Uso de discos de datos con conjuntos de escalado
Puede crear y usar discos de datos con conjuntos de escalado. En un tutorial anterior, ha aprendido cómo [administrar discos de Azure](tutorial-manage-disks.md), donde se describían las prácticas recomendadas y mejoras de rendimiento para la creación de aplicaciones en los discos de datos en lugar de en el disco del SO.

### <a name="create-scale-set-with-data-disks"></a>Creación de conjunto de escalado con discos de datos
Para crear un conjunto de escalado y conectar discos de datos, agregue el parámetro `--data-disk-sizes-gb` al comando [az vmss create](/cli/azure/vmss#create). En el ejemplo siguiente se crea un conjunto de escalado con discos de datos de *50*Gb conectados a cada instancia:

```azurecli
az vmss create \
  --resource-group myResourceGroupScaleSet \
  --name myScaleSetDisks \
  --image Canonical:UbuntuServer:14.04.4-LTS:latest \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.txt \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 50
```

Cuando se quitan las instancias de un conjunto de escalado, también se quitan los discos de datos conectados.

### <a name="add-data-disks"></a>Adición de discos de datos
Para agregar un disco de datos a las instancias en el conjunto de escalado, use [az vmss disk attach](/cli/azure/vmss/disk#attach). En el ejemplo siguiente se agrega un disco de *50* Gb a cada instancia:

```azurecli
az vmss disk attach `
    --resource-group myResourceGroupScaleSet `
    --name myScaleSet `
    --size-gb 50 `
    --lun 2
```

### <a name="detach-data-disks"></a>Desacoplamiento de discos de datos
Para quitar un disco de datos a las instancias en el conjunto de escalado, use [az vmss disk detach](/cli/azure/vmss/disk#detach). En el ejemplo siguiente se quita el disco de datos en el LUN *2* de cada instancia:

```azurecli
az vmss disk detach `
    --resource-group myResourceGroupScaleSet `
    --name myScaleSet `
    --lun 2
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a crear un conjunto de escalado de máquinas virtuales. Pase al siguiente tutorial para obtener más información sobre el concepto de equilibrio de carga de las máquinas virtuales.

[Load balance virtual machines](tutorial-load-balancer.md) (Equilibrio de carga de máquinas virtuales)

