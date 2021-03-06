---
title: "Características de alta disponibilidad de HDInsight (Hadoop) | Microsoft Docs"
description: "Obtenga información acerca de cómo los clústeres de HDInsight basados en Linux mejoran la confiabilidad y la disponibilidad mediante el uso de un nodo principal adicional. Obtenga información sobre cómo esto afecta a los servicios de Hadoop como Ambari y Hive, y cómo conectarse individualmente a cada nodo principal mediante SSH."
services: hdinsight
editor: cgronlun
manager: jhubbard
author: Blackmist
documentationcenter: 
tags: azure-portal
ms.assetid: 99c9f59c-cf6b-4529-99d1-bf060435e8d4
ms.service: hdinsight
ms.custom: hdinsightactive
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 04/03/2017
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: aaf97d26c982c1592230096588e0b0c3ee516a73
ms.openlocfilehash: 6e001d497dba1e3cc0a987fd0950854fe2564d2c
ms.lasthandoff: 04/27/2017


---
# <a name="availability-and-reliability-of-hadoop-clusters-in-hdinsight"></a>Disponibilidad y fiabilidad de clústeres de Hadoop en HDInsight

Los clústeres de HDInsight proporcionan dos nodos principales para aumentar la disponibilidad y la confiabilidad de los servicios y trabajos de Hadoop en ejecución.

Hadoop logra una alta disponibilidad y confiabilidad mediante la retención de copias de los datos y servicios en varios nodos de un clúster. Sin embargo, las distribuciones estándar de Hadoop suelen tener un único nodo principal. Cualquier interrupción de ese nodo principal puede causar que el clúster deje de funcionar. No plantea ningún problema con HDInsight.

> [!IMPORTANT]
> Linux es el único sistema operativo que se usa en la versión 3.4 de HDInsight, o en las superiores. Para más información, consulte [El contrato de nivel de servicio para las versiones de clúster de HDInsight](hdinsight-component-versioning.md#hdi-version-33-nearing-deprecation-date).

## <a name="understanding-the-nodes"></a>Descripción de los nodos

Los nodos de un clúster de HDInsight se implementan mediante Máquinas virtuales de Azure. Si se produce un error en un nodo, se desconecta y se crea un nuevo nodo para reemplazar el nodo con error. Mientras el nodo está sin conexión, se utiliza otro nodo del mismo tipo hasta que se vuelve a conectar el nuevo nodo.

> [!NOTE]
> Si el nodo está analizando los datos cuando se produce un error, se pierde su progreso en el trabajo. El trabajo se vuelve a enviar a otro nodo.

En las secciones siguientes se describen los tipos de nodo individuales usados con HDInsight. No todos los tipos de nodo se utilizan para un tipo de clúster. Por ejemplo, un tipo de clúster de Hadoop no tiene ningún nodo Nimbus. Para más información sobre los nodos usados por los tipos de clúster de HDInsight, vea la sección Tipos de clúster en el documento [Creación de clústeres de Hadoop basados en Linux en HDInsight](hdinsight-hadoop-provision-linux-clusters.md#cluster-types).

### <a name="head-nodes"></a>Nodos principales

Ambos nodos principales están activos y en ejecución dentro del clúster de HDInsight al mismo tiempo. Algunos servicios, como HDFS o YARN, solo están “activos” en un nodo principal en un determinado momento. Otros servicios como HiveServer2 o MetaStore de Hive están activos en ambos nodos principales al mismo tiempo.

Los nodos principales y otros nodos de HDInsight tienen un valor numérico como parte del nombre de host del nodo. Por ejemplo, `hn0-CLUSTERNAME` o `hn4-CLUSTERNAME`.

> [!IMPORTANT]
> No asocie el valor numérico a si un nodo es principal o secundario. El valor numérico solo está presente para proporcionar un nombre único para cada nodo.

### <a name="nimbus-nodes"></a>Nodos Nimbus

Para los clústeres de Storm, los nodos Nimbus proporcionan una funcionalidad similar a la de JobTracker de Hadoop al distribuir y supervisar el procesamiento a través de nodos de trabajo. HDInsight proporciona dos nodos Nimbus para el tipo de clúster de Storm.

### <a name="zookeeper-nodes"></a>Nodos Zookeeper

Los nodos [ZooKeeper](http://zookeeper.apache.org/) se usan para la elección del líder de los servicios principales en los nodos principales y para asegurarse de que los servicios, los nodos de datos (trabajo) y las puertas de enlace saben en qué nodo principal está activo un servicio principal. De forma predeterminada, HDInsight proporciona tres nodos ZooKeeper.

### <a name="worker-nodes"></a>Nodos de trabajo

Los nodos de trabajo realizan el análisis de los datos reales cuando se envía un trabajo al clúster. Si se produce un error en un nodo de trabajo, la tarea que estaba realizando se envía a otro nodo de trabajo. De forma predeterminada, HDInsight crea cuatro nodos de trabajo. Puede cambiar este número para satisfacer sus necesidades durante y después de la creación del clúster.

### <a name="edge-node"></a>Nodo perimetral

El nodo perimetral no participa activamente en el análisis de datos dentro del clúster, pero se usa por desarrolladores o científicos de datos al trabajar con Hadoop. El nodo perimetral se encuentra en la misma Red virtual de Azure como los demás nodos del clúster y puede acceder directamente a todos los demás nodos. Como no participa en el análisis de datos para el clúster, se puede utilizar sin ninguna preocupación de llevar los recursos fuera los trabajos de análisis o servicios críticos de Hadoop.

Actualmente, el servidor de R en HDInsight es el único tipo de clúster que proporciona un nodo perimetral de forma predeterminada. Para el servidor de R en HDInsight, se usa el nodo perimetral para probar el código de R localmente en el nodo antes de enviarlo al clúster para su procesamiento distribuido.

Para más información sobre el uso de un nodo perimetral con tipos de clúster que no sean de R Server, consulte el documento [Uso de nodos perimetrales en HDInsight](hdinsight-apps-use-edge-node.md).

## <a name="accessing-the-nodes"></a>Acceso a los nodos

Se proporciona acceso al clúster a través de Internet mediante una puerta de enlace pública. El acceso está limitado a la conexión a los nodos principales y, si existe, al nodo perimetral. El hecho de contar con varios nodos principales no afecta al acceso a servicios que se ejecutan en los nodos principales. La puerta de enlace pública enruta las solicitudes al nodo principal que hospeda el servicio solicitado. Por ejemplo, si Ambari está hospedado en el nodo principal secundario, la puerta de enlace enruta las solicitudes entrantes de Ambari a ese nodo.

El acceso a través de la puerta de enlace pública se limita a los puertos 443 (HTTPS), 22 y 23.

* El puerto __443__ se usa para acceder a Ambari y a otra interfaz de usuario web o API de REST hospedadas en los nodos principales.

* El puerto __22__ se usa para acceder al nodo principal primario o al nodo perimetral mediante SSH.

* El puerto __23__ se usa para acceder al nodo principal secundario mediante SSH. Por ejemplo, `ssh username@mycluster-ssh.azurehdinsight.net` se conecta al nodo principal primario del clúster llamado **mycluster**.

Para más información sobre cómo usar SSH, vea el documento [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

### <a name="internal-fully-qualified-domain-names-fqdn"></a>Nombres completos de dominio (FQDN) internos

Los nodos de un clúster de HDInsight tienen una dirección IP interna y el FQDN al que solo se puede acceder desde el clúster. Al obtener acceso a servicios en el clúster mediante la dirección IP o FQDN interna, debe usar Ambari para comprobar la dirección IP o FQDN que se usará al obtener acceso al servicio.

Por ejemplo, el servicio de Oozie solo puede ejecutarse en un nodo principal y el uso del comando `oozie` desde una sesión de SSH requiere la dirección URL del servicio. La dirección URL puede conseguirse en Ambari mediante el comando siguiente:

    curl -u admin:PASSWORD "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations?type=oozie-site&tag=TOPOLOGY_RESOLVED" | grep oozie.base.url

Este comando devuelve un valor similar al siguiente comando, que contiene la dirección URL interna para usarla con el comando `oozie`:

    "oozie.base.url": "http://hn0-CLUSTERNAME-randomcharacters.cx.internal.cloudapp.net:11000/oozie"

Para más información sobre cómo trabajar con la API de REST de Ambari, vea [Supervisión y administración de HDInsight con la API de REST de Ambari](hdinsight-hadoop-manage-ambari-rest-api.md).

### <a name="accessing-other-node-types"></a>Acceso a otros tipos de nodos

Puede conectarse a los nodos que no son accesibles directamente a través de Internet mediante los métodos siguientes:

* **SSH**: una vez conectado a un nodo principal mediante SSH, puede usar SSH desde el nodo principal para conectarse a otros nodos del clúster. Para más información, vea el documento [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

* **Túnel SSH**: si tiene que acceder a un servicio web hospedado en uno de los nodos que no está expuesto a Internet, debe usar un túnel SSH. Para más información, vea el documento [Uso de un túnel SSH con HDInsight](hdinsight-linux-ambari-ssh-tunnel.md).

* **Azure Virtual Network**: si el clúster de HDInsight forma parte de una red de Azure Virtual Network, cualquier recurso en la misma red virtual puede acceder directamente a todos los nodos del clúster. Para más información, vea el documento [Extensión de las funcionalidades de HDInsight con Azure Virtual Network](hdinsight-extend-hadoop-virtual-network.md).

## <a name="how-to-check-on-a-service-status"></a>Cómo comprobar el estado del servicio

Para comprobar el estado de los servicios que se ejecutan en el nodo principal, use la interfaz de usuario web de Ambari o la API de REST de Ambari.

### <a name="ambari-web-ui"></a>Interfaz de usuario web de Ambari

La interfaz de usuario web de Ambari es visible en https://CLUSTERNAME.azurehdinsight.net. Reemplace **CLUSTERNAME** por el nombre del clúster. Si se le solicita, introduzca las credenciales de usuario HTTP de su clúster. El nombre de usuario HTTP predeterminado es **admin** y la contraseña es la contraseña que especificó al crear el clúster.

Cuando llegue a la página de Ambari, se enumeran los servicios instalados a la izquierda de la página.

![Servicios instalados](./media/hdinsight-high-availability-linux/services.png)

Hay una serie de iconos que pueden aparecer junto a un servicio para indicar el estado. Las alertas relacionadas con un servicio se pueden ver mediante el vínculo **Alertas** que se encuentra en la parte superior de la página. Puede seleccionar cada servicio para ver más información sobre él.

Mientras que la página del servicio proporciona información sobre el estado y la configuración de cada servicio, no proporciona información sobre en qué nodo principal se está ejecutando el servicio. Para ver esta información, use el vínculo **Hosts** de la parte superior de la página. En esta página se muestran los hosts del clúster, incluidos los nodos principales.

![lista de hosts](./media/hdinsight-high-availability-linux/hosts.png)

Al seleccionar el vínculo de uno de los nodos principales se muestran los servicios y componentes que se ejecutan en ese nodo.

![Estado del componente](./media/hdinsight-high-availability-linux/nodeservices.png)

Para más información sobre el uso de Ambari, vea [Supervisión y administración de HDInsight con la interfaz de usuario web de Ambari](hdinsight-hadoop-manage-ambari.md).

### <a name="ambari-rest-api"></a>API de REST de Ambari

La API de REST de Ambari está disponible en Internet, y la puerta de enlace pública controla las solicitudes de enrutamiento para el nodo principal que hospeda actualmente la API de REST.

Puede usar el siguiente comando para comprobar el estado de un servicio a través de la API de REST de Ambari:

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICENAME?fields=ServiceInfo/state

* Reemplace **CONTRASEÑA** por la contraseña de la cuenta del usuario (admin) HTTP.
* Reemplace **CLUSTERNAME** por el nombre del clúster.
* Reemplace **SERVICENAME** por el nombre del servicio para comprobar el estado de

Por ejemplo, para comprobar el estado del servicio **HDFS** en un clúster denominado **mycluster**, con la contraseña **password**, debería usar el comando siguiente:

    curl -u admin:password https://mycluster.azurehdinsight.net/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state

La respuesta es similar al siguiente formato JSON:

    {
      "href" : "http://hn0-CLUSTERNAME.randomcharacters.cx.internal.cloudapp.net:8080/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state",
      "ServiceInfo" : {
        "cluster_name" : "mycluster",
        "service_name" : "HDFS",
        "state" : "STARTED"
      }
    }

La dirección URL nos indica que el servicio se está ejecutando en el nodo principal **hn0-CLUSTERNAME**.

El estado nos indica que el servicio se está ejecutando o se ha **INICIADO**.

Si no sabe qué servicios están instalados en el clúster, puede usar el comando siguiente para recuperar una lista:

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services

Para más información sobre cómo trabajar con la API de REST de Ambari, vea [Supervisión y administración de HDInsight con la API de REST de Ambari](hdinsight-hadoop-manage-ambari-rest-api.md).

#### <a name="service-components"></a>Componentes de servicio

Los servicios pueden contener componentes cuyo estado desea comprobar de forma individual. Por ejemplo, HDFS contiene el componente NameNode. Para ver información sobre un componente, el comando sería:

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component

Si no sabe qué componentes proporciona un servicio, puede usar el comando siguiente para recuperar una lista:

    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component

## <a name="how-to-access-log-files-on-the-head-nodes"></a>Acceso a los archivos de registro en los nodos principales

### <a name="ssh"></a>SSH

Mientras está conectado a un nodo principal a través de SSH, los archivos de registro pueden encontrarse en **/var/log**. Por ejemplo, **/var/log/hadoop-yarn/yarn** contiene registros de YARN.

Cada nodo principal puede tener entradas de registro único, por lo que debe comprobar los registros en ambos.

### <a name="sftp"></a>SFTP

También se puede conectar con el nodo principal mediante el protocolo SSH File Transfer Protocol o el protocolo seguro de transferencia de archivos (SFTP) y descargar los archivos de registro directamente.

De igual forma a utilizar un cliente SSH, al conectarse al clúster debe proporcionar el nombre de cuenta de usuario SSH y la dirección SSH del clúster. Por ejemplo: `sftp username@mycluster-ssh.azurehdinsight.net`. Debe proporcionar la contraseña de la cuenta cuando se le solicite, o bien proporcionar una clave pública mediante el parámetro `-i`.

Una vez conectado, se le presentará un símbolo del sistema `sftp>` . Desde este símbolo del sistema, puede cambiar los directorios, cargar y descargar archivos. Por ejemplo, los siguientes comandos cambian los directorios al directorio **/var/log/hadoop/hdfs** y después descargan todos los archivos en el directorio.

    cd /var/log/hadoop/hdfs
    get *

Para ver una lista de comandos disponibles, escriba `help` en el símbolo del sistema `sftp>`.

> [!NOTE]
> También hay interfaces gráficas que le permiten visualizar el sistema de archivos cuando se conecta mediante SFTP. Por ejemplo, [MobaXTerm](http://mobaxterm.mobatek.net/) le permite examinar el sistema de archivos mediante una interfaz similar al Explorador de Windows.

### <a name="ambari"></a>Ambari

> [!NOTE]
> Para acceder a archivos de registro mediante Ambari, debe usar un túnel SSH. La interfaz web de los servicios individuales no se exponen públicamente en Internet. Para más información sobre cómo usar un túnel SSH, consulte [Uso de la tunelización SSH para acceder a la interfaz de usuario web de Ambari, ResourceManager, JobHistory, NameNode, Oozie y otras interfaces de usuario web](hdinsight-linux-ambari-ssh-tunnel.md).

En la interfaz de usuario web de Ambari, seleccione el servicio para el que desea consultar los registros (por ejemplo, YARN). Después, utilice **Vínculos rápidos** para seleccionar de qué nodo principal desea consultar los registros.

![Uso de vínculos rápidos para ver los registros](./media/hdinsight-high-availability-linux/viewlogs.png)

## <a name="how-to-configure-the-node-size"></a>Configuración del tamaño del nodo

El tamaño de un nodo solo se puede seleccionar durante la creación del clúster. Puede encontrar una lista de los diferentes tamaños de máquina virtual disponibles para HDInsight, incluido el núcleo, la memoria y el almacenamiento local para cada uno, en la [página de precios de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

Al crear un nuevo clúster, puede especificar el tamaño de los nodos. A continuación se ofrece información sobre cómo especificar el tamaño mediante [Azure Portal][preview-portal], [Azure PowerShell][azure-powershell] y la [CLI de Azure][azure-cli]:

* **Azure Portal**: al crear un clúster, puede establecer el tamaño de los nodos que usa el clúster:

    ![Imagen del asistente para creación de clústeres con selección del tamaño del nodo](./media/hdinsight-high-availability-linux/headnodesize.png)

* **CLI de Azure**: cuando se usa el comando `azure hdinsight cluster create`, puede establecer el tamaño de los nodos principal, de trabajo y ZooKeeper mediante los parámetros `--headNodeSize`, `--workerNodeSize` y `--zookeeperNodeSize`.

* **Azure PowerShell**: cuando se usa el cmdlet `New-AzureRmHDInsightCluster`, puede establecer el tamaño de los nodos principal, de trabajo y ZooKeeper mediante los parámetros `-HeadNodeVMSize`, `-WorkerNodeSize` y `-ZookeeperNodeSize`.

## <a name="next-steps"></a>Pasos siguientes

Use los siguientes vínculos para obtener más información sobre los aspectos que se mencionan en este documento.

* [Referencia de REST de Ambari](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)
* [Instalación y configuración de la interfaz de la línea de comandos de Azure](../cli-install-nodejs.md)
* [Instale y configure Azure PowerShell.](/powershell/azure/overview)
* [Administración de HDInsight mediante Ambari](hdinsight-hadoop-manage-ambari.md)
* [Aprovisionamiento de clústeres de HDInsight basado en Linux](hdinsight-hadoop-provision-linux-clusters.md)

[preview-portal]: https://portal.azure.com/
[azure-powershell]: /powershell/azureps-cmdlets-docs
[azure-cli]: ../cli-install-nodejs.md

