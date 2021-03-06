---
title: "Información sobre la protección del acceso a los datos de DocumentDB | Microsoft Docs"
description: "Obtenga información sobre los conceptos de control de acceso en DocumentDB, incluidas las claves maestras, las claves de solo lectura, los usuarios y los permisos."
services: documentdb
author: mimig1
manager: jhubbard
editor: monicar
documentationcenter: 
ms.assetid: 8641225d-e839-4ba6-a6fd-d6314ae3a51c
ms.service: documentdb
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/23/2017
ms.author: mimig
ms.translationtype: Human Translation
ms.sourcegitcommit: 8f291186c6a68dea8aa00b846a2e6f3ad0d7996c
ms.openlocfilehash: 7ab474747c74295a5dba9a6f3ad32000a7551a9c
ms.contentlocale: es-es
ms.lasthandoff: 04/28/2017


---
# <a name="securing-access-to-documentdb-data"></a>Protección del acceso a los datos de DocumentDB
En este artículo se proporciona información general sobre la protección del acceso a los datos almacenados en [DocumentDB de Microsoft Azure](https://azure.microsoft.com/services/documentdb/).

DocumentDB usa dos tipos de claves para autenticar usuarios y proporcionar acceso a sus datos y recursos. 

|Tipo de clave|Recursos|
|---|---|
|[Claves maestras](#master-keys) |Se utilizan para los recursos administrativos: cuentas de base de datos, bases de datos, usuarios y permisos|
|[Tokens de recursos](#resource-tokens)|Se usan para recursos de aplicaciones: colecciones, documentos, datos adjuntos, procedimientos almacenados, desencadenadores y UDF|

<a id="master-keys"></a>

## <a name="master-keys"></a>Claves maestras 

Las claves maestras proporcionan acceso a todos los recursos administrativos de la cuenta de base de datos. Claves maestras:  
- Proporcionan acceso a cuentas, bases de datos, usuarios y permisos. 
- No se pueden usar para proporcionar acceso granular a colecciones y documentos.
- Se crean durante la creación de una cuenta.
- Se pueden regenerar en cualquier momento.

Cada cuenta consta de dos claves maestras: una principal y una secundaria. El fin de las claves dobles es que pueda regenerar, o distribuir claves, lo que proporciona un acceso continuo a su cuenta y sus datos. 

Además de las dos claves maestras de la cuenta de DocumentDB, hay dos claves de solo lectura. Estas claves de solo lectura permiten operaciones de lectura en la cuenta. Las claves de solo lectura no proporcionan acceso a los recursos de los permisos de lectura.

Las claves maestras principal, secundaria, de solo lectura y de lectura y escritura se pueden recuperar y volver a generar desde Azure Portal. Para ver instrucciones al respecto, consulte [Visualización, copia y regeneración de las claves de acceso](documentdb-manage-account.md#a-idkeysaview-copy-and-regenerate-access-keys).

![Control de acceso (IAM) en Azure Portal: demostración de la seguridad de bases de datos NoSQL](./media/documentdb-secure-access-to-data/nosql-database-security-master-key-portal.png)

El proceso de rotación de la clave maestra es simple. Navegue hasta Azure Portal para recuperar la clave secundaria, reemplace la clave principal por la clave secundaria en la aplicación y, finalmente, rote la clave principal en Azure Portal.

![Rotación de la clave maestra en Azure Portal: demostración de la seguridad de bases de datos NoSQL](./media/documentdb-secure-access-to-data/nosql-database-security-master-key-rotate-workflow.png)

### <a name="code-sample-to-use-a-master-key"></a>Ejemplo de código para usar una clave maestra

El fragmento de código de ejemplo ilustra cómo usar un punto de conexión y la clave maestra de la cuenta de DocumentDB para crear una instancia de DocumentClient y una base de datos. 

```csharp
//Read the DocumentDB endpointUrl and authorization keys from config.
//These values are available from the Azure portal on the NOSQL (DocumentDB) account blade under "Keys".
//NB > Keep these values in a safe and secure location. Together they provide Administrative access to your DocDB account.

private static readonly string endpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
private static readonly SecureString authorizationKey = ToSecureString(ConfigurationManager.AppSettings["AuthorizationKey"]);

client = new DocumentClient(new Uri(endpointUrl), authorizationKey);

// Create Database
Database database = await client.CreateDatabaseAsync(
    new Database
    {
        Id = databaseName
    });
```

<a id="resource-tokens"></a>

## <a name="resource-tokens"></a>Tokens de recursos

Los tokens de recursos proporcionan acceso a los recursos de la aplicación en una base de datos. Los tokens de recursos:
- Proporcionan acceso a colecciones, claves de partición, documentos, datos adjuntos, procedimientos almacenados, desencadenadores y UDF concretos.
- Se crean cuando a un [usuario](#users) se le conceden [permisos](#permissions) para un recurso concreto.
- Se vuelven a crear cuando una llamada POST, GET o PUT aplica una acción a un recurso de permiso.
- Usan un token de recurso de hash construido específicamente para el usuario, el recurso y el permiso.
- Está limitado por un período de validez personalizable. El intervalo de tiempo válido predeterminado es una hora. Sin embargo, la vigencia del token puede especificarse explícitamente hasta un máximo de cinco horas.
- Proporcionan una alternativa segura a proporcionar la clave maestra. 
- Permiten que los clientes lean, escriban y eliminen recursos de la cuenta de DocumentDB en función de los permisos que se les haya otorgado.

Si desea proporcionar acceso a los recursos de su cuenta de DocumentDB a un cliente que no es de confianza con la clave maestra, puede usar un token de recurso (mediante la creación de usuarios y permisos de DocumentDB).  

Los tokens de recurso de DocumentDB ofrecen una alternativa segura que permite a los clientes leer, escribir y eliminar recursos de la cuenta de DocumentDB en función de los permisos que se les hayan concedido y sin necesidad de una clave maestra o de solo lectura.

Este es un patrón de diseño típico para solicitar, generar y entregar tokens de recursos a los clientes:

1. Se configura un servicio de nivel intermedio para que una aplicación móvil pueda usarlo para compartir fotos del usuario. 
2. Dicho servicio tiene la clave maestra de la cuenta de DocumentDB.
3. La aplicación fotográfica se instala en los dispositivos móviles de los usuarios finales. 
4. Al iniciar sesión, dicha aplicación establece la identidad del usuario con el servicio de nivel intermedio. Este mecanismo de establecimiento de identidad depende únicamente de la aplicación.
5. Una vez establecida la identidad, el servicio de nivel intermedio solicita permisos en función de esta.
6. El servicio de nivel intermedio reenvía un token de recurso a la aplicación de teléfono.
7. La aplicación de teléfono puede seguir usando el token de recurso para obtener acceso directo a los recursos de DocumentDB con los permisos definidos por el token de recurso y para el intervalo permitido por dicho token. 
8. Cuando el token de recurso expira, las solicitudes posteriores reciben un mensaje 401 de excepción no autorizada.  En este punto, la aplicación de teléfono restablece la identidad y solicita un nuevo token de recurso.

    ![Flujo de trabajo de tokens de recursos de DocumentDB](./media/documentdb-secure-access-to-data/resourcekeyworkflow.png)

La generación y administración de los tokens de recursos las controlan las bibliotecas de cliente de DocumentDB nativas; sin embargo, si se utiliza REST, debe construir los encabezados de solicitud o autenticación. Para más información acerca de cómo crear encabezados de autenticación para REST, consulte [Access Control on DocumentDB Resources](https://docs.microsoft.com/en-us/rest/api/documentdb/access-control-on-documentdb-resources) (Control de acceso en recursos de DocumentDB) o [el código fuente de nuestros SDK](https://github.com/Azure/azure-documentdb-node/blob/master/source/lib/auth.js).

Para ver un ejemplo de un servicio de nivel intermedio que se usa para generarlo o los tokens de recursos del agente, consulte la [aplicación ResourceTokenBroker](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin/UserItems/ResourceTokenBroker/ResourceTokenBroker/Controllers).

<a id="users"></a>

## <a name="users"></a>Usuarios
Los usuarios de DocumentDB se asocian a una base de datos de este tipo.  Cada base de datos puede contener cero, o más, usuarios de DocumentDB.  El siguiente código de ejemplo muestra cómo crear un recurso de usuario de DocumentDB.

```csharp
//Create a user.
User docUser = new User
{
    Id = "mobileuser"
};

docUser = await client.CreateUserAsync(UriFactory.CreateDatabaseUri("db"), docUser);
```

> [!NOTE]
> Cada usuario de DocumentDB tiene una propiedad PermissionsLink que se puede usar para recuperar la lista de [permisos](#permissions) asociados al usuario.
> 
> 

<a id="permissions"></a>

## <a name="permissions"></a>Permisos
Un recurso de permiso de DocumentDB se asocia a un usuario de dicha base de datos.  Cada usuario puede contener cero o más permisos de DocumentDB.  Un recurso de permiso proporciona acceso a un token de seguridad que el usuario necesita al intentar obtener acceso a un recurso de aplicación específico.
Hay dos niveles de acceso disponibles que puede proporcionar un recurso de permiso:

* Todo: el usuario tiene pleno permiso en el recurso.
* Lectura: el usuario solo puede leer el contenido del recurso, pero no realizar operaciones de escritura, actualización o eliminación en este.

> [!NOTE]
> Para ejecutar procedimientos almacenados de DocumentDB, el usuario debe tener el permiso Todo en la colección donde se va a ejecutar el procedimiento almacenado.
> 
> 

### <a name="code-sample-to-create-permission"></a>Ejemplo de código para crear permisos

El código de ejemplo siguiente muestra cómo crear un recurso de permisos, leer el token de dicho recurso y asociar los permisos al [usuario](#users) creado anteriormente.

```csharp
// Create a permission.
Permission docPermission = new Permission
{
    PermissionMode = PermissionMode.Read,
    ResourceLink = documentCollection.SelfLink,
    Id = "readperm"
};
  
docPermission = await client.CreatePermissionAsync(UriFactory.CreateUserUri("db", "user"), docPermission);
Console.WriteLine(docPermission.Id + " has token of: " + docPermission.Token);
```

Si ha especificado una clave de partición para la colección y, después, el permiso para los recursos de la colección, de los documentos y de los datos adjuntos también debe incluir la clave ResourcePartitionKey, además de ResourceLink.

### <a name="code-sample-to-read-permissions-for-user"></a>Ejemplo de código para los permisos de lectura del usuario

Para obtener fácilmente todos los recursos de permisos asociados a un usuario concreto, DocumentDB dispone de una fuente de permisos para cada objeto de usuario.  El fragmento de código siguiente muestra cómo recuperar el permiso asociado al usuario creado anteriormente, construir una lista de permisos y crear instancias de un nuevo elemento DocumentClient en nombre del usuario.

```csharp
//Read a permission feed.
FeedResponse<Permission> permFeed = await client.ReadPermissionFeedAsync(
  UriFactory.CreateUserUri("db", "myUser"));
 List<Permission> permList = new List<Permission>();

foreach (Permission perm in permFeed)
{
    permList.Add(perm);
}

DocumentClient userClient = new DocumentClient(new Uri(endpointUrl), permList);
```

## <a name="next-steps"></a>Pasos siguientes
* Para más información acerca de la seguridad de las bases de datos de DocumentDB, consulte [DocumentDB: NoSQL database security](documentdb-nosql-database-security.md) (DocumentDB: seguridad de bases de datos NoSQL).
* Para obtener información acerca de cómo administrar las claves maestra y de solo lectura, consulte [Administración de una cuenta de DocumentDB](documentdb-manage-account.md#a-idkeysaview-copy-and-regenerate-access-keys).
* Para aprender a construir tokens de autorización de DocumentDB, consulte [Access Control on DocumentDB Resources](https://docs.microsoft.com/en-us/rest/api/documentdb/access-control-on-documentdb-resources) (Control de acceso en recursos de DocumentDB).

