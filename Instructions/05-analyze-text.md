---
lab:
  title: Análisis de texto
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 27cd22e81d4fe8fda27e6fc960d3b3aeb8debded
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801330"
---
# <a name="analyze-text"></a>Análisis de texto

El servicio de **lenguaje** forma parte de Cognitive Services y permite el análisis de texto, que incluye la detección del idioma, el análisis de sentimiento, la extracción de frases clave y el reconocimiento de entidades.

Por ejemplo, supongamos que una agencia de viajes quiere procesar las reseñas de hoteles que se han enviado al sitio web de la empresa. Mediante el servicio de lenguaje, la agencia puede determinar el idioma en el que se ha escrito cada reseña, la opinión (positiva, neutra o negativa) de las reseñas, las frases clave que podrían indicar los temas principales que se tratan en la reseña y las entidades con nombre, como lugares, puntos de referencia o personas mencionadas en las reseñas.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

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
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## <a name="prepare-to-use-the-language-sdk-for-text-analytics"></a>Preparación del uso del SDK de lenguaje para el análisis de texto

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de análisis de texto del servicio de lenguaje para analizar reseñas de hoteles.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **05-analyze-text** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **text-analysis** y abra un terminal integrado. A continuación, instale el paquete del SDK de Text Analytics mediante la ejecución del comando adecuado para sus preferencias de lenguaje:
    
    **C#**
    
    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```
    
    **Python**
    
    ```
    pip install azure-ai-textanalytics==5.1.0
    ```
    
3. Consulte el contenido de la carpeta **text-analysis** y observe que contiene un archivo para las opciones de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Cognitive Services. Guarde los cambios.

4. Observe que la carpeta **text-analysis** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: text-analysis.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Text Analytics:

    **C#**
    
    ```C#
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

5. En la función **Main**, observe que ya se ha proporcionado código para cargar la clave y el punto de conexión de Cognitive Services del archivo de configuración. A continuación, busque el comentario **Create client using endpoint and key**(Crear cliente mediante el punto de conexión y la clave) y agregue el código siguiente para crear un cliente para Text Analysis API:

    **C#**

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(cogSvcKey);
    Uri endpoint = new Uri(cogSvcEndpoint);
    TextAnalyticsClient CogClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(cog_key)
    cog_client = TextAnalyticsClient(endpoint=cog_endpoint, credential=credential)
    ```

6. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-analysis** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

6. Observe la salida, ya que el código debe ejecutarse sin errores y mostrar el contenido de cada archivo de texto de revisión en la carpeta **reviews**. La aplicación crea correctamente un cliente para Text Analytics API, pero no lo usa. Lo corregiremos en el siguiente procedimiento.

## <a name="detect-language"></a>Detectar idioma

Ahora que ha creado un cliente para Text Analytics API, vamos a usarlo para detectar el idioma en el que se escribe cada revisión.

1. En la función **Main** del programa, busque el comentario **Get language** (Obtener idioma). A continuación, en este comentario, agregue el código necesario para detectar el idioma de cada documento de revisión:

    **C#**
    
    ```C
    // Get language
    DetectedLanguage detectedLanguage = CogClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**
    
    ```Python
    # Get language
    detectedLanguage = cog_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

    > **Nota**: *En este ejemplo, cada revisión se analiza individualmente, lo que da lugar a una llamada independiente al servicio para cada archivo. Un enfoque alternativo es crear una colección de documentos y pasarlos al servicio en una sola llamada. En ambos enfoques, la respuesta del servicio consta de una colección de documentos; por eso, en el código de Python anterior, se especifica el índice del primer (y único) documento en la respuesta ([0]).*

6. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-analysis** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

7. Observe la salida y que esta vez se identifica el idioma de cada revisión.

## <a name="evaluate-sentiment"></a>Evaluación de la opinión

El *análisis de sentimiento* es una técnica que se usa habitualmente para clasificar el texto como *positivo* o *negativo* (o posiblemente *neutro* o *mixto*). Se usa normalmente para analizar publicaciones en redes sociales, reseñas de productos y otros elementos en los que la opinión del texto puede proporcionar información útil.

1. En la función **Main** del programa, busque el comentario **Get sentiment** (Obtener opinión). A continuación, en este comentario, agregue el código necesario para detectar la opinión de cada documento de revisión:

    **C#**
    
    ```C
    // Get sentiment
    DocumentSentiment sentimentAnalysis = CogClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**
    
    ```Python
    # Get sentiment
    sentimentAnalysis = cog_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-analysis** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

3. Observe la salida y que se detecta la opinión de las revisiones.

## <a name="identify-key-phrases"></a>Identificación de frases clave

Puede ser útil identificar frases clave en un cuerpo de texto para ayudar a determinar los temas principales que se tratan.

1. En la función **Main** del programa, busque el comentario **Get key phrases** (Obtener frases clave). A continuación, en este comentario, agregue el código necesario para detectar las frases clave en cada documento de revisión:

    **C#**

    ```C
    // Get key phrases
    KeyPhraseCollection phrases = CogClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```
    
    **Python**
    
    ```Python
    # Get key phrases
    phrases = cog_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-analysis** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Observe la salida y que cada documento contiene frases clave que proporcionan información sobre el tema de la revisión.

## <a name="extract-entities"></a>Extraer entidades

A menudo, los documentos u otros cuerpos de texto mencionan personas, lugares, períodos de tiempo u otras entidades. Text Analytics API puede detectar varias categorías (y subcategorías) de la entidad en el texto.

1. En la función **Main** del programa, busque el comentario **Get entities** (Obtener entidades). A continuación, en este comentario, agregue el código necesario para identificar las entidades que se mencionan en cada revisión:

    **C#**
    
    ```C
    // Get entities
    CategorizedEntityCollection entities = CogClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get entities
    entities = cog_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-analysis** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Observe la salida y las entidades que se han detectado en el texto.

## <a name="extract-linked-entities"></a>Extracción de entidades vinculadas

Además de las entidades clasificadas, Text Analytics API puede detectar entidades para las que hay vínculos conocidos a orígenes de datos, como Wikipedia.

1. En la función **Main** del programa, busque el comentario **Get linked entities** (Obtener entidades vinculadas). A continuación, en este comentario, agregue el código necesario para identificar las entidades vinculadas que se mencionan en cada revisión:

    **C#**
    
    ```C
    // Get linked entities
    LinkedEntityCollection linkedEntities = CogClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get linked entities
    entities = cog_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-analysis** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Observe la salida y las entidades vinculadas que se identifican.

## <a name="more-information"></a>Más información

Para obtener más información sobre el servicio de **lenguaje**, consulte la [documentación de Text Analytics](https://docs.microsoft.com/azure/cognitive-services/language-service/).
