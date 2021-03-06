---
title: Tendencias de Twitter con Apache Storm en HDInsight | Microsoft Docs
description: "Aprenda a usar Trident para crear una topología de Apache Storm que determine las tendencias en Twitter, según los hash tags."
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 63b280ea-5c07-4dc8-a35f-dccf5a96ba93
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 04/14/2017
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 0d6f6fb24f1f01d703104f925dcd03ee1ff46062
ms.openlocfilehash: d588221586f151319436525c5098b0bb2694e5f9
ms.lasthandoff: 04/17/2017


---
# <a name="determine-twitter-trending-topics-with-apache-storm-on-hdinsight"></a>Determinación de las tendencias de Twitter con Apache Storm en HDInsight

Aprenda a usar Trident para crear una topología de Storm que determine las tendencias (hash tags) en Twitter.

Trident es una abstracción de alto nivel que ofrece herramientas como uniones, agregaciones, agrupaciones, funciones y filtros. Además, Trident agrega primitivas para realizar el procesamiento incremental, con estado. El ejemplo usado en este documento es una topología de Trident con un spout y una función personalizados. También usa varias funciones integradas disponibles en Trident.

## <a name="requirements"></a>Requisitos

* <a href="http://www.oracle.com/technetwork/java/javase/downloads/index.html" target="_blank">Java y JDK 1.8</a>

* <a href="http://maven.apache.org/what-is-maven.html" target="_blank">Maven</a>

* <a href="http://git-scm.com/" target="_blank">Git</a>

* Una cuenta de desarrollador de Twitter

## <a name="download-the-project"></a>Descarga del proyecto

Use el código siguiente para clonar el proyecto de forma local.

    git clone https://github.com/Blackmist/TwitterTrending

## <a name="understanding-the-topology"></a>Descripción de la topología

El siguiente diagrama se muestra cómo fluyen los datos a través de esta topología:

![Topología](./media/hdinsight-storm-twitter-trending/trident.png)

> [!NOTE]
> Este diagrama es una vista simplificada de la topología. Se distribuyen varias instancias de los componentes entre los nodos del clúster.


El código de Trident que implementa la topología es como sigue:

    topology.newStream("spout", spout)
        .each(new Fields("tweet"), new HashtagExtractor(), new Fields("hashtag"))
        .groupBy(new Fields("hashtag"))
        .persistentAggregate(new MemoryMapState.Factory(), new Count(), new Fields("count"))
        .newValuesStream()
        .applyAssembly(new FirstN(10, "count"))
        .each(new Fields("hashtag", "count"), new Debug());

Este código realiza las acciones siguientes:

1. Crea un flujo desde el spout. El spout recupera los tweets de Twitter y los filtra por palabras clave específicas (love, music y coffee en este ejemplo).

2. Se utiliza HashtagExtractor, una función personalizada, para extraer hash tags de cada tweet. Estos hash tags se envían a la secuencia.

3. El flujo se agrupa por hash tag y se pasa a un agregador. Este agregador crea un recuento de cuántas veces se ha repetido cada hashtag. Estos datos se conservan en memoria. Por último, se emite un nuevo flujo que contiene el hash tag y el recuento.

4. El ensamblado **FirstN** se aplica para que devuelva solo los 10 valores más altos, basados en el campo de recuento.

> [!NOTE]
> Para obtener más información sobre cómo trabajar con Trident, consulte el documento [Información general sobre la API de Trident](http://storm.apache.org/releases/current/Trident-API-Overview.html).

### <a name="the-spout"></a>El spout

El spout, **TwitterSpout**, usa [Twitter4j](http://twitter4j.org/en/) para recuperar tweets desde Twitter. Se crea un filtro para las palabras __amor__, **música** y **café**. Los tweets entrantes (estado) que coinciden con el filtro se almacenan en una cola de bloqueo enlazada. Por último, los elementos se sacan de la cola y se emiten a la topología.

### <a name="the-hashtagextractor"></a>El HashtagExtractor

Para extraer hash tags, se usa [getHashtagEntities](http://twitter4j.org/javadoc/twitter4j/EntitySupport.html#getHashtagEntities--) para recuperar todos los hash tags contenidos en el tweet. Después se envían a la secuencia.

## <a name="configure-twitter"></a>Configuración de Twitter

Siga estos pasos para registrar una nueva aplicación de Twitter y obtener la información del token de acceso y de consumidor necesaria para leer desde Twitter:

1. Vaya a [Aplicaciones de Twitter](https://apps.twitter.com) y haga clic en el botón **Crear nueva aplicación**. Al rellenar el formulario, deje el campo **URL de devolución de llamada** vacío.

2. Cuando se cree la aplicación, haga clic en la pestaña **Claves y tokens de acceso** .

3. Copie la información de **Clave de consumidor** y **Secreto de consumidor**.

4. En la parte inferior de la página, seleccione **Crear mi token de acceso** , si no existen tokens. Cuando se hayan creado los tokens, copie la información de **Token de acceso** y **Secreto del token de acceso**.

5. En el proyecto **TwitterSpoutTopology** anteriormente clonado, abra el archivo **resources/twitter4j.properties**, agregue la información recopilada en los pasos anteriores y, por último, guarde el archivo.

## <a name="build-the-topology"></a>Generación de la topología

Use el código siguiente para crear el proyecto:

        cd [directoryname]
        mvn compile

## <a name="test-the-topology"></a>Prueba de la topología

Use el comando siguiente para probar localmente la topología:

    mvn compile exec:java -Dstorm.topology=com.microsoft.example.TwitterTrendingTopology

Después de que se inicie la topología, debería ver la información de depuración que contiene los hash tags y recuentos que ha emitido la topología. El resultado debe ser similar al siguiente texto:

    DEBUG: [Quicktellervalentine, 7]
    DEBUG: [GRAMMYs, 7]
    DEBUG: [AskSam, 7]
    DEBUG: [poppunk, 1]
    DEBUG: [rock, 1]
    DEBUG: [punkrock, 1]
    DEBUG: [band, 1]
    DEBUG: [punk, 1]
    DEBUG: [indonesiapunkrock, 1]

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha probado localmente la topología, descubra cómo implementar la topología: [Implementación y administración de topologías de Apache Storm en HDInsight](hdinsight-storm-deploy-monitor-topology.md).

También se puede interesar en los siguientes temas de Storm:

* [Desarrollo de las topologías de Java para Storm en HDInsight con Maven](hdinsight-storm-develop-java-topology.md)
* [Desarrollo de las topologías de C# para Storm en HDInsight con Visual Studio](hdinsight-storm-develop-csharp-visual-studio-topology.md)

Para obtener más ejemplos de Storm para HDInsight:

* [Topologías de ejemplo para Storm en HDInsight](hdinsight-storm-example-topology.md)


