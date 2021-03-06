---
title: "Detección y diagnóstico de problemas de la aplicación web: Azure Application Insights | Microsoft Docs"
description: Analice los bloqueos y detecte y diagnostique problemas de rendimiento en sus aplicaciones.
author: alancameronwills
services: application-insights
documentationcenter: 
manager: carmonm
ms.assetid: 6ccab5d4-34c4-4303-9d3b-a0f1b11e6651
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 03/14/2017
ms.author: awills
translationtype: Human Translation
ms.sourcegitcommit: fd35f1774ffda3d3751a6fa4b6e17f2132274916
ms.openlocfilehash: 05fe4996a8a9c886f2f1b61471dc80550633ecf6
ms.lasthandoff: 03/16/2017


---
# <a name="detect-triage-and-diagnose-with-application-insights"></a>Detección, evaluación de errores y diagnóstico con Application Insights


[Application Insights](app-insights-overview.md) es un servicio de Application Performance Management (APM) extensible para desarrolladores web. Ayuda a averiguar cómo está funcionando su aplicación y cómo se está usando cuando está activa. Y si hay un problema, le permite tener información sobre él y ayuda a evaluar el impacto y a determinar la causa.

A continuación se muestra una cuenta de un equipo que desarrolla aplicaciones web:

* *"Hace un par de días implementamos una revisión 'secundaria'. No ejecutamos un prueba superada amplia, pero por desgracia algunos cambios inesperados se combinaron en la carga, lo que provocó incompatibilidad entre el front-end y el back-end. Inmediatamente, surgieron excepciones de servidor, nuestra alerta se disparó y nos hicimos conscientes de la situación. Con unos cuantos clics en el portal de Application Insights, teníamos suficiente información de las pilas de llamada de excepción para acotar el problema. Inmediatamente revertimos y limitamos el daño. Application Insights ha hecho que esta parte del ciclo de desarrollo sea muy fácil y simplifica la acción".*

Veamos cómo un equipo de desarrollo web típico usa Application Insights para administrar el rendimiento. Seguiremos al equipo de Fabrikam Bank que desarrolla el sistema de banca en línea (OBS).

![Sitio web de un banco de ejemplo](./media/app-insights-detect-triage-diagnose/03-bank.png)

El equipo trabaja según un ciclo de devOps como este:

![Ciclo de DevOps](./media/app-insights-detect-triage-diagnose/00-devcycle.png)

Los requisitos son la fuente del trabajo pendiente de desarrollo (lista de tareas). Trabajan en sprints cortos, que a menudo entregan software que funciona (normalmente en forma de mejoras y extensiones de la aplicación existente). La aplicación activa se actualiza frecuentemente con nuevas características. Mientras está activa, el equipo supervisa su rendimiento y uso con la ayuda de Application Insights. Estos datos de APM se retroalimentan en su trabajo pendiente de desarrollo.

El equipo usa Application Insights para supervisar estrechamente la aplicación web activa para:

* Rendimiento. Desean comprender cómo varían los tiempos de respuesta con el número de solicitudes; cuánto se está usando la CPU, la red, el disco y otros recursos; y dónde están los cuellos de botella.
* Errores. Si hay excepciones o errores de solicitud, o si un contador de rendimiento se sale de su intervalo habitual, el equipo necesita tener conocimiento de ello rápidamente para que pueda llevar a cabo la acción correspondiente.
* Uso. Siempre que se publique una nueva característica, el equipo desea saber en qué medida se usa y si los usuarios tienen dificultades con ella.

Centrémonos en la parte de comentarios del ciclo:

![Detect-Triage-Diagnose](./media/app-insights-detect-triage-diagnose/01-pipe1.png)

## <a name="detect-poor-availability"></a>Detección de disponibilidad insuficiente
Marcela Markova es desarrolladora senior del equipo OBS y es responsable de supervisar el rendimiento en línea. Configura varias [pruebas de disponibilidad](app-insights-monitor-web-app-availability.md):

* Una prueba de una única dirección URL para la página de inicio principal de la aplicación, http://fabrikambank.com/onlinebanking/. Establece los criterios de código HTTP 200 y el texto 'Welcome!'. Si se produce un error en esta prueba, es que sucede algo grave con la red o los servidores o quizás sea un problema de implementación. (O alguien ha cambiado el mensaje de bienvenida sin avisarla).
* Una prueba de varios paso más profunda, que inicia una sesión y obtiene una lista de cuentas actuales, y donde se comprueban algunos detalles importantes en cada página. Esta prueba comprueba que funciona el vínculo a la base de datos de cuentas. Marcela usa un identificador de cliente ficticio: se mantienen algunos de ellos con fines de prueba.

Con estas pruebas preparadas, está segura de que el equipo se enterará rápidamente de cualquier interrupción.  

Los errores se muestran como puntos rojos en el gráfico de pruebas web:

![Presentación de pruebas web que se han ejecutado durante el período anterior](./media/app-insights-detect-triage-diagnose/04-webtests.png)

Pero lo más importante, se enviará por correo electrónico una alerta sobre cualquier error al equipo de desarrollo. De esa manera, estarán informados antes casi que todos los clientes.

## <a name="monitor-performance-metrics"></a>Supervisión de las métricas de rendimiento.
En la misma página de información general de Application Insights, hay un gráfico que muestra diversas [métricas clave](app-insights-web-monitor-performance.md).

![Varias métricas](./media/app-insights-detect-triage-diagnose/05-perfMetrics.png)

El tiempo de carga de la página del explorador se obtiene de la telemetría enviada directamente desde las páginas web. El tiempo de respuesta del servidor, el número de solicitudes del servidor y el número de solicitudes con error se miden en el servidor web y se envían a Application Insights desde allí.

Marcela está un poco preocupada por el gráfico de respuesta del servidor. Dicho gráfico muestra el tiempo promedio que pasa entre la recepción en él de una solicitud HTTP del explorador de un usuario y la devolución de la respuesta. No es infrecuente ver una variación en este gráfico, porque la carga del sistema varía. No obstante, en este caso, parece haber una correlación entre los pequeños aumentos en el recuento de solicitudes y los grandes aumentos en el tiempo de respuesta. Esto podría indicar que el sistema está funcionando al borde de sus límites.

Abre los gráficos de servidores:

![Varias métricas](./media/app-insights-detect-triage-diagnose/06.png)

Parece no haber ninguna señal de limitación de recursos, por lo que es posible que los desplazamientos en los gráficos de respuesta del servidor sean simplemente una coincidencia.

## <a name="alerts"></a>Alertas
A pesar de ello, quiere supervisar los tiempos de respuesta. Si no son demasiado altos, desea saberlo inmediatamente.

Así que establece una [alerta](app-insights-metrics-explorer.md) para conocer los tiempos de respuesta que superen el umbral habitual. Esto le da la seguridad de que obtendrá información si los tiempos de respuesta son lentos.

![Agregar hoja de alertas](./media/app-insights-detect-triage-diagnose/07-alerts.png)

Las alertas se pueden establecer según una amplia variedad de métricas. Por ejemplo, puede recibir mensajes de correo electrónico si el recuento de excepciones se vuelve alto, o si la memoria disponible baja, o si se produce un pico en las solicitudes de cliente.

## <a name="smart-detection-alerts"></a>Alertas de detección inteligente
El día siguiente, llega una alerta de Application Insights por correo electrónico. Pero cuando la abre, se da cuenta de que no es la alerta de tiempo de respuesta que ha configurado. En lugar de eso, le indica que se ha producido un aumento repentino de las solicitudes con error; es decir, las solicitudes que han devuelto 500 códigos de error o más.

Las solicitudes con error indican los casos donde los usuarios han visto un error, normalmente a continuación de una excepción en el código. Quizás verán un mensaje que dice "Lo sentimos, no pudimos actualizar sus datos ahora" o, en el peor de los casos, un volcado de la pila en la pantalla del usuario, cortesía del servidor web.

Esta alerta es una sorpresa, porque la última vez que lo consultó, el recuento de solicitudes con error era bastante bajo. En un servidor ocupado, era de esperar un pequeño número de errores.

También le sorprendió no haber tenido que configurar esta alerta. De hecho, la detección inteligente se incluye automáticamente con Application Insights. Se ajusta de forma automática al patrón de errores habitual de la aplicación y "se adapta a" los errores de una página concreta, con una carga elevada o vinculados a otras métricas. Genera la alarma solo si hay un aumento por encima de lo que se espera.

![correo electrónico de diagnóstico proactivo](./media/app-insights-detect-triage-diagnose/21.png)

Se trata de un correo electrónico muy útil. No solo genera una alarma. También lleva a cabo gran parte de las tareas de diagnóstico y evaluación de errores.

Muestra cuántos clientes se ven afectados y en qué páginas web u operaciones. Marcela puede decidir si necesita implicar a todo el equipo que trabaja en esto como un simulacro de incendio o si se puede ignorar hasta la próxima semana.

El correo electrónico también muestra que se ha producido una excepción determinada y (todavía más interesante) que el error está asociado con las llamadas con error a una base de datos determinada. Esto explica por qué el error ha aparecido repentinamente, aunque el equipo de Marcela no haya implementado recientemente ninguna actualización.

Hace ping al coordinador del equipo de base de datos. Sí, han lanzado una revisión en la última media hora y, por lo que parece, es posible que se haya producido un pequeño cambio en el esquema...

Por tanto, el problema se está resolviendo, incluso antes de investigar los registros y en los 15 minutos posteriores a su aparición. Aun así, Marcela hace clic en el vínculo para abrir Application Insights. Se abre directamente en una solicitud con error y ve la llamada a la base de datos con error en la lista asociada de llamadas de dependencia.

![solicitud con error](./media/app-insights-detect-triage-diagnose/23.png)

## <a name="detecting-exceptions"></a>Detección de excepciones
Con un poco de configuración, se notifican [excepciones](app-insights-asp-net-exceptions.md) automáticamente a Application Insights. También se pueden capturar de manera explícita mediante la inserción de llamadas a [TrackException()](app-insights-api-custom-events-metrics.md#trackexception) en el código:  

    var telemetry = new TelemetryClient();
    ...
    try
    { ...
    }
    catch (Exception ex)
    {
       // Set up some properties:
       var properties = new Dictionary <string, string>
         {{"Game", currentGame.Name}};

       var measurements = new Dictionary <string, double>
         {{"Users", currentGame.Users.Count}};

       // Send the exception telemetry:
       telemetry.TrackException(ex, properties, measurements);
    }


El equipo de Fabrikam Bank ha evolucionado la práctica de enviar siempre telemetría en una excepción, a menos que haya una recuperación obvia.  

De hecho, su estrategia es incluso más amplia que eso: envían telemetría en todos los casos donde el cliente se siente frustrado por lo quería hacer, ya corresponda a una excepción en el código o no. Por ejemplo, si un sistema externo de transferencia interbancaria devuelve un mensaje «no se puede realizar esta transacción» por algún motivo operativo (no es error del cliente), entonces realizan el seguimiento de ese evento.

    var successCode = AttemptTransfer(transferAmount, ...);
    if (successCode < 0)
    {
       var properties = new Dictionary <string, string>
            {{ "Code", returnCode, ... }};
       var measurements = new Dictionary <string, double>
         {{"Value", transferAmount}};
       telemetry.TrackEvent("transfer failed", properties, measurements);
    }

TrackException se usa para informar de excepciones porque envía una copia de la pila. TrackEvent se emplea para informar de otros eventos. Puede vincular cualquier propiedad que podría resultar útil en el diagnóstico.

Las excepciones y los eventos se muestran en la hoja [Diagnostic Search](app-insights-diagnostic-search.md) (Búsqueda de diagnóstico). Puede profundizar en ellos para ver propiedades adicionales y el seguimiento de la pila.

![En la búsqueda de diagnóstico, utilice filtros para mostrar determinados tipos de datos.](./media/app-insights-detect-triage-diagnose/appinsights-333facets.png)

## <a name="monitoring-user-activity"></a>Seguimiento de la actividad de usuario
Cuando el tiempo de respuesta es bueno de manera coherente y hay algunas excepciones, el equipo de desarrollo puede continuar con la usabilidad. Puede pensar en cómo mejorar la experiencia del usuario y en cómo animar a más usuarios a lograr los objetivos deseados.

Por ejemplo, un viaje de usuario típico a través del sitio web tiene un claro "embudo". Muchos clientes examinan las tasas de diferentes tipos de préstamo. Algunos de ellos rellenan el formulario de presupuesto. De aquellos que reciben un presupuesto, algunos siguen adelante y obtienen el préstamo.

![Recuentos de vistas de página](./media/app-insights-detect-triage-diagnose/12-funnel.png)

Teniendo en cuenta dónde abandonan los mayores números de clientes, la empresa puede calcula cómo conseguir que más usuarios pasen a la parte inferior del embudo. En algunos casos se puede producir un error en la experiencia de usuario (UX): por ejemplo, es difícil encontrar el botón "siguiente" o las instrucciones no son evidentes. Lo más probable es que haya motivos empresariales más importantes para los abandonos: quizás las tasas de préstamo son demasiado altas.

Independientemente de los motivos, los datos ayudan al equipo a averiguar qué hacen los usuarios. Se pueden insertar más llamadas de seguimiento para averiguar más detalles. TrackEvent() se puede usar para contar las acciones del usuario, desde los detalles mínimos como los clics de botón individuales hasta logros importantes como la liquidación de un préstamo.

El equipo se está acostumbrando a tener información sobre la actividad del usuario. En la actualidad, cada vez que diseñan una nueva característica, calculan cómo obtendrán comentarios de su uso. Diseñan llamadas de seguimiento en la característica desde el principio. Usan los comentarios para mejorar la característica de cada ciclo de desarrollo.

## <a name="proactive-monitoring"></a>Supervisión proactiva
Marcela no se sienta de brazos cruzados a esperar las alertas. Poco después de cada reimplementación, echa un vistazo a los [tiempos de respuesta](app-insights-web-monitor-performance.md), tanto la cifra general como la tabla de solicitudes más lentas, así como los recuentos de excepciones.  

![Gráfico de tiempo de respuesta y cuadrícula de tiempos de respuesta del servidor.](./media/app-insights-detect-triage-diagnose/09-dependencies.png)

Para evaluar el efecto sobre el rendimiento de cada implementación, normalmente compara cada semana con la última. Si hay un empeoramiento repentino, lo notifica a los desarrolladores pertinentes.

## <a name="triage"></a>Evaluación de errores
Evaluar la gravedad y la extensión de un problema es el primer paso tras la detección. ¿Debemos llamar al equipo a medianoche? ¿O puede dejarse hasta el siguiente hueco conveniente en el trabajo pendiente? Hay algunas cuestiones importantes en la evaluación de errores.

¿Cuánto está sucediendo? Los gráficos de la hoja de información general proporcionan cierta perspectiva sobre un problema. Por ejemplo, la aplicación de Fabrikam generó cuatro alertas de prueba web en una noche. Al examinar el gráfico por la mañana, el equipo pudo ver que había algunos puntos rojos, aunque todavía la mayoría de las pruebas eran verdes. Cuando se profundizó en el gráfico de disponibilidad, se tuvo la certeza de que todos estos problemas intermitentes procedían de una ubicación de prueba. Se trataba obviamente de un problema de red que afectaba a una sola ruta, y es probable que se aclarara por sí solo.  

Por el contrario, un aumento considerable y estable en el gráfico de recuento de excepciones o tiempos de respuesta es claramente algo de lo que tener miedo.

Una táctica de evaluación de errores útil es probarlo usted mismo. Si se tropieza con el mismo problema, sabrá que es real.

¿Qué fracción de usuarios se vieron afectados? Para obtener una respuesta aproximada, divida la tasa de error entre el recuento de sesiones.

![Gráficos de las solicitudes y sesiones con error](./media/app-insights-detect-triage-diagnose/10-failureRate.png)

En el caso de respuesta lenta, compare la tabla de solicitudes de respuesta más lenta con la frecuencia de uso de cada página.

¿Qué importancia tiene el escenario bloqueado? ¿Se trata de un problema funcional que bloquea un caso de usuario determinado?, ¿tiene mucha importancia? Si los clientes no pueden pagar sus facturas, esto es grave; si no pueden cambiar sus preferencias de color de pantalla, quizás pueda esperar. Los detalles del evento o excepción, o la identidad de la página lenta, le indican dónde tienen problemas los clientes.

## <a name="diagnosis"></a>Diagnóstico
El diagnóstico no es exactamente lo mismo que la depuración. Antes de iniciar el seguimiento a través del código, debe tener una idea aproximada de por qué, dónde y cuándo se está produciendo el problema.

**¿Cuándo sucede?** La vista histórica proporcionada por los gráficos de eventos y métricas facilita la creación de una correlación entre los efectos y las posibles causas. Si hay picos intermitentes en las tasas de excepción o de tiempo de respuesta, examine el número de solicitudes: si alcanza el máximo al mismo tiempo, parece un problema de recursos. ¿Es necesario asignar más CPU o memoria? ¿O se trata de una dependencia que no puede administrar la carga?

**¿Somos nosotros?**  Si tiene un descenso repentino en el rendimiento de un tipo concreto de solicitud, por ejemplo cuando el cliente desea un extracto de cuenta, hay una posibilidad de que sea un subsistema externo en lugar de la aplicación web. En el Explorador de métricas, seleccione la tasa de errores de dependencia y las tasas de duración de dependencia y compare sus historiales de las pasadas horas o días con el problema que ha detectado. Si hay cambios que se correlacionan, es posible que un subsistema externo sea el culpable.  

![Gráficos de errores de dependencia y duración de llamadas a dependencias](./media/app-insights-detect-triage-diagnose/11-dependencies.png)

Algunos problemas de dependencia lenta son problemas de ubicación geográfica. Fabrikam Bank usa máquinas virtuales de Azure, y descubrieron que habían ubicado sin darse cuenta el servidor web y el servidor de cuentas en distintos países. Migrando uno de ellos, se obtuvo una mejora considerable.

**¿Qué hicimos?** Si el problema no parece estar en una dependencia, y no ha estado siempre ahí, es probable que se deba a un cambio reciente. La perspectiva histórica proporcionada por los gráficos de métricas y eventos facilita la correlación de cualquier cambio repentino con las implementaciones. De esta forma se limita la búsqueda del problema.

**¿Qué está ocurriendo?** Algunos problemas se producen solo en raras ocasiones y pueden ser difíciles de rastrear mediante pruebas sin conexión. Todo lo que podemos hacer es intentar capturar el error cuando se produzca en vivo. Puede inspeccionar los volcados de pila en los informes de excepciones. Además, puede escribir llamadas de seguimiento con su plataforma de registro favorita o con TrackTrace() o TrackEvent().  

Fabrikam tenía un problema intermitente con las transferencias entre cuentas, pero solo con determinados tipos de cuenta. Para entender mejor lo que estaba ocurriendo, insertaron llamadas a TrackTrace() en los puntos clave del código y asociaron el tipo de cuenta como una propiedad para cada llamada. De esta forma fue fácil filtrar solo esos seguimientos en la búsqueda de diagnóstico. También asociaron valores de parámetros como propiedades y medidas a las llamadas de seguimiento.

## <a name="dealing-with-it"></a>Lidiar con él
Una vez que ha diagnosticado el problema, puede elaborar un plan para corregirlo. Es posible que necesite revertir un cambio reciente, o quizás solo pueda seguir adelante y corregirlo. Una vez que se realiza la corrección, Application Insights le indicará si se ha realizado correctamente.  

El equipo de desarrollo de Fabrikam Bank adopta un enfoque más estructurado para medir el rendimiento de lo que solían hacer antes del uso de Application Insights.

* Establecen objetivos de rendimiento en términos de medidas específicas en la página de información general de Application Insights.
* Diseñan las medidas de rendimiento en la aplicación desde el principio, por ejemplo, las métricas que miden el progreso del usuario a través de 'embudos'.  

## <a name="usage"></a>Uso
Application Insights puede usarse también para saber qué hacen los usuarios con una aplicación. Una vez que funciona sin problemas, al equipo le gustaría saber qué características son las más populares, lo que les gusta a los usuarios o con lo que tienen dificultad y la frecuencia con que vuelven. Eso ayudará a establecer prioridades en su próximo trabajo. Y pueden planear la medición del éxito de cada característica como parte del ciclo de desarrollo. [Más información](app-insights-web-track-usage.md).

## <a name="your-applications"></a>Sus aplicaciones
Así es como un equipo usa Application Insights no solo para solucionar problemas individuales, sino también para mejorar su ciclo de vida de desarrollo. Espero que les haya dado algunas ideas sobre cómo Application Insights puede ayudarle con la administración del rendimiento de sus propias aplicaciones.

## <a name="video"></a>Vídeo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>Pasos siguientes
Puede comenzar a trabajar de varias maneras, según las características de la aplicación. Seleccione lo que más le convenga:

* [Aplicación web ASP.NET](app-insights-asp-net.md)
* [Aplicación web de Java](app-insights-java-get-started.md)
* [Aplicación web de Node.js](app-insights-nodejs.md)
* Aplicaciones ya implementadas hospedadas en [IIS](app-insights-monitor-web-app-availability.md), [J2EE](app-insights-java-live.md) o [Azure](app-insights-azure.md).
* [Páginas web](app-insights-javascript.md): aplicación de una sola página o una página web normal. Use esta característica por su cuenta o además de las opciones de servidor.
* [Pruebas de disponibilidad](app-insights-monitor-web-app-availability.md) para probar la aplicación desde la red Internet pública.

