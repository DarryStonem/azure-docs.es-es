---
title: Uso de Pig en Hadoop con PowerShell en HDInsight | Microsoft Docs
description: "Aprenda a enviar trabajos de Pig a un clúster de Hadoop en HDInsight mediante Azure PowerShell."
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 737089c1-b494-4387-9def-7b4dac3be532
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 03/21/2017
ms.author: larryfr
ms.custom: H1Hack27Feb2017,hdinsightactive
translationtype: Human Translation
ms.sourcegitcommit: 785d3a8920d48e11e80048665e9866f16c514cf7
ms.openlocfilehash: dcc248d398f2e2aa0f563a0868e80b1e7ee6a3fb
ms.lasthandoff: 04/12/2017

---
# <a name="use-azure-powershell-to-run-pig-jobs-with-hdinsight"></a>Uso de Azure PowerShell para ejecutar trabajos de Pig con HDInsight

[!INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

Este documento proporciona un ejemplo de uso de Azure PowerShell para enviar trabajos de Pig a un clúster de Hadoop en HDInsight. Pig permite escribir trabajos de MapReduce mediante un lenguaje (Pig Latin) que modela las transformaciones de datos, en lugar de asignar y reducir las funciones.

> [!NOTE]
> Este documento no ofrece una descripción detallada de cómo funcionan las instrucciones de Pig Latin que se usan en los ejemplos. Para obtener información sobre Pig Latin utilizado en este ejemplo, consulte [Uso de Hive con Hadoop en HDInsight](hdinsight-use-pig.md).

## <a id="prereq"></a>Requisitos previos

* **Un clúster de Azure HDInsight**.

  > [!IMPORTANT]
  > Linux es el único sistema operativo que se usa en la versión 3.4 de HDInsight, o en las superiores. Para más información, consulte [El contrato de nivel de servicio para las versiones de clúster de HDInsight](hdinsight-component-versioning.md#hdi-version-33-nearing-deprecation-date).

* **Una estación de trabajo con Azure PowerShell**.

[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

## <a id="powershell"></a>Ejecución de trabajos de Pig mediante PowerShell

Azure PowerShell proporciona *cmdlets* que le permiten ejecutar de manera remota trabajos de Pig en HDInsight. Internamente, PowerShell realiza llamadas de REST a la instancia de [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) que se ejecuta en el clúster de HDInsight.

Los cmdlets siguientes se utilizan cuando se ejecutan trabajos de Pig en un clúster de HDInsight remoto:

* **Login-AzureRmAccount**: autentica Azure PowerShell en la suscripción de Azure.
* **New-AzureRmHDInsightPigJobDefinition**: crea una *definición de trabajo* con las instrucciones de Pig Latin especificadas.
* **Start-AzureRmHDInsightJob**: envía la definición del trabajo a HDInsight, inicia el trabajo y devuelve un objeto de *trabajo* que se puede usar para comprobar el estado del trabajo.
* **Wait-AzureRmHDInsightJob**: usa el objeto de trabajo para comprobar el estado del trabajo. Esperará hasta que el trabajo se haya completado o haya superado el tiempo de espera.
* **Get-AzureRmHDInsightJobOutput**: se usa para recuperar la salida del trabajo.

Los pasos siguientes muestran cómo usar estos cmdlets para ejecutar un trabajo en el clúster de HDInsight.

1. Mediante un editor, guarde el código siguiente como **pigjob.ps1**.

    [!code-powershell[main](../../powershell_scripts/hdinsight/use-pig/use-pig.ps1?range=5-51)]

1. Abra un nuevo símbolo del sistema de Windows PowerShell. Cambie los directorios a la ubicación del archivo **pigjob.ps1** y, a continuación, utilice el siguiente comando para ejecutar el script:

        .\pigjob.ps1

    Se le indica que inicie sesión en su suscripción de Azure. Luego, se le pedirá que proporcione el nombre de cuenta y la contraseña de HTTPS/Admin para el clúster de HDInsight.

2. Cuando el trabajo se complete, debe devolver información de manera similar al siguiente texto:

        Start the Pig job ...
        Wait for the Pig job to complete ...
        Display the standard output ...
        (TRACE,816)
        (DEBUG,434)
        (INFO,96)
        (WARN,11)
        (ERROR,6)
        (FATAL,2)

## <a id="troubleshooting"></a>Solución de problemas

Si no se devuelve ninguna información cuando se completa el trabajo, pudo haberse producido un error durante el procesamiento. Para ver la información de error para este trabajo, agregue lo siguiente al final del archivo **pigjob.ps1** , guárdelo y, a continuación, ejecútelo de nuevo.

    # Print the output of the Pig job.
    Write-Host "Display the standard error output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $pigJob.JobId `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

Se devuelve la información escrita en STDERR en el servidor cuando ejecute el trabajo y puede ayudar a determinar por qué se ha producido el error en el trabajo.

## <a id="summary"></a>Resumen
Como puede ver, Azure PowerShell proporciona una manera fácil de ejecutar trabajos de Pig en un clúster de HDInsight, de supervisar el estado del trabajo y de recuperar el resultado.

## <a id="nextsteps"></a>Pasos siguientes
Para obtener información general sobre Pig en HDInsight, siga estos pasos:

* [Uso de Pig con Hadoop en HDInsight](hdinsight-use-pig.md)

Para obtener información sobre otras maneras de trabajar con Hadoop en HDInsight:

* [Uso de Hive con Hadoop en HDInsight](hdinsight-use-hive.md)
* [Uso de MapReduce con Hadoop en HDInsight](hdinsight-use-mapreduce.md)

