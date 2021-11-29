---
lab:
  title: Traducción de texto
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 4ca4394ce5d9456abeabbdded1ee219f271f284e
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625891"
---
# <a name="translate-text"></a>Traducción de texto

**Translator** es un servicio cognitivo que permite traducir texto entre idiomas.

Por ejemplo, supongamos que una agencia de viajes quiere examinar las reseñas de hoteles que se han enviado al sitio web de la empresa, estandarizando el inglés como el idioma que se usa para el análisis. Mediante el uso del servicio Translator, pueden determinar el idioma en el que se ha escrito cada reseña y, si aún no está en inglés, traducirla desde cualquier idioma de origen en el que se haya escrito en inglés.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="provision-a-cognitive-services-resource"></a>Aprovisionamiento de un recurso de Cognitive Services

Si aún no tiene ninguno en su suscripción, deberá aprovisionar un recurso de **Cognitive Services**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **+Crear un recurso**, busque *Cognitive Services* y cree un recurso de **Cognitive Services** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
3. Active las casillas necesarias y cree el recurso.
4. Espere a que se complete la implementación y, a continuación, vea los detalles de la implementación.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará una de las claves y la ubicación en la que se aprovisiona el servicio desde esta página en el procedimiento siguiente.

## <a name="prepare-to-use-the-translator-service"></a>Preparación para usar el servicio Translator

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa la API de REST de Translator para traducir las reseñas de hoteles.

> **Nota**: Puede elegir usar la API de **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **06-translate-text** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Consulte el contenido de la carpeta **text-translation** y fíjese en que contiene un archivo para las opciones de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para incluir una **clave** de autenticación para el recurso de Cognitive Services y la **ubicación** en la que se implementa (<u>no</u> el punto de conexión): debe copiar ambos valores de la página **Keys and Endpoint** (Claves y punto de conexión) para el recurso de Cognitive Services. Guarde los cambios.
3. Tenga en cuenta que la carpeta **text-translation** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: text-translation.py

    Abra el archivo de código y examine el código que contiene.

4. En la función **Main**, fíjese en que ya se ha proporcionado código para cargar la clave de Cognitive Services y la región del archivo de configuración. El punto de conexión del servicio de traducción también se especifica en el código.
5. Haga clic con el botón derecho en la carpeta **text-translation**, abra un terminal integrado y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

6. Observe la salida, ya que el código debe ejecutarse sin errores, y mostrar el contenido de cada archivo de texto de reseñas en la carpeta **reviews**. Actualmente, la aplicación no hace uso del servicio Translator. Lo corregiremos en el siguiente procedimiento.

## <a name="detect-language"></a>Detectar idioma

El servicio Translator puede detectar automáticamente el idioma de origen del texto que se va a traducir, pero también permite detectar explícitamente el idioma en el que está escrito el texto.

1. En el archivo de código, busque la función **GetLanguage**, que actualmente devuelve "en" para todos los valores de texto.
2. En la función **GetLanguage**, en el comentario **Use the Translator detect function** (Usar la función de detección de Translator), agregue el código siguiente para usar la API de REST de Translator para detectar el idioma del texto especificado, y tenga cuidado de no reemplazar el código al final de la función que devuelve el idioma:

**C#**

```C#
// Use the Translator detect function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/detect?api-version=3.0";
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get language
        JArray jsonResponse = JArray.Parse(responseContent);
        language = (string)jsonResponse[0]["language"]; 
    }
}
```

**Python**

```Python
# Use the Translator detect function
path = '/detect'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0'
}

headers = {
'Ocp-Apim-Subscription-Key': cog_key,
'Ocp-Apim-Subscription-Region': cog_region,
'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get language
language = response[0]["language"]
```

3. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-translation** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. Observe la salida y tenga en cuenta que esta vez se identifica el idioma de cada reseña.

## <a name="translate-text"></a>Traducir texto

Ahora que la aplicación puede determinar el idioma en el que están escritas las reseñas, puede usar el servicio Translator para traducir las reseñas que no estén en inglés al inglés.

1. En el archivo de código, busque la función **Translate**, que actualmente devuelve una cadena vacía para todos los valores de texto.
2. En la función **Translate**, en el comentario **Use the Translator translate function** (Usar la función de traducción de Translator), agregue el código siguiente para usar la API de REST de Translator para traducir el texto especificado desde el idioma de origen al inglés, y tenga cuidado de no reemplazar el código al final de la función que devuelve la traducción:

**C#**

```C#
// Use the Translator translate function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/translate?api-version=3.0&from=" + sourceLanguage + "&to=en" ;
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get translation
        JArray jsonResponse = JArray.Parse(responseContent);
        translation = (string)jsonResponse[0]["translations"][0]["text"];  
    }
}
```

**Python**

```Python
# Use the Translator translate function
path = '/translate'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0',
    'from': source_language,
    'to': ['en']
}

headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get translation
translation = response[0]["translations"][0]["text"]
```

3. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-translation** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. Observe la salida y que las reseñas que no están en inglés se traducen al inglés.

## <a name="more-information"></a>Más información

Para más información sobre el servicio **Translator**, consulte la [documentación de Translator](https://docs.microsoft.com/azure/cognitive-services/translator/).
