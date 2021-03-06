---
title: "Bibliotecas de administración de Azure Event Hubs | Microsoft Docs"
description: "Administración de entidades y espacios de nombres de Event Hubs desde .NET"
services: event-hubs
cloud: na
documentationcenter: na
author: jtaubensee
manager: timlt
ms.assetid: 
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 4/10/2017
ms.author: jotaub;sethm
translationtype: Human Translation
ms.sourcegitcommit: 785d3a8920d48e11e80048665e9866f16c514cf7
ms.openlocfilehash: a9023448c4ced1edf54c84bb103454cbd76fbfba
ms.lasthandoff: 04/12/2017


---

# <a name="event-hubs-management-libraries"></a>Bibliotecas de administración de Event Hubs

Las bibliotecas de administración de Event Hubs pueden aprovisionar dinámicamente las entidades y los espacios de nombres de Event Hubs. Esto posibilita los escenarios complejos de implementación y mensajería, lo que permite determinar mediante programación qué entidades aprovisionar. Estas bibliotecas están actualmente disponibles para .NET.

## <a name="supported-functionality"></a>Funcionalidad admitida

* Creación, actualización y eliminación de espacios de nombres
* Creación, actualización y eliminación de Event Hubs
* Creación, actualización y eliminación de grupos de consumidores

## <a name="prerequisites"></a>Requisitos previos

Para comenzar a usar las bibliotecas de administración de Event Hubs, debe autenticarse con Azure Active Directory (AAD). AAD requiere que se autentique como una entidad de servicio que proporciona acceso a los recursos de Azure. Para más información sobre cómo crear una entidad de servicio, consulte uno de los siguientes artículos:  

* [Uso de Azure Portal para crear una aplicación de Active Directory y una entidad de servicio con acceso a los recursos](../azure-resource-manager/resource-group-create-service-principal-portal.md)
* [Uso de Azure PowerShell para crear a una entidad de servicio para acceder a recursos](../azure-resource-manager/resource-group-authenticate-service-principal.md)
* [Uso de la CLI de Azure para crear a una entidad de servicio para acceder a recursos](../azure-resource-manager/resource-group-authenticate-service-principal-cli.md)

Estos tutoriales le proporcionarán valores para `AppId` (id. de cliente), `TenantId` y `ClientSecret` (clave de autenticación), los cuales se usan para la autenticación mediante las bibliotecas de administración. Debe tener permisos de "propietario" en el grupo de recursos en el que desea realizar la ejecución.

## <a name="programming-pattern"></a>Modelo de programación

El patrón para manipular los recursos de Event Hubs sigue un protocolo común:

1. Obtenga un token de Azure Active Directory mediante la biblioteca `Microsoft.IdentityModel.Clients.ActiveDirectory`.
    ```csharp
    var context = new AuthenticationContext($"https://login.windows.net/{tenantId}");

    var result = await context.AcquireTokenAsync(
        "https://management.core.windows.net/",
        new ClientCredential(clientId, clientSecret)
    );
    ```

1. Cree el objeto `EventHubManagementClient`.
    ```csharp
    var creds = new TokenCredentials(token);
    var ehClient = new EventHubManagementClient(creds)
    {
        SubscriptionId = SettingsCache["SubscriptionId"]
    };
    ```

1. Establezca los parámetros de `CreateOrUpdate` en los valores especificados.
    ```csharp
    var ehParams = new EventHubCreateOrUpdateParameters()
    {
        Location = SettingsCache["DataCenterLocation"]
    };
    ```

1. Ejecute la llamada.
    ```csharp
    await ehClient.EventHubs.CreateOrUpdateAsync(resourceGroupName, namespaceName, EventHubName, ehParams);
    ```

## <a name="next-steps"></a>Pasos siguientes
* [Ejemplo de administración de .NET](https://github.com/Azure-Samples/event-hubs-dotnet-management/)
* [Referencia de Microsoft.Azure.Management.EventHub](/dotnet/api/Microsoft.Azure.Management.EventHub) 

