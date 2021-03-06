---
title: "Introducción a la CLI de Azure para Batch | Microsoft Docs"
description: "Obtenga una introducción rápida a los comandos de Batch en la CLI de Azure para administrar los recursos del servicio Azure Batch"
services: batch
documentationcenter: 
author: tamram
manager: timlt
editor: 
ms.assetid: fcd76587-1827-4bc8-a84d-bba1cd980d85
ms.service: batch
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: multiple
ms.workload: big-compute
ms.date: 01/23/2017
ms.author: tamram
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 0d8472cb3b0d891d2b184621d62830d1ccd5e2e7
ms.openlocfilehash: 698c481e2eff5e0a3b893a0377d9f4cd2f052eb4
ms.lasthandoff: 03/21/2017


---
# <a name="manage-batch-resources-with-azure-cli"></a>Administración de recursos de Batch con la CLI de Azure

La interfaz de línea de comandos de Azure(CLI de Azure) multiplataforma (CLI de Azure) le permite administrar sus cuentas de Batch, y recursos como grupo, trabajos y tareas en los shells de comandos de Linux, Mac y Windows. Con la CLI de Azure, puede realizar directamente y mediante scripts muchas de las mismas tareas que lleva a cabo con las API de Batch, el portal de Azure y los cmdlets de Batch PowerShell.

En este artículo se usa la versión 0.10.5 de la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos
* [Instalación de la CLI de Azure](../cli-install-nodejs.md)
* [Conexión a una suscripción de Azure desde la interfaz de la línea de comandos de Azure (CLI de Azure)](../xplat-cli-connect.md)
* Cambie al modo **Resource Manager**: `azure config mode arm`

> [!TIP]
> Se recomienda actualizar su instalación de la CLI de Azure con frecuencia para aprovechar las mejoras y actualizaciones del servicio.
> 
> 

## <a name="command-help"></a>Ayuda de comandos
Para mostrar texto de ayuda de todos los comandos de la CLI de Azure, anexe `-h` como la única opción después del comando. Por ejemplo:

* Para obtener ayuda sobre el comando `azure` escriba: `azure -h`
* Para obtener una lista de todos los comandos de Batch en la CLI, utilice: `azure batch -h`
* Para obtener ayuda acerca de la creación de una cuenta de Batch, escriba: `azure batch account create -h`

En caso de duda, utilice la opción de línea de comandos `-h` para obtener ayuda sobre cualquier comando de CLI de Azure.

## <a name="create-a-batch-account"></a>Crear una cuenta de lote
Uso:

    azure batch account create [options] <name>

Ejemplo:

    azure batch account create --location "West US"  --resource-group "resgroup001" "batchaccount001"

Crea una nueva cuenta de Batch con los parámetros especificados. Debe especificar al menos una ubicación, un grupo de recursos y un nombre de cuenta. Si no tiene un grupo de recursos, créelo ejecutando `azure group create` y especifique una de las regiones de Azure (como "oeste de EE. UU.") para la opción `--location`. Por ejemplo:

    azure group create --name "resgroup001" --location "West US"

> [!NOTE]
> El nombre de la cuenta de Batch debe ser único en la región de Azure en que se crea la cuenta. Puede contener solo caracteres alfanuméricos en minúscula y su longitud debe oscilar entre 3 y 24 caracteres. No se pueden usar caracteres especiales como `-` o `_` en nombres de cuenta de Batch.
> 
> 

### <a name="linked-storage-account-autostorage"></a>Cuenta de almacenamiento vinculada (almacenamiento automático)
Al crear su cuenta de Batch, puede vincularla a (opcionalmente) una cuenta de almacenamiento **de uso general** . La característica de [paquetes de aplicación](batch-application-packages.md) de Batch usa el almacenamiento de blobs en un cuenta de almacenamiento de uso general, al igual que la biblioteca [.NET de convenciones de archivo de Lote](batch-task-output.md). Estas características opcionales le ayudan a implementar las aplicaciones que ejecutan las tareas de Lote y a conservar los datos que generan.

Para vincular una cuenta existente de Azure Storage existente a una nueva cuenta de Batch al crearla, especifique la opción `--autostorage-account-id`. Esta opción requiere el Id. de recurso completo de la cuenta de almacenamiento.

En primer lugar, muestre los detalles de su cuenta de almacenamiento:

    azure storage account show --resource-group "resgroup001" "storageaccount001"

Luego, use el valor **Url** para la opción `--autostorage-account-id`. El valor Url comienza por "/ subscriptions /" y contiene el Id. de suscripción y la ruta de acceso de los recursos a la cuenta de Storage:

    azure batch account create --location "West US"  --resource-group "resgroup001" --autostorage-account-id "/subscriptions/8ffffff8-4444-4444-bfbf-8ffffff84444/resourceGroups/resgroup001/providers/Microsoft.Storage/storageAccounts/storageaccount001" "batchaccount001"

## <a name="delete-a-batch-account"></a>Eliminar una cuenta de lote
Uso:

    azure batch account delete [options] <name>

Ejemplo:

    azure batch account delete --resource-group "resgroup001" "batchaccount001"

Elimina la cuenta de Batch especificada. Cuando se le solicite, confirme que desea quitar la cuenta (la cuenta puede tardar un tiempo en eliminarse).

## <a name="manage-account-access-keys"></a>Administración de claves de acceso de cuentas
Para [crear y modificar recursos](#create-and-modify-batch-resources) en una cuenta de Batch se necesita una clave de acceso.

### <a name="list-access-keys"></a>Enumeración de teclas de acceso
Uso:

    azure batch account keys list [options] <name>

Ejemplo:

    azure batch account keys list --resource-group "resgroup001" "batchaccount001"

Enumera las claves de cuenta para la cuenta de Batch dada.

### <a name="generate-a-new-access-key"></a>Generar una nueva clave de acceso
Uso:

    azure batch account keys renew [options] --<primary|secondary> <name>

Ejemplo:

    azure batch account keys renew --resource-group "resgroup001" --primary "batchaccount001"

Vuelve a generar la clave de cuenta especificada para la cuenta de Batch dada.

## <a name="create-and-modify-batch-resources"></a>Creación y modificación de recursos de Lote
La CLI de Azure se puede usar para crear, leer, actualizar y eliminar (CRUD) recursos de Batch como grupos, nodos de procesos, trabajos y tareas. Estas operaciones de CRUD requieren el nombre de la cuenta de Batch, la clave de acceso y el punto de conexión. Puede especificarlos con las opciones `-a`, `-k`, y `-u` o establecer [variables de entorno](#credential-environment-variables) que la CLI usa automáticamente (si se rellenan).

### <a name="credential-environment-variables"></a>Variables de entorno de credenciales
Puede establecer las variables de entorno `AZURE_BATCH_ACCOUNT`, `AZURE_BATCH_ACCESS_KEY` y `AZURE_BATCH_ENDPOINT`, en lugar de especificar las opciones `-a`, `-k`, y `-u` en la línea de comandos de cada comando que se ejecute. La CLI de Batch utiliza estas variables (si se establecen) para que pueda omitir las opciones `-a`, `-k`, y `-u`. El resto de este artículo asume que se usan estas variables de entorno.

> [!TIP]
> Enumere las claves con `azure batch account keys list` y muestre el punto de conexión de la cuenta con `azure batch account show`.
> 
> 

### <a name="json-files"></a>Archivos JSON
Al crear recursos de Batch, como grupos y trabajos, puede especificar un archivo JSON que contenga la configuración del recurso nuevo, en lugar de pasar sus parámetros como opciones de la línea de comandos. Por ejemplo:

`azure batch pool create my_batch_pool.json`

Aunque puede realizar muchas operaciones de creación de recursos utilizando solo las opciones de línea de comandos, algunas características requieren un archivo con formato JSON que contenga los detalles de los recursos. Por ejemplo, si desea especificar archivos de recursos para una tarea de inicio, debe utilizar un archivo JSON.

Para encontrar el archivo JSON necesario para crear un recurso, consulte la documentación de [Referencia de API de REST de Batch][rest_api] en MSDN. Cada tema de "Agregar *tipo de recurso*" contiene JSON de ejemplo para crear el recurso, que puede utilizar como plantillas para los archivos JSON. Por ejemplo, en [Add a pool to an account][rest_add_pool] (Incorporación de un grupo a una cuenta) puede encontrar JSON para crear grupos.

> [!NOTE]
> Si especifica un archivo JSON al crear un recurso, se ignoran los restantes parámetros que especifique en la línea de comandos de dicho recurso.
> 
> 

## <a name="create-a-pool"></a>Creación de un grupo
Uso:

    azure batch pool create [options] [json-file]

Ejemplo (configuración de máquina virtual):

    azure batch pool create --id "pool001" --target-dedicated 1 --vm-size "STANDARD_A1" --image-publisher "Canonical" --image-offer "UbuntuServer" --image-sku "14.04.2-LTS" --node-agent-id "batch.node.ubuntu 14.04"

Ejemplo (configuración de Cloud Services):

    azure batch pool create --id "pool002" --target-dedicated 1 --vm-size "small" --os-family "4"

Crea un grupo de nodos de proceso en el servicio de Batch.

Como se mencionó en [Información general de las características de Batch para desarrolladores](batch-api-basics.md#pool), al seleccionar un sistema operativo para los nodos del grupo dispone de dos opciones: **configuración de máquina virtual** y **configuración de Cloud Services**. Utilice las opciones `--image-*` para crear grupos de configuración de máquina virtual y `--os-family` para crear grupos de configuración de Cloud Service. No puede especificar las opciones `--os-family` y `--image-*`.

Puede especificar [paquetes de aplicación](batch-application-packages.md) del grupo y la línea de comandos de una [tarea de inicio](batch-api-basics.md#start-task). Sin embargo, para especificar los archivos de recursos de la tarea de inicio, debe utilizar un [archivo JSON](#json-files).

Elimine un grupo con:

    azure batch pool delete [pool-id]

> [!TIP]
> Compruebe en la [lista de imágenes de máquina virtual](batch-linux-nodes.md#list-of-virtual-machine-images) los valores adecuados para las opciones `--image-*`.
> 
> 

## <a name="create-a-job"></a>Creación de un trabajo
Uso:

    azure batch job create [options] [json-file]

Ejemplo:

    azure batch job create --id "job001" --pool-id "pool001"

Agrega un trabajo a la cuenta de Batch y especifica el grupo en el que ejecutan sus tareas.

Elimine un trabajo con:

    azure batch job delete [job-id]

## <a name="list-pools-jobs-tasks-and-other-resources"></a>Enumeración de grupos, trabajos, tareas y otros recursos
Cada tipo de recurso de proceso de Batch admite un comando `list` que consulta la cuenta de Batch y enumera los recursos de ese tipo. Por ejemplo, puede enumerar los grupos de su cuenta y las tareas de un trabajo:

    azure batch pool list
    azure batch task list --job-id "job001"

### <a name="listing-resources-efficiently"></a>Enumeración eficaz de los recursos
Para agilizar las consultas, puede especificar las opciones **select**, **filter** y **expand** de cláusula para las operaciones `list`. Utilice estas opciones para limitar la cantidad de datos que devuelve el servicio Batch. Dado que todo el filtrado se produce en el lado del servidor, solo cruzan la conexión los datos que le interesan. Use estas cláusulas para ahorrar ancho de banda (y, por consiguiente, tiempo) al realizar operaciones de lista.

Por ejemplo, esto devolverá solo aquellos grupos cuyos identificadores comiencen por "renderTask":

    azure batch task list --job-id "job001" --filter-clause "startswith(id, 'renderTask')"

La CLI de Batch admite las tres cláusulas compatibles con el servicio Batch:

* `--select-clause [select-clause]`  Devuelve un subconjunto de propiedades para cada entidad
* `--filter-clause [filter-clause]`  Devuelve solo las entidades que coincidan con la expresión de OData especificada
* `--expand-clause [expand-clause]`  Obtiene la información de la entidad en una única llamada REST subyacente. La cláusula expand solo admite la propiedad `stats` en este momento.

Para más información sobre los tres cláusulas y la realización de consultas de lista con ellos, consulte [Consulta eficaz del servicio Azure Batch](batch-efficient-list-queries.md).

## <a name="application-package-management"></a>Administración de paquetes de aplicación
Los paquetes de aplicación proporcionan una manera simplificada de implementar aplicaciones a los nodos de proceso de los grupos. Con la CLI de Azure, puede cargar paquetes de aplicación, administrar versiones del paquete y eliminar paquetes.

Para crear una nueva aplicación y agregar una versión del paquete:

**Cree** una aplicación:

    azure batch application create "resgroup001" "batchaccount001" "MyTaskApplication"

**Agregue** un paquete de aplicación:

    azure batch application package create "resgroup001" "batchaccount001" "MyTaskApplication" "1.10-beta3" package001.zip

**Active** el paquete:

    azure batch application package activate "resgroup001" "batchaccount001" "MyTaskApplication" "1.10-beta3" zip

Especifique la **versión predeterminada** de la aplicación:

    azure batch application set "resgroup001" "batchaccount001" "MyTaskApplication" --default-version "1.10-beta3"

### <a name="deploy-an-application-package"></a>Implementación de un paquete de aplicación
Puede especificar uno o varios paquetes de aplicación para implementarlos al crear un grupo nuevo. Cuando se especifica un paquete en el momento de la creación de un grupo, se implementa en todos los nodos cuando cada nodo se incorpora al grupo. También se implementan paquetes cuando un nodo se reinicia o se restablece su imagen inicial.

Especifique la opción `--app-package-ref` al crear un grupo para implementar un paquete de aplicación en los nodos del grupo cuando se unen a este. La opción `--app-package-ref` acepta una lista delimitada por puntos y comas de identificadores de aplicaciones que se van a implementar en los nodos de proceso.

    azure batch pool create --pool-id "pool001" --target-dedicated 1 --vm-size "small" --os-family "4" --app-package-ref "MyTaskApplication"

Actualmente, cuando crea un grupo mediante las opciones de línea de comandos, no se puede especificar *qué* versión del paquete de aplicación se va a implementar en los nodos de proceso, por ejemplo "1.10-beta3". Por lo tanto, primero debe especificar una versión predeterminada de la aplicación con `azure batch application set [options] --default-version <version-id>` antes de crear el grupo (consulte la sección anterior). Sin embargo, puede especificar una versión del paquete para el grupo si usa un [archivo JSON](#json-files) en lugar de las opciones de línea de comandos al crear el grupo.

Puede encontrar más información sobre los paquetes de aplicación en [Implementación de aplicaciones con paquetes de aplicación de Azure Batch](batch-application-packages.md).

> [!IMPORTANT]
> Para utilizar paquetes de aplicación, primero se debe [vincular una cuenta de Almacenamiento de Azure](#linked-storage-account-autostorage) a su cuenta de Lote.
> 
> 

### <a name="update-a-pools-application-packages"></a>Actualización de los paquetes de aplicación de un grupo
Para actualizar las aplicaciones asignadas a un grupo existente, emita el comando `azure batch pool set` con la opción `--app-package-ref`:

    azure batch pool set --pool-id "pool001" --app-package-ref "MyTaskApplication2"

Para implementar el nuevo paquete de aplicación en los nodos de proceso que ya están en un grupo existente, debe reiniciar o restablecer la imagen inicial de esos nodos:

    azure batch node reboot --pool-id "pool001" --node-id "tvm-3105992504_1-20160930t164509z"

> [!TIP]
> Puede obtener una lista de los nodos de un grupo junto con sus identificadores de nodo con `azure batch node list`.
> 
> 

Tenga en cuenta que ya debe haber configurado la aplicación con una versión predeterminada antes de la implementación (`azure batch application set [options] --default-version <version-id>`).

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas
Esta sección pretende proporcionar los recursos que se pueden utilizar al solucionar problemas de la CLI de Azure. No necesariamente solucionará todos los problemas, pero puede ayudarle a restringir la causa e indicarle recursos de ayuda.

* Use `-h` para obtener **texto de ayuda** de cualquier comando de CLI.
* Use `-v` y `-vv` para mostrar la salida **detallada** del comando; `-vv` proporciona detalles adicionales y muestra las solicitudes y respuestas de REST reales. Estos modificadores son útiles para mostrar la salida completa del error.
* Puede ver la **salida del comando como JSON** con la opción `--json`. Por ejemplo, `azure batch pool show "pool001" --json` muestra las propiedades de pool001 en formato JSON. Puede copiar y modificar esta salida para usarla en `--json-file` (consulte [Archivos JSON](#json-files) en este mismo artículo).
* El [foro de Batch en MSDN][batch_forum] es un recurso muy útil que supervisan muy de cerca los miembros del equipo de Batch. Publique en él sus preguntas si experimenta problemas o desea obtener ayuda acerca de una operación concreta.
* La CLI de Azure no es compatible actualmente con todas las operaciones de los recursos de Batch. Por ejemplo, actualmente no se puede especificar una *versión* del paquete de aplicación para un grupo, solo el identificador de paquete. En tales casos, es preciso proporcionar un `--json-file` para el comando, en lugar de utilizar opciones de línea de comandos. Asegúrese de mantenerse al día y tener la versión más reciente de la CLI para poder disfrutar las mejoras futuras.

## <a name="next-steps"></a>Pasos siguientes
* Consulte [Implementación de aplicaciones con paquetes de aplicación de Azure Batch](batch-application-packages.md) para averiguar cómo utilizar esta característica para administrar e implementar las aplicaciones que se ejecutan en nodos de proceso de Batch.
* Para más información acerca de cómo reducir el número de elementos y el tipo de información que se devuelve en las consultas a Batch, consulte [Consulta eficaz del servicio Azure Batch](batch-efficient-list-queries.md) .

[batch_forum]: https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=azurebatch
[github_readme]: https://github.com/Azure/azure-xplat-cli/blob/dev/README.md
[rest_api]: https://msdn.microsoft.com/library/azure/dn820158.aspx
[rest_add_pool]: https://msdn.microsoft.com/library/azure/dn820174.aspx

