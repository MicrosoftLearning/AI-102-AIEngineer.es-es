---
lab:
  title: Supervisión de Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: e0e0042421a4f7150fc3b95cef80887c81a78f3a
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801339"
---
# <a name="monitor-cognitive-services"></a>Supervisión de Cognitive Services

Azure Cognitive Services puede ser una parte fundamental de una infraestructura general de aplicaciones. Es importante poder supervisar la actividad y recibir alertas sobre problemas que pueden necesitar atención.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="provision-a-cognitive-services-resource"></a>Aprovisionamiento de un recurso de Cognitive Services

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso de **Cognitive Services**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **+Crear un recurso**, busque *Cognitive Services* y cree un recurso de **Cognitive Services** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
3. Active las casillas necesarias y cree el recurso.
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Keys and Endpoint** (Claves y punto de conexión). Anote el URI del punto de conexión; lo necesitará más adelante.

## <a name="configure-an-alert"></a>Configuración de una alerta

Vamos a empezar la supervisión mediante la definición de una regla de alertas para que pueda detectar actividad en el recurso de Cognitive Services.

1. En Azure Portal, vaya al recurso de Cognitive Services y consulte su página **Alertas** (en la sección **Supervisión**).
2. Seleccione **Nueva regla de alertas**.
3. En la página **Crear regla de alertas**, en **Ámbito**, compruebe que aparece el recurso de Cognitive Services.
4. En **Condición**, haga clic en **Agregar condición** y vea el panel **Seleccionar una señal** que aparece a la derecha, donde puede seleccionar un tipo de señal para supervisar.
5. En la lista **Tipo de señal**, seleccione **Registro de actividad** y, a continuación, en a lista filtrada, seleccione **Enumerar claves**.
6. Revise la actividad de las últimas 6 horas.
7. Seleccione la pestaña **Acciones**. Tenga en cuenta que puede especificar un *grupo de acciones*. Esto le permite configurar acciones automatizadas cuando se desencadena una alerta (por ejemplo, enviar una notificación por correo electrónico). No lo haremos en este ejercicio; pero puede ser útil en un entorno de producción.
8. En la pestaña **Detalles**, establezca el **nombre de la regla de alerta** en **Alerta de lista de claves**.
9. Seleccione **Revisar + crear**. 
10. Revise la configuración de la alerta. Seleccione **Crear** y espere a que se cree la regla de alerta.
11. En Visual Studio Code, haga clic con el botón derecho en la carpeta **03-monitor** y abra un terminal integrado. A continuación, escriba el siguiente comando para iniciar sesión en su suscripción de Azure mediante la CLI de Azure.

    ```
    az login
    ```

    Se abrirá una pestaña del explorador web que le solicitará que inicie sesión en Azure. Hágalo y, a continuación, cierre la pestaña del explorador y vuelva a Visual Studio Code.

    > **Sugerencia**: Si tiene varias suscripciones, deberá asegurarse de que está trabajando en la que contiene el recurso de Cognitive Services.  Use este comando para determinar la suscripción actual.
    >
    > ```
    > az account show
    > ```
    >
    > Si necesita cambiar la suscripción, ejecute este comando y cambie *&lt;subscriptionName&gt;* por el nombre de suscripción correcto.
    >
    > ```
    > az account set --subscription <subscriptionName>
    > ```

10. Ahora puede usar el comando siguiente para obtener la lista de claves de Cognitive Services, reemplazando *&lt;resourceName&gt;* por el nombre del recurso de Cognitive Services y *&lt; resourceGroup&gt;* por el nombre del grupo de recursos en el que lo creó.

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

El comando devuelve una lista de las claves del recurso de Cognitive Services.

11. Vuelva al explorador con Azure Portal y actualice la **página Alertas**. Debería ver una alerta de **Gravedad 4** en la tabla (si no es así, espere hasta cinco minutos y vuelva a actualizar).
12. Seleccione una alerta para ver sus detalles.

## <a name="visualize-a-metric"></a>Visualización de una métrica

Además de definir alertas, puede ver las métricas del recurso de Cognitive Services para supervisar su uso.

1. En Azure Portal, en la página del recurso de Cognitive Services, seleccione **Métricas** (en la sección **Supervisión**).
2. Si no hay ningún gráfico existente, seleccione **+ Nuevo gráfico**. A continuación, en la lista **Métrica**, revise las métricas posibles que puede visualizar y seleccione **Llamadas totales**.
3. En la lista **Agregación**, seleccione **Recuento**.  Esto le permitirá supervisar el número total de llamadas al recurso de Cognitive Services, lo que resulta útil para determinar cuánto se utiliza el servicio durante un período de tiempo.
4. Para generar algunas solicitudes para Cognitive Service, usará **curl**, una herramienta de línea de comandos para las solicitudes HTTP. En Visual Studio Code, en la carpeta **03-monitor**, abra **rest-test.cmd** y edite el comando **curl** que contiene (que se muestra a continuación), reemplazando *&lt;yourEndpoint&gt;* y *&lt;yourKey&gt;* por el URI del punto de conexión y la clave **Key1** para usar la API de Text Analytics en el recurso de Cognitive Services.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.1/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':           [{'id':1,'text':'hello'}]}"
    ```

5. Guarde los cambios y, a continuación, en el terminal integrado de la carpeta **03-monitor**, ejecute el siguiente comando:

    ```
    rest-test
    ```

El comando devuelve un documento JSON que contiene información sobre el idioma detectado en los datos de entrada (que debe ser inglés).

6. Vuelva a ejecutar el comando **rest-test** varias veces para generar alguna actividad de llamada (puede usar la clave **^** para recorrer en ciclos los comandos anteriores).
7. Vuelva a la página **Métricas** en Azure Portal y actualice el gráfico de recuento **Llamadas totales**. Las llamadas realizadas con *curl* pueden tardar unos minutos en reflejarse en el gráfico; siga actualizando el gráfico hasta que se actualice para incluirlas.

## <a name="more-information"></a>Más información

Una de las opciones para supervisar Cognitive Services es usar el *registro de diagnóstico*. Una vez habilitado, el registro de diagnóstico captura abundante información sobre el uso de recursos de Cognitive Services y puede ser una herramienta útil de supervisión y depuración. Puede tardar más de una hora después de configurar el registro de diagnóstico para generar alguna información, motivo por el que no se ha explorado en este ejercicio; pero puede obtener más información al respecto en la [documentación de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/diagnostic-logging).
