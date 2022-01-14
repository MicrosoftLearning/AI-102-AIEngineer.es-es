---
lab:
  title: Creación de una aplicación cliente de Language Understanding
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 36f75e47910707c959495140be43c4196649d471
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625890"
---
# <a name="create-a-language-understanding-client-application"></a>Creación de una aplicación cliente de Language Understanding

El servicio Language Understanding permite definir una aplicación que encapsula un modelo de lenguaje que las aplicaciones pueden usar para interpretar la entrada de lenguaje natural de los usuarios, predecir la *intención* de los usuarios (lo que quieren lograr) e identificar las *entidades* a las que se debe aplicar la intención. Puede crear aplicaciones cliente que consuman aplicaciones de Language Understanding directamente a través de interfaces de REST o mediante kits de desarrollo de software (SDK) específicos del lenguaje.

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

## <a name="import-train-and-publish-a-language-understanding-app"></a>Importación, entrenamiento y publicación de una aplicación de Language Understanding

Si ya tiene una aplicación **Reloj** de un ejercicio anterior, puede usarla en este ejercicio. De lo contrario, siga estas instrucciones para crearla.

1. En una nueva pestaña del explorador, abra el portal de Language Understanding en `https://www.luis.ai`.
2. Inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure. Si es la primera vez que ha iniciado sesión en el portal de Language Understanding, es posible que tenga que conceder a la aplicación algunos permisos para acceder a los detalles de la cuenta. A continuación, complete los pasos de *bienvenida* seleccionando la suscripción de Azure y el recurso de creación que acaba de crear.
3. Abra la página **Aplicaciones de conversación** junto a **+Nueva aplicación**, vea la lista desplegable y seleccione **Importar como LU**.
Vaya a la subcarpeta **10-luis-client** de la carpeta del proyecto que contiene los archivos de laboratorio de este ejercicio y seleccione **Clock.lu**. A continuación, especifique un nombre único para la aplicación Clock.
4. Si aparece un panel con consejos para crear una aplicación eficaz de Language Understanding, ciérrelo.
5. En la parte superior del portal de Language Understanding, seleccione **Entrenar** para entrenar la aplicación.
6. En la parte superior derecha del portal de Language Understanding, seleccione **Publicar** y publique la aplicación en la **Espacio de producción**.
7. Una vez completada la publicación, en la parte superior del portal de Language Understanding, seleccione **Administrar**.
8. En la página **Configuración**, anote el **identificador de aplicación**. Las aplicaciones cliente lo necesitan para usar la aplicación.
9. En la página **Recursos de Azure**, en **Recursos de predicción**, si no aparece ningún recurso de predicción, agregue el recurso de predicción en la suscripción de Azure.
10. Anote la **Clave principal**, la **Clave secundaria** y la **Dirección URL de punto de conexión** para el recurso de predicción. Las aplicaciones cliente necesitan el punto de conexión y una de las claves para conectarse al recurso de predicción y autenticarse.

## <a name="prepare-to-use-the-language-understanding-sdk"></a>Preparación para usar el SDK de Language Understanding

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa la aplicación de reloj de Language Understanding para predecir las intenciones de la entrada del usuario y responder adecuadamente.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **10-luis-client** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **clock-client** y abra un terminal integrado. A continuación, instale el SDK de Language Understanding mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
```

*Además del paquete **Tiempo de ejecución** (predicción), hay un paquete **Creación** que puede usar para escribir código para crear y administrar modelos de Language Understanding.*

**Python**

```
pip install azure-cognitiveservices-language-luis==0.7.0
```
    
*En el paquete del SDK de Python se incluyen clases para la **predicción** y la **creación**.*

3. Consulte el contenido de la carpeta **clock-client** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para incluir el **Id. de aplicación** de la aplicación de Language Understanding, la **Dirección URL de punto de conexión** y una de las **Claves** del recurso de predicción (en la página **Administrar** para la aplicación en el portal de Language Understanding).

4. Tenga en cuenta que la carpeta **clock-client** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: clock-client.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del idioma para importar los espacios de nombres que necesitará para usar el SDK de predicción de Language Understanding:

**C#**

```C#
// Import namespaces
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
```

**Python**

```Python
# Import namespaces
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
```

## <a name="get-a-prediction-from-the-language-understanding-app"></a>Obtención de una predicción desde la aplicación de Language Understanding

Ahora está listo para implementar código que usa el SDK para obtener una predicción de la aplicación de Language Understanding.

1. En la función **Main**, fíjese en que ya se ha proporcionado código para cargar el id. de aplicación, el punto de conexión de predicción y la clave del archivo de configuración. A continuación, busque el comentario **Create a client for the LU app** (Crear un cliente para la aplicación de LU) y agregue el código siguiente para crear un cliente de predicción para la aplicación de Language Understanding:

**C#**

```C#
// Create a client for the LU app
var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.ApiKeyServiceClientCredentials(predictionKey);
var luClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
```

**Python**

```Python
# Create a client for the LU app
credentials = CognitiveServicesCredentials(lu_prediction_key)
lu_client = LUISRuntimeClient(lu_prediction_endpoint, credentials)
```

2. Tenga en cuenta que el código de la función **Main** solicita entrada del usuario hasta que ese escribe "quit". Dentro de este bucle, busque el comentario **Call the LU app to get intent and entities** (Llamada a la aplicación LU para obtener la intención y las entidades) y agregue el código siguiente:

**C#**

```C#
// Call the LU app to get intent and entities
var slot = "Production";
var request = new PredictionRequest { Query = userText };
PredictionResponse predictionResponse = await luClient.Prediction.GetSlotPredictionAsync(luAppId, slot, request);
Console.WriteLine(JsonConvert.SerializeObject(predictionResponse, Formatting.Indented));
Console.WriteLine("--------------------\n");
Console.WriteLine(predictionResponse.Query);
var topIntent = predictionResponse.Prediction.TopIntent;
var entities = predictionResponse.Prediction.Entities;
```

**Python**

```Python
# Call the LU app to get intent and entities
request = { "query" : userText }
slot = 'Production'
prediction_response = lu_client.prediction.get_slot_prediction(lu_app_id, slot, request)
top_intent = prediction_response.prediction.top_intent
entities = prediction_response.prediction.entities
print('Top Intent: {}'.format(top_intent))
print('Entities: {}'.format (entities))
print('-----------------\n{}'.format(prediction_response.query))
```

La llamada a la aplicación de Language Understanding devuelve una predicción, que incluye la intención superior (la más probable), así como cualquier entidad detectada en la expresión de entrada. La aplicación cliente debe usar ahora esa predicción para determinar y realizar la acción adecuada.

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
            if (entities.ContainsKey("Location"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Location"].ToString());
                // ML entities are strings, get the first one
                location = entityJson[0].ToString();
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
            if (entities.ContainsKey("Date"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Date"].ToString());
                // Regex entities are strings, get the first one
                date = entityJson[0].ToString();
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
            if (entities.ContainsKey("Weekday"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Weekday"].ToString());
                // List entities are lists
                day = entityJson[0][0].ToString();
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
        if 'Location' in entities:
            # ML entities are strings, get the first one
            location = entities['Location'][0]
    # Get the time for the specified location
    print(GetTime(location))

elif top_intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if len(entities) > 0:
        # Check for a Date entity
        if 'Date' in entities:
            # Regex entities are strings, get the first one
            date_string = entities['Date'][0]
    # Get the day for the specified date
    print(GetDay(date_string))

elif top_intent == 'GetDate':
    day = 'today'
    # Check for entities
    if len(entities) > 0:
        # Check for a Weekday entity
        if 'Weekday' in entities:
            # List entities are lists
            day = entities['Weekday'][0][0]
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

> **Nota**: La lógica de la aplicación es deliberadamente sencilla y tiene una serie de limitaciones. Por ejemplo, al obtener la hora, solo se admite un conjunto restringido de ciudades y se omite el horario de verano. El objetivo es ver un ejemplo de un patrón típico para usar Language Understanding en el que la aplicación debe:
>
>   1. Conectarse a un punto de conexión de la predicción.
>   2. Enviar una expresión para obtener una predicción.
>   3. Implementar la lógica para responder correctamente a la intención y las entidades predichos.

6. Cuando haya terminado de probar, escriba *quit*.

## <a name="more-information"></a>Más información

Para más información sobre cómo crear un cliente de Language Understanding, consulte la [documentación para desarrolladores](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource).
