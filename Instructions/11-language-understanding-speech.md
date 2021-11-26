---
lab:
  title: Uso de los servicios de Voz y Language Understanding
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 31b88612bce38780f828859f8e2b166dbe8df34d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625981"
---
# <a name="use-the-speech-and-language-understanding-services"></a>Uso de los servicios de Voz y Language Understanding

Puede integrar el servicio de Voz con el servicio Language Understanding para crear aplicaciones que puedan determinar de forma inteligente las intenciones del usuario a partir de la entrada hablada.

> **Nota**: Este ejercicio funciona mejor si tiene un micrófono. Algunos entornos virtuales hospedados pueden capturar audio del micrófono local, pero si esto no funciona (o no tiene micrófono), puede usar un archivo de audio proporcionado para la entrada de voz. Siga las instrucciones detenidamente, ya que deberá elegir diferentes opciones en función de si usa un micrófono o el archivo de audio.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="create-language-understanding-resources"></a>Creación de los recursos de Language Understanding

Si ya tiene recursos de creación y predicción de Language Understanding en su suscripción de Azure, puede usarlos en este ejercicio. De lo contrario, siga estas instrucciones para crearlos.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **&amp;#65291;Crear un recurso**, busque *language understanding* y cree un recurso de **Language Understanding** con la siguiente configuración:
    - **Create option** (Opción de creación): ambos
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Nombre**: *escriba un nombre único*
    - **Ubicación de creación**: *seleccione su ubicación preferida*
    - **Plan de tarifa de creación**: F0
    - **Ubicación de predicción**: *elija la <u>misma ubicación</u> que la ubicación de creación*
    - **Plan de tarifa de predicción**: F0 (*Si F0 no está disponible, elija S0*)

3. Espere a que se creen los recursos y observe que se aprovisionan dos recursos de Language Understanding, uno para la creación y otro para la predicción. Puede ver ambos navegando hasta el grupo de recursos donde los creó.

## <a name="prepare-a-language-understanding-app"></a>Preparación de una aplicación de Language Understanding

Si ya tiene una aplicación **Clock** de un ejercicio anterior, ábrala en el portal de Language Understanding en `https://www.luis.ai`. De lo contrario, siga estas instrucciones para crearla.

1. En una nueva pestaña del explorador, abra el portal de Language Understanding en `https://www.luis.ai`.
2. Inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure. Si es la primera vez que ha iniciado sesión en el portal de Language Understanding, es posible que tenga que conceder a la aplicación algunos permisos para acceder a los detalles de la cuenta. A continuación, complete los pasos de *bienvenida* seleccionando la suscripción de Azure y el recurso de creación que acaba de crear.
3. Abra la página **Aplicaciones de conversación** junto a **+Nueva aplicación**, vea la lista desplegable y seleccione **Importar como LU**.
Vaya a la subcarpeta **11-luis-speech** de la carpeta del proyecto que contiene los archivos de laboratorio de este ejercicio y seleccione **Clock.lu**. A continuación, especifique un nombre único para la aplicación Clock.
4. Si aparece un panel con consejos para crear una aplicación eficaz de Language Understanding, ciérrelo.

## <a name="train-and-publish-the-app-with-speech-priming"></a>Entrenamiento y publicación de la aplicación con *preparación para la voz*

1. En la parte superior del portal de Language Understanding, seleccione **Entrenar** para entrenar la aplicación si aún no se ha entrenado.
2. En la parte superior derecha del portal de Language Understanding, seleccione **Publicar**. A continuación, seleccione el **espacio de producción** y cambie la configuración para habilitar **Preparación para la voz** (esto dará lugar a un mejor rendimiento para el reconocimiento de voz).
3. Una vez completada la publicación, en la parte superior del portal de Language Understanding, seleccione **Administrar**.
4. En la página **Configuración**, anote el **identificador de aplicación**. Las aplicaciones cliente lo necesitan para usar la aplicación.
5. En la página **Recursos de Azure**, en **Recursos de predicción**, si no aparece ningún recurso de predicción, agregue el recurso de predicción en la suscripción de Azure.
6. Anote la **clave principal**, la **clave secundaria** y la **ubicación** (¡<u>no</u> el punto de conexión!) para el recurso de predicción. Las aplicaciones cliente del SDK de Voz necesitan la ubicación y una de las claves para conectarse al recurso de predicción y autenticarse.

## <a name="configure-a-client-application-for-language-understanding"></a>Configuración de una aplicación cliente para Language Understanding

En este ejercicio, creará una aplicación cliente que acepta la entrada hablada y usa la aplicación Language Understanding para predecir la intención del usuario.

> **Nota**: En este ejercicio, puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **11-luis-speech** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Vea el contenido de la carpeta **speaking-clock-client** y fíjese en que contiene un archivo para las opciones de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para incluir el **identificador de aplicación** de la aplicación de Language Understanding y la **ubicación** (<u>no</u> el punto de conexión completo; por ejemplo, *eastus*) y una de las **claves** para su recurso de predicción (desde la página **Administrar** de la aplicación en el portal de Language Understanding).

## <a name="install-sdk-packages"></a>Instalación de paquetes de SDK

Para usar el SDK de Voz con el servicio de Language Understanding, debe instalar el paquete del SDK de Voz para su lenguaje de programación.

1. En Visual Studio, haga clic con el botón derecho en la carpeta **speaking-clock-client** y abra un terminal integrado. A continuación, instale el SDK de Language Understanding mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

2. Además, si el sistema <u>no</u> tiene un micrófono que funcione, deberá usar un archivo de audio para proporcionar entradas habladas para la aplicación. En este caso, use los siguientes comandos para instalar un paquete adicional para que el programa pueda reproducir el archivo de audio (puede omitir esto si piensa usar un micrófono):

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

3. Tenga en cuenta que la carpeta **speaking-clock-client** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: speaking-clock-client.py

4. Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico según el lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Voz:

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Intent;
    ```

    **Python**

    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. Además, si el sistema <u>no</u> tiene un micrófono que funcione, en las importaciones del espacio de nombres existente, agregue el código siguiente para importar la biblioteca que usará para reproducir un archivo de audio:

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

## <a name="create-an-intentrecognizer"></a>Creación de *IntentRecognizer*

La clase **IntentRecognizer** proporciona un objeto de cliente que puede usar para obtener predicciones de Language Understanding a partir de una entrada hablada.

1. En la función **Main**, fíjese en que ya se ha proporcionado código para cargar el identificador de aplicación, la región de predicción y la clave del archivo de configuración. A continuación, busque el comentario **Configurar servicio de voz y obtener el reconocedor de intenciones** y agregue el código siguiente en función de si va a usar un micrófono o un archivo de audio para la entrada de voz:

    ### <a name="if-you-have-a-working-microphone"></a>**Si tiene un micrófono que funcione:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

    ### <a name="if-you-need-to-use-an-audio-file"></a>**Si necesita usar un archivo de audio:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    string audioFile = "time-in-london.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    System.Threading.Thread.Sleep(2000);
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    audioFile = 'time-in-london.wav'
    playsound(audioFile)
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

## <a name="get-a-predicted-intent-from-spoken-input"></a>Obtener una intención predicha a partir de la entrada hablada

Ahora está listo para implementar código que use el SDK de Voz para obtener una intención predicha a partir de la entrada hablada.

1. En la función **Main**, inmediatamente debajo del código que acaba de agregar, busque el comentario **Obtener el modelo a partir del identificador de aplicación y agregar las intenciones que queremos usar** y agregue el código siguiente para obtener el modelo de Language Understanding (en función de su identificador de aplicación) y especifique las intenciones que queremos que identifique el reconocedor.

    **C#**

    ```C#
    // Get the model from the AppID and add the intents we want to use
    var model = LanguageUnderstandingModel.FromAppId(luAppId);
    recognizer.AddIntent(model, "GetTime", "time");
    recognizer.AddIntent(model, "GetDate", "date");
    recognizer.AddIntent(model, "GetDay", "day");
    recognizer.AddIntent(model, "None", "none");
    ```

    *Tenga en cuenta que puede especificar un identificador basado en cadena para cada intención*

    **Python**

    ```Python
    # Get the model from the AppID and add the intents we want to use
    model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
    intents = [
        (model, "GetTime"),
        (model, "GetDate"),
        (model, "GetDay"),
        (model, "None")
    ]
    recognizer.add_intents(intents)
    ```

2. En el comentario **Procesar entrada de voz**, agregue el código siguiente, que usa el reconocedor para llamar asincrónicamente al servicio Language Understanding con entrada hablada y recuperar la respuesta. Si la respuesta incluye una intención predicha, se muestra la consulta hablada, la intención predicha y la respuesta JSON completa. De lo contrario, el código controla la respuesta en función del motivo devuelto.

**C#**

```C
// Process speech input
string intent = "";
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // Intent was identified
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // Get the first entity (if any)

    // Apply the appropriate action
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // Speech was recognized, but no intent was identified.
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // Speech wasn't recognized
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // Something went wrong
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");
    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# Process speech input
intent = ''
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # Get the first entity (if any)
    
    # Apply the appropriate action
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # Speech was recognized, but no intent was identified.
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # Speech wasn't recognized
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # Something went wrong
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```

El código que ha agregado hasta ahora identifica la *intención*, pero algunas intenciones pueden hacer referencia a *entidades*, por lo que debe agregar código para extraer la información de entidad del JSON devuelto por el servicio.

3. En el código que acaba de agregar, busque el comentario **Obtener la primera entidad (si la hay)** y agregue el código siguiente debajo de él:

**C#**

```C
// Get the first entity (if any)
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine(entityType + ": " + entityValue);
}
```

**Python**

```Python
# Get the first entity (if any)
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
    
El código ahora usa la aplicación Language Understanding para predecir una intención, así como las entidades que se detectaron en la expresión de entrada. La aplicación cliente debe usar ahora esa predicción para determinar y realizar la acción adecuada.

4. Debajo del código que acaba de agregar, busque el comentario **Aplicar la acción adecuada** y agregue el código siguiente, que comprueba las intenciones admitidas por la aplicación(**GetTime**,**GetDate** y **GetDay**) y determina si se ha detectado alguna entidad pertinente, antes de llamar a una función existente para generar una respuesta adecuada.

**C#**

```C#
// Apply the appropriate action
switch (intent)
{
    case "time":
        var location = "local";
        // Check for entities
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if intent == 'GetTime':
    location = 'local'
    # Check for entities
    if entity_type == 'Location':
        location = entity_value
    # Get the time for the specified location
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if entity_type == 'Date':
        date_string = entity_value
    # Get the day for the specified date
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # Check for entities
    if entity_type == 'Weekday':
        # List entities are lists
        day = entity_value
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

## <a name="run-the-client-application"></a>Ejecución de la aplicación cliente

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock-client** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock-client.py
    ```

2. Si usa un micrófono, diga expresiones en voz alta para probar la aplicación. Por ejemplo, pruebe lo siguiente (vuelva a ejecutar el programa cada vez):

    *¿Qué hora es?*
    
    *¿Qué hora tenemos?*

    *¿Qué día es?*

    *¿Qué hora es en Londres?*

    *¿Qué fecha tenemos?*

    *¿Qué fecha es el domingo?*

> **Nota**: La lógica de la aplicación es deliberadamente sencilla y tiene una serie de limitaciones, pero debe servir para probar la capacidad del modelo de Language Understanding para predecir intenciones a partir de entradas habladas mediante el SDK de Voz. Es posible que tenga problemas para reconocer la intención **GetDay** con una entidad de fecha específica debido a la dificultad para verbalizar una fecha en el formato *MM/DD/AAAA*.

## <a name="more-information"></a>Más información

Para obtener más información sobre la integración de Voz y Language Understanding, consulte la [documentación de Voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition).
