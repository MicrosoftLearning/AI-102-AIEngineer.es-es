---
lab:
  title: Creación de una aplicación cliente de reconocimiento del lenguaje conversacional (vista preliminar)
ms.openlocfilehash: 486a6716a189156739e9107824e561f7bb8fa95c
ms.sourcegitcommit: 2c92ea1491cac72a4765755cf2bc6d8b5f657a46
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2022
ms.locfileid: "141580029"
---
# <a name="create-a-language-service-client-application"></a>Creación de una aplicación cliente de Language Service

La característica Reconocimiento del lenguaje conversacional de Azure Cognitive Service para lenguaje le permite definir un modelo de lenguaje conversacional que las aplicaciones cliente pueden usar para interpretar entrada de lenguaje natural de los usuarios, predecir la *intención* de los usuarios (lo que quieren lograr) e identificar *entidades* a las que debería aplicarse la intención. Puede crear aplicaciones cliente que consuman modelos de reconocimiento del lenguaje conversacional directamente a través de interfaces de REST o mediante kits de desarrollo de software (SDK) específicos del lenguaje.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

1. Inicie Visual Studio Code.

2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).

3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.

4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="create-language-service-resources"></a>Creación de recursos de Language Service

Si ya tiene un recurso de Language Service en su suscripción de Azure, puede usarlo en este ejercicio. De lo contrario, siga estas instrucciones para crear uno.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.

2. Seleccione el botón **&#65291;Crear un recurso**, busque *servicio de lenguaje* y cree un recurso de **Language Service** con los valores siguientes:

    - **Características predeterminadas**: Todo
    - **Características personalizadas**: ninguna
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *westus2*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: *S*
    - **Términos legales**: *active la casilla para confirmar*
    - **Aviso de IA responsable**: *active la casilla para confirmar*

3. Espere a que se creen los recursos. Para ver el recurso, vaya al grupo de recursos en el que lo creó.

## <a name="import-train-and-publish-a-conversational-language-understanding-model"></a>Importación, entrenamiento y publicación de un modelo de reconocimiento del lenguaje conversacional

Si ya tiene un proyecto **Reloj** de un ejercicio o laboratorio anterior, puede usarlo en este ejercicio. De lo contrario, siga estas instrucciones para crearla.

1. En una nueva pestaña del explorador, abra el portal de vista previa de Language Studio en `https://language.cognitive.azure.com`.

2. Inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure. Si es la primera vez que inicia sesión en el portal de Language Service, deberá conceder permisos a la aplicación para acceder a los detalles de su cuenta. A continuación, complete los pasos de *bienvenida* seleccionando la suscripción de Azure y el recurso de creación que acaba de crear.

3. Abra la página **Reconocimiento del lenguaje conversacional**.

3. Junto a **&#65291;Crear nuevo proyecto**, vea la lista desplegable y seleccione **Importar**. Haga clic en **Elegir archivo** y vaya a la subcarpeta **10b-clu-client-(preview)** en la carpeta del proyecto que contiene los archivos de laboratorio de este ejercicio. Seleccione **Reloj.json**, haga clic en **Abrir** y, a continuación, haga clic en **Listo**.

4. Si aparece un panel con consejos para crear una aplicación de Language Service efectiva, ciérrelo.

5. A la izquierda del portal de Language Studio, seleccione **Entrenar modelo** para entrenar la aplicación. Haga clic en **Iniciar un trabajo de entrenamiento**, asigne el nombre **Reloj** al modelo y asegúrese de que la evaluación con entrenamiento esté habilitada. El entrenamiento puede tardar varios minutos en completarse.

    > **Nota**: Dado que el nombre del modelo **Reloj** está codificado en el código clock-client (usado más adelante en el laboratorio), escriba en mayúsculas el nombre exactamente como se describe. 

6. A la izquierda del portal de Language Studio, seleccione **Implementar modelo** y use **Agregar implementación** para crear la implementación del modelo Reloj con el nombre **producción**.

    > **Nota**: Dado que el nombre de implementación **producción** está codificado en el código clock-client (usado más adelante en el laboratorio), escriba en mayúsculas el nombre exactamente como se describe. 

7. Una vez completada la implementación, seleccione la implementación **producción** y, a continuación, haga clic en **Obtener dirección URL de predicción**. Las aplicaciones cliente necesitan la dirección URL del punto de conexión para usar el modelo implementado.

8. A la izquierda del portal de Language Studio, seleccione **Configuración del proyecto** y anote la **clave principal**. Las aplicaciones cliente necesitan la clave principal para usar el modelo implementado.

9. Las aplicaciones cliente necesitan información de la dirección URL de predicción y la clave de Language Service para conectarse al modelo implementado y poder autenticarse.

## <a name="prepare-to-use-the-language-service-sdk"></a>Preparación para usar el SDK de Language Service

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el modelo Reloj (el modelo de reconocimiento del lenguaje conversacional publicado) para predecir las intenciones de la entrada del usuario y responder adecuadamente.

> **Nota**: Puede elegir usar el SDK para **.NET** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **10b-clu-client-(preview)** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.

2. Haga clic con el botón derecho en la carpeta **clock-client** y seleccione **Abrir en el terminal integrado**. A continuación, instale el paquete SDK de Conversational Language Service mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-language-conversations --pre
    python -m pip install python-dotenv
    python -m pip install python-dateutil
    ```

3. Consulte el contenido de la carpeta **clock-client** y fíjese en que contiene un archivo para los valores de configuración:

    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para incluir la **dirección URL del punto de conexión** y la **clave principal** del recurso de Language. Puede encontrar los valores necesarios en Azure Portal o Language Studio de la siguiente manera:

    - Azure Portal: Abra su recurso de Language. En **Administración de recursos**, seleccione **Claves y puntos de conexión**. Copie los valores **CLAVE 1** y **Punto de conexión** en el archivo de configuración.
    - Language Studio: Abra el proyecto **Reloj**. El punto de conexión de Language Service está disponible en la página **Implementar modelo**, en **Obtener dirección URL de predicción**, y la **clave principal** está disponible en la página **Configuración del proyecto**. La parte del punto de conexión de Language Service de la dirección URL de predicción termina con **.cognitiveservices.azure.com/** . Por ejemplo: `https://ai102-langserv.cognitiveservices.azure.com/`.

4. Tenga en cuenta que la carpeta **clock-client** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: clock-client.py

    Abra el archivo de código y, en la parte superior, bajo las referencias existentes al espacio de nombres, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Language Service:

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    from azure.ai.language.conversations.models import ConversationAnalysisOptions
    ```

## <a name="get-a-prediction-from-the-conversational-language-model"></a>Obtención de una predicción y prueba del modelo de lenguaje conversacional

Ahora está listo para implementar código que usa el SDK para obtener una predicción del modelo de lenguaje conversacional.

1. En la función **Main**, fíjese en que ya se ha proporcionado código para cargar el punto de conexión de predicción y la clave del archivo de configuración. A continuación, busque el comentario **Crear un cliente para el modelo de Language Service** y agregue el código siguiente para crear un cliente de predicción para la aplicación de Language Service:

    **C#**

    ```C#
    // Create a client for the Language service model
    Uri lsEndpoint = new Uri(predictionEndpoint);
    AzureKeyCredential lsCredential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient lsClient = new ConversationAnalysisClient(lsEndpoint, lsCredential);
    ```

    **Python**

    ```Python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

2. Tenga en cuenta que el código de la función **Main** solicita entrada del usuario hasta que ese escribe "quit". Dentro de este bucle, busque el comentario **Llamada al modelo de Language Service para obtener la intención y las entidades** y agregue el código siguiente:

    **C#**

    ```C#
    // Call the Language service model to get intent and entities
    var lsProject = "Clock";
    var lsSlot = "production";

    ConversationsProject conversationsProject = new ConversationsProject(lsProject, lsSlot);
    Response<AnalyzeConversationResult> response = await lsClient.AnalyzeConversationAsync(userText, conversationsProject);

    ConversationPrediction conversationPrediction = response.Value.Prediction as ConversationPrediction;

    Console.WriteLine(JsonConvert.SerializeObject(conversationPrediction, Formatting.Indented));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].Confidence > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }

    var entities = conversationPrediction.Entities;
    ```

    **Python**

    ```Python
    # Call the Language service model to get intent and entities
    convInput = ConversationAnalysisOptions(
        query = userText
        )
    
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        result = client.analyze_conversations(
            convInput, 
            project_name=cls_project, 
            deployment_name=deployment_slot
            )

    # list the prediction results
    top_intent = result.prediction.top_intent
    entities = result.prediction.entities

    print("view top intent:")
    print("\ttop intent: {}".format(result.prediction.top_intent))
    print("\tcategory: {}".format(result.prediction.intents[0].category))
    print("\tconfidence score: {}\n".format(result.prediction.intents[0].confidence_score))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity.category))
        print("\ttext: {}".format(entity.text))
        print("\tconfidence score: {}".format(entity.confidence_score))

    print("query: {}".format(result.query))
    ```

    La llamada al modelo de Language Service devuelve una predicción o un resultado, que incluye la intención superior (la más probable), así como cualquier entidad detectada en la expresión de entrada. La aplicación cliente debe usar ahora esa predicción para determinar y realizar la acción adecuada.

3. Busque el comentario **Apply the appropriate action** (Aplicar la acción adecuada) y agregue el código siguiente, que comprueba las intenciones admitidas por la aplicación (**GetTime**,**GetDate** y **GetDay**) y determina si se ha detectado alguna entidad pertinente, antes de llamar a una función existente para generar una respuesta adecuada.

    **C#**

    ```C#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a location entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Location")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        location = entity.Text;
                    }
                    
                }

            }

            // Get the time for the specified location
            var getTimeTask = Task.Run(() => GetTime(location));
            string timeResponse = await getTimeTask;
            Console.WriteLine(timeResponse);
            break;

        case "GetDay":
            var date = DateTime.Today.ToShortDateString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Date entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Date")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        date = entity.Text;
                    }
                }
            }
            // Get the day for the specified date
            var getDayTask = Task.Run(() => GetDay(date));
            string dayResponse = await getDayTask;
            Console.WriteLine(dayResponse);
            break;

        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Weekday entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Weekday")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        day = entity.Text;
                    }
                }
                
            }
            // Get the date for the specified day
            var getDateTask = Task.Run(() => GetDate(day));
            string dateResponse = await getDateTask;
            Console.WriteLine(dateResponse);
            break;

        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**

    ```Python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity.category:
                    # ML entities are strings, get the first one
                    location = entity.text
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity.category:
                    # Regex entities are strings, get the first one
                    date_string = entity.text
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity.category:
                # List entities are lists
                    day = entity.text
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

4. Guarde los cambios y vuelva al terminal integrado de la carpeta **clock-client** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python clock-client.py
    ```

5. Cuando se le solicite, escriba las expresiones para probar la aplicación. Por ejemplo, pruebe lo siguiente:

    *Hello*
    
    *¿Qué hora tenemos?*

    *¿Qué hora es en Londres?*

    *¿Qué fecha tenemos?*

    *¿Qué fecha es el domingo?*

    *¿Qué día es?*

    *¿Qué día es 01/01/2025?*

> **Nota**: La lógica de la aplicación es deliberadamente sencilla y tiene una serie de limitaciones. Por ejemplo, al obtener la hora, solo se admite un conjunto restringido de ciudades y se omite el horario de verano. El objetivo es ver un ejemplo de un patrón típico para usar Language Service en el que la aplicación debe:
>
>   1. Conectarse a un punto de conexión de la predicción.
>   2. Enviar una expresión para obtener una predicción.
>   3. Implementar la lógica para responder correctamente a la intención y las entidades predichos.

6. Cuando haya terminado de probar, escriba *quit*.

## <a name="more-information"></a>Más información

Para más información sobre cómo crear un cliente de Language Service, consulte la [documentación para desarrolladores](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource).
