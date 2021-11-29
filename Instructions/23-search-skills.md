---
lab:
  title: Creación de una aptitud personalizada para Azure Cognitive Search
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 288b9badac3a3ec4d1461a5da14faa2e52143959
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625898"
---
# <a name="create-a-custom-skill-for-azure-cognitive-search"></a>Creación de una aptitud personalizada para Azure Cognitive Search

Azure Cognitive Search usa una canalización de enriquecimiento de aptitudes cognitivas para extraer campos generados mediante inteligencia artificial de documentos e incluirlos en un índice de búsqueda. Hay un conjunto completo de aptitudes integradas que puede usar, pero si tiene un requisito específico que no cumplen estas aptitudes, puede crear una aptitud personalizada.

En este ejercicio, creará una aptitud personalizada que tabula la frecuencia de las palabras individuales de un documento para generar una lista de las cinco palabras más usadas y agregarla a una solución de búsqueda para Margie's Travel, una agencia de viajes ficticia.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús+Ctrl+P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="create-azure-resources"></a>Creación de recursos de Azure

> **Nota**: Si ha completado previamente el ejercicio **[Creación de una solución de Azure Cognitive Search](22-azure-search.md)** y todavía tiene esos recursos de Azure en su suscripción, puede omitir esta sección y empezar en la sección **Creación de una solución de búsqueda**. De lo contrario, siga los pasos que se indican a continuación para aprovisionar los recursos de Azure necesarios.

1. En un explorador web, abra Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Consulte los **Grupos de recursos** de su suscripción.
3. Si usa una suscripción restringida en la que se le ha proporcionado un grupo de recursos, seleccione el grupo de recursos para ver sus propiedades. En caso contrario, cree un nuevo grupo de recursos con el nombre que prefiera y vaya a él cuando se haya creado.
4. En la página **Información general** del grupo de recursos, tome nota de los valores de **Id. de suscripción** y **Ubicación**. Los necesitará, junto con el nombre del grupo de recursos, en los pasos posteriores.
5. En Visual Studio Code, expanda la carpeta **23-custom-search-skill** y seleccione **setup.cmd**. Usará este script por lotes para ejecutar los comandos de la interfaz de la línea de comandos (CLI) de Azure necesarios para crear los recursos de Azure que necesita.
6. Haga clic con el botón derecho en la carpeta **23-custom-search-skill** y seleccione **Abrir en terminal integrado**.
7. En el panel de terminal, escriba el siguiente comando para establecer una conexión autenticada con su suscripción de Azure.

    ```
    az login --output none
    ```

8. Cuando se le solicite, inicie sesión en su suscripción de Azure. Después, vuelva a Visual Studio Code y espere a que se complete el proceso de inicio de sesión.
9. Ejecute el siguiente comando para enumerar las ubicaciones de Azure.

    ```
    az account list-locations -o table
    ```

10. En la salida, busque el valor de **Nombre** que se corresponda con la ubicación del grupo de recursos (por ejemplo, para *Este de EE. UU.* , el nombre correspondiente es *eastus*).
11. En el script **setup.cmd**, modifique las declaraciones de las variables **subscription_id**, **resource_group** y **location** con los valores adecuados del id. de suscripción, el nombre del grupo de recursos y el nombre de la ubicación. A continuación, guarde los cambios.
12. En el terminal de la carpeta **23-custom-search-skill**, escriba el siguiente comando para ejecutar el script:

    ```
    setup
    ```

    > **Nota**: El módulo de la CLI de búsqueda está en versión preliminar y puede dejar de responder en el proceso *- En ejecución ..* . process. Si esto sucede durante más de dos minutos, presione Ctrl+C para cancelar la operación de ejecución larga y, a continuación, seleccione **N** cuando se le pregunte si desea finalizar el script. A continuación, debería completarse correctamente.
    >
    > Si se produce un error en el script, asegúrese de guardarlo con los nombres de variable correctos e inténtelo de nuevo.

13. Cuando se complete el script, revise la salida que se muestra y anote la siguiente información de los recursos de Azure (necesitará estos valores más adelante):
    - Nombre de la cuenta de almacenamiento
    - Cadena de conexión de almacenamiento
    - Cuenta de Cognitive Services
    - Clave de Cognitive Services
    - Punto de conexión del servicio de búsqueda
    - Clave de administración del servicio de búsqueda
    - Clave de consulta del servicio de búsqueda

14. En Azure Portal, actualice el grupo de recursos y compruebe que contiene la cuenta de Azure Storage, el recurso de Azure Cognitive Services y el recurso de Azure Cognitive Search.

## <a name="create-a-search-solution"></a>Creación de una solución de búsqueda

Ahora que tiene los recursos de Azure necesarios, puede crear una solución de búsqueda que conste de los siguientes componentes:

- Un **origen de datos** que haga referencia a los documentos del contenedor de almacenamiento de Azure.
- Un **conjunto de aptitudes** que defina una canalización de enriquecimiento de aptitudes para extraer campos generados mediante inteligencia artificial de los documentos.
- Un **índice** que defina un conjunto de registros de documentos en los que se pueden realizar búsquedas.
- Un **indexador** que extraiga los documentos del origen de datos, aplique el conjunto de aptitudes y rellene el índice.

En este ejercicio, usará la interfaz de REST de Azure Cognitive Search para crear estos componentes mediante el envío de solicitudes JSON.

1. En Visual Studio Code, en la carpeta **23-custom-search-skill**, expanda la carpeta **create-search** y seleccione **data_source.json**. Este archivo contiene una definición JSON para un origen de datos denominado **margies-custom-data**.
2. Reemplace el marcador de posición **YOUR_CONNECTION_STRING** por la cadena de conexión de la cuenta de almacenamiento de Azure, que debe ser similar a la siguiente:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Puede encontrar la cadena de conexión en la página **Claves de acceso** de su cuenta de almacenamiento en Azure Portal.*

3. Guarde y cierre el archivo JSON actualizado.
4. En la carpeta **create-search,** , abra **skillset.json**. Este archivo contiene una definición JSON para un conjunto de aptitudes denominado **margies-custom-skillset**.
5. En la parte superior de la definición del conjunto de aptitudes, en el elemento **cognitiveServices**, reemplace el marcador de posición **YOUR_COGNITIVE_SERVICES_KEY** por cualquiera de las claves de los recursos de Cognitive Services.

    *Puede encontrar las claves en la página **Claves y punto de conexión** del recurso de Cognitive Services en Azure Portal.*

6. Guarde y cierre el archivo JSON actualizado.
7. En la carpeta **create-search,** , abra **index.json**. Este archivo contiene una definición JSON para un índice denominado **margies-custom-index**.
8. Revise el archivo JSON del índice y ciérrelo sin realizar ningún cambio.
9. En la carpeta **create-search,** , abra **indexer.json**. Este archivo contiene una definición JSON para un indexador denominado **margies-custom-indexer**.
10. Revise el archivo JSON del indexador y ciérrelo sin realizar ningún cambio.
11. En la carpeta **create-search,** , abra **create-search.cmd**. Este script por lotes usa la utilidad cURL para enviar las definiciones JSON a la interfaz de REST del recurso de Azure Cognitive Search.
12. Reemplace los marcadores de posición de variables **YOUR_SEARCH_URL** y **YOUR_ADMIN_KEY** por la **dirección URL** y una de las **claves de administración** del recurso de Azure Cognitive Search.

    *Puede encontrar estos valores en las páginas **Información general** y **Claves** del recurso de Azure Cognitive Search en Azure Portal.*

13. Guarde el archivo por lotes actualizado.
14. Haga clic con el botón derecho en la carpeta **create-search** y seleccione **Abrir en terminal integrado**.
15. En el panel del terminal de la carpeta **create-search**, escriba el siguiente comando para ejecutar el script por lotes.

    ```
    create-search
    ```

16. Cuando se complete el script, en Azure Portal, en la página del recurso de Azure Cognitive Search, seleccione la página **Indizadores** y espere a que se complete el proceso de indexación.

    *Puede seleccionar **Actualizar** para supervisar el progreso de la operación de indexación. Puede tardar alrededor de un minuto en completarse.*

## <a name="search-the-index"></a>Búsqueda en el índice

Ahora que tiene un índice, puede realizar búsquedas en él.

1. En la parte superior del panel del recurso de Azure Cognitive Search, seleccione **Explorador de búsqueda**.
2. En el Explorador de búsqueda, en el cuadro **Cadena de consulta**, escriba la siguiente cadena de consulta y, a continuación, seleccione **Buscar**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment gt 0.5
    ```

    Esta consulta recupera los valores de **url**, **sentiment** y **keyphrases** de todos los documentos que mencionan *London* (Londres), cuyo autor es *Reviewer* (Revisor) y que tienen una puntuación de **opinión** superior a *0,5* (es decir, reseñas positivas que mencionan "London" [Londres]).

## <a name="create-an-azure-function-for-a-custom-skill"></a>Creación de una función de Azure para una aptitud personalizada

La solución de búsqueda incluye una serie de aptitudes cognitivas integradas que enriquecen el índice con información de los documentos, como las puntuaciones de opinión y las listas de frases clave que se han visto en la tarea anterior.

Puede mejorar aún más el índice mediante la creación de aptitudes personalizadas. Por ejemplo, puede ser útil identificar las palabras que se usan con más frecuencia en cada documento, pero ninguna aptitud integrada ofrece esta funcionalidad.

Para implementar la funcionalidad de recuento de palabras como una aptitud personalizada, deberá crear una función de Azure en el lenguaje que prefiera.

1. En Visual Studio Code, vaya a la pestaña de extensiones de Azure (**&boxplus;**) y compruebe que la extensión **Azure Functions** está instalada. Esta extensión le permite crear e implementar funciones de Azure Functions desde Visual Studio Code.
2. En la pestaña de Azure ( **&Delta;** ), en el panel **Azure Functions**, cree un nuevo proyecto (&#128194;) con la siguiente configuración, en función del lenguaje que prefiera:

    ### <a name="c"></a>**C#**

    - **Carpeta**: vaya a **23-custom-search-skill/C-Sharp/wordcount**
    - **Lenguaje**: C#
    - **Plantilla**: Desencadenador de HTTP
    - **Nombre de la función**: wordcount
    - **Espacio de nombres**: margies.search
    - **Nivel de autorización**: Función

    ### <a name="python"></a>**Python**

    - **Carpeta**: vaya a **23-custom-search-skill/Python/wordcount**
    - **Lenguaje**: Python
    - **Entorno virtual**: omita la configuración del entorno virtual.
    - **Plantilla**: Desencadenador de HTTP
    - **Nombre de la función**: wordcount
    - **Nivel de autorización**: Función

    *Si se le pide que sobrescriba el archivo **launch.json**, sobrescríbalo.*

3. Vuelva a la pestaña **Explorador** ( **&#128461;** ) y compruebe que la carpeta **wordcount** ahora contiene los archivos de código de la función de Azure.

    *Si elige Python, los archivos de código pueden estar en una subcarpeta, también denominada **wordcount**.*

4. El archivo de código principal de la función se debería haber abierto automáticamente. Si no es así, abra el archivo adecuado del lenguaje elegido:
    - **C#** : wordcount.cs
    - **Python**: \_\_init\_\_&#46;py

5. Reemplace todo el contenido del archivo con el código siguiente del idioma elegido:

### <a name="c"></a>**C#**

```C#
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;
using System.Text.RegularExpressions;
using System.Linq;

namespace margies.search
{
    public static class wordcount
    {

        //define classes for responses
        private class WebApiResponseError
        {
            public string message { get; set; }
        }

        private class WebApiResponseWarning
        {
            public string message { get; set; }
        }

        private class WebApiResponseRecord
        {
            public string recordId { get; set; }
            public Dictionary<string, object> data { get; set; }
            public List<WebApiResponseError> errors { get; set; }
            public List<WebApiResponseWarning> warnings { get; set; }
        }

        private class WebApiEnricherResponse
        {
            public List<WebApiResponseRecord> values { get; set; }
        }

        //function for custom skill
        [FunctionName("wordcount")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)]HttpRequest req, ILogger log)
        {
            log.LogInformation("Function initiated.");

            string recordId = null;
            string originalText = null;

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            // Validation
            if (data?.values == null)
            {
                return new BadRequestObjectResult(" Could not find values array");
            }
            if (data?.values.HasValues == false || data?.values.First.HasValues == false)
            {
                return new BadRequestObjectResult("Could not find valid records in values array");
            }

            WebApiEnricherResponse response = new WebApiEnricherResponse();
            response.values = new List<WebApiResponseRecord>();
            foreach (var record in data?.values)
            {
                recordId = record.recordId?.Value as string;
                originalText = record.data?.text?.Value as string;

                if (recordId == null)
                {
                    return new BadRequestObjectResult("recordId cannot be null");
                }

                // Put together response.
                WebApiResponseRecord responseRecord = new WebApiResponseRecord();
                responseRecord.data = new Dictionary<string, object>();
                responseRecord.recordId = recordId;
                responseRecord.data.Add("text", Count(originalText));

                response.values.Add(responseRecord);
            }

            return (ActionResult)new OkObjectResult(response); 
        }


            public static string RemoveHtmlTags(string html)
        {
            string htmlRemoved = Regex.Replace(html, @"<script[^>]*>[\s\S]*?</script>|<[^>]+>| ", " ").Trim();
            string normalised = Regex.Replace(htmlRemoved, @"\s{2,}", " ");
            return normalised;
        }

        public static List<string> Count(string text)
        {
            
            //remove html elements
            text=text.ToLowerInvariant();
            string html = RemoveHtmlTags(text);
            
            //split into list of words
            List<string> list = html.Split(" ").ToList();
            
            //remove any non alphabet characters
            var onlyAlphabetRegEx = new Regex(@"^[A-z]+$");
            list = list.Where(f => onlyAlphabetRegEx.IsMatch(f)).ToList();

            //remove stop words
            string[] stopwords = { "", "i", "me", "my", "myself", "we", "our", "ours", "ourselves", "you", 
                    "you're", "you've", "you'll", "you'd", "your", "yours", "yourself", 
                    "yourselves", "he", "him", "his", "himself", "she", "she's", "her", 
                    "hers", "herself", "it", "it's", "its", "itself", "they", "them", 
                    "their", "theirs", "themselves", "what", "which", "who", "whom", 
                    "this", "that", "that'll", "these", "those", "am", "is", "are", "was",
                    "were", "be", "been", "being", "have", "has", "had", "having", "do", 
                    "does", "did", "doing", "a", "an", "the", "and", "but", "if", "or", 
                    "because", "as", "until", "while", "of", "at", "by", "for", "with", 
                    "about", "against", "between", "into", "through", "during", "before", 
                    "after", "above", "below", "to", "from", "up", "down", "in", "out", 
                    "on", "off", "over", "under", "again", "further", "then", "once", "here", 
                    "there", "when", "where", "why", "how", "all", "any", "both", "each", 
                    "few", "more", "most", "other", "some", "such", "no", "nor", "not", 
                    "only", "own", "same", "so", "than", "too", "very", "s", "t", "can", 
                    "will", "just", "don", "don't", "should", "should've", "now", "d", "ll",
                    "m", "o", "re", "ve", "y", "ain", "aren", "aren't", "couldn", "couldn't", 
                    "didn", "didn't", "doesn", "doesn't", "hadn", "hadn't", "hasn", "hasn't", 
                    "haven", "haven't", "isn", "isn't", "ma", "mightn", "mightn't", "mustn", 
                    "mustn't", "needn", "needn't", "shan", "shan't", "shouldn", "shouldn't", "wasn", 
                    "wasn't", "weren", "weren't", "won", "won't", "wouldn", "wouldn't"}; 
            list = list.Where(x => x.Length > 2).Where(x => !stopwords.Contains(x)).ToList();
            
            //get distict words by key and count, and then order by count.
            var keywords = list.GroupBy(x => x).OrderByDescending(x => x.Count());
            var klist = keywords.ToList();

            // return the top 10 words
            var numofWords = 10;
            if(klist.Count<10)
                numofWords=klist.Count;
            List<string> resList = new List<string>();
            for (int i = 0; i < numofWords; i++)
            {
                resList.Add(klist[i].Key);
            }
            return resList;
        }
    }
}
```

## <a name="python"></a>**Python**

```Python
import logging
import os
import sys
import json
from string import punctuation
from collections import Counter
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Wordcount function initiated.')

    # The result will be a "values" bag
    result = {
        "values": []
    }
    statuscode = 200

    # We're going to exclude words from this list in the word counts
    stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
                "you're", "you've", "you'll", "you'd", 'your', 'yours', 'yourself', 
                'yourselves', 'he', 'him', 'his', 'himself', 'she', "she's", 'her', 
                'hers', 'herself', 'it', "it's", 'its', 'itself', 'they', 'them', 
                'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
                'this', 'that', "that'll", 'these', 'those', 'am', 'is', 'are', 'was',
                'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
                'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
                'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
                'about', 'against', 'between', 'into', 'through', 'during', 'before', 
                'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
                'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
                'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
                'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
                'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 
                'will', 'just', 'don', "don't", 'should', "should've", 'now', 'd', 'll',
                'm', 'o', 're', 've', 'y', 'ain', 'aren', "aren't", 'couldn', "couldn't", 
                'didn', "didn't", 'doesn', "doesn't", 'hadn', "hadn't", 'hasn', "hasn't", 
                'haven', "haven't", 'isn', "isn't", 'ma', 'mightn', "mightn't", 'mustn', 
                "mustn't", 'needn', "needn't", 'shan', "shan't", 'shouldn', "shouldn't", 'wasn', 
                "wasn't", 'weren', "weren't", 'won', "won't", 'wouldn', "wouldn't"]

    try:
        values = req.get_json().get('values')
        logging.info(values)

        for rec in values:
            # Construct the basic JSON response for this record
            val = {
                    "recordId": rec['recordId'],
                    "data": {
                        "text":None
                    },
                    "errors": None,
                    "warnings": None
                }
            try:
                # get the text to be processed from the input record
                txt = rec['data']['text']
                # remove numeric digits
                txt = ''.join(c for c in txt if not c.isdigit())
                # remove punctuation and make lower case
                txt = ''.join(c for c in txt if c not in punctuation).lower()
                # remove stopwords
                txt = ' '.join(w for w in txt.split() if w not in stopwords)
                # Count the words and get the most common 10
                wordcount = Counter(txt.split()).most_common(10)
                words = [w[0] for w in wordcount]
                # Add the top 10 words to the output for this text record
                val["data"]["text"] = words
            except:
                # An error occured for this text record, so add lists of errors and warning
                val["errors"] =[{"message": "An error occurred processing the text."}]
                val["warnings"] = [{"message": "One or more inputs failed to process."}]
            finally:
                # Add the value for this record to the response
                result["values"].append(val)
    except Exception as ex:
        statuscode = 500
        # A global error occurred, so return an error response
        val = {
                "recordId": None,
                "data": {
                    "text":None
                },
                "errors": [{"message": ex.args}],
                "warnings": [{"message": "The request failed to process."}]
            }
        result["values"].append(val)
    finally:
        # Return the response
        return func.HttpResponse(body=json.dumps(result), mimetype="application/json", status_code=statuscode)
```
    
6. Guarde el archivo actualizado.
7. Haga clic con el botón derecho en la carpeta **wordcount** que contiene los archivos de código y seleccione **Deploy to Function App** (Implementar en la aplicación de funciones). A continuación, implemente la función con la siguiente configuración específica del lenguaje (inicie sesión en Azure si se le solicita):

    ### <a name="c"></a>**C#**

    - **Suscripción**: seleccione su suscripción a Azure (si se le solicita).
    - **Función**: Create a new Function App in Azure (Advanced) (Creación de una aplicación de funciones en Azure [avanzado])
    - **Nombre de la aplicación de funciones**: escriba un nombre globalmente único.
    - **Entorno de ejecución**: .NET Core 3.1
    - **SO**: Linux
    - **Plan de hospedaje**: consumo
    - **Grupo de recursos**: el grupo de recursos que contiene el recurso de Azure Cognitive Search.
        - Nota: Si este grupo de recursos ya contiene una aplicación web basada en Windows, no podrá implementar en él una función basada en Linux. Elimine la aplicación web existente o implemente la función en un grupo de recursos diferente.
    - **Cuenta de almacenamiento**: el recuento de almacenamiento donde se almacenan los documentos de Margie's Travel.
    - **Application Insights**: omita esto por ahora.

    *Visual Studio Code implementará la versión compilada de la función (en la subcarpeta **bin**) según los valores de configuración de la carpeta **.vscode**, los cuales se guardaron al crear el proyecto de función.*

    ### <a name="python"></a>**Python**

    - **Suscripción**: seleccione su suscripción a Azure (si se le solicita).
    - **Función**: Create a new Function App in Azure (Advanced) (Creación de una aplicación de funciones en Azure [avanzado])
    - **Nombre de la aplicación de funciones**: escriba un nombre globalmente único.
    - **Entorno de ejecución**: Python 3.8
    - **Plan de hospedaje**: consumo
    - **Grupo de recursos**: el grupo de recursos que contiene el recurso de Azure Cognitive Search.
        - Nota: Si este grupo de recursos ya contiene una aplicación web basada en Windows, no podrá implementar en él una función basada en Linux. Elimine la aplicación web existente o implemente la función en un grupo de recursos diferente.
    - **Cuenta de almacenamiento**: el recuento de almacenamiento donde se almacenan los documentos de Margie's Travel.
    - **Application Insights**: omita esto por ahora.

8. Espere a Visual Studio Code para implementar la función. Cuando finalice la implementación, aparecerá una notificación.

## <a name="test-the-function"></a>Probar la función

Ahora que ha implementado la función en Azure, puede probarla en Azure Portal.

1. Abra [Azure Portal](https://portal.azure.com) y vaya al grupo de recursos donde ha creado la aplicación de funciones. Después, abra el servicio de aplicaciones para su aplicación de funciones.
2. En el panel de su servicio de aplicaciones, en la página **Funciones**, abra la función **wordcount**.
3. En el panel de la función **wordcount**, vaya a la página **Código y prueba** y abra el panel **Probar/ejecutar**.
4. En el panel **Probar/ejecutar**, reemplace el **cuerpo** existente por el siguiente código JSON, que refleja el esquema esperado por una aptitud de Azure Cognitive Search en la que los registros que contienen datos de uno o varios documentos se envían para su procesamiento:

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
5. Haga clic en **Ejecutar** y analice el contenido de la respuesta HTTP que devuelve la función. El contenido refleja el esquema que espera Azure Cognitive Search al consumir una aptitud, en el que se devuelve una respuesta para cada documento. En este caso, la respuesta está formada por hasta 10 términos en cada documento en orden descendente según la frecuencia con la que aparecen:

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger",
            "burning",
            "bright",
            "darkness",
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

6. Cierre el panel **Probar/ejecutar** y, en el panel de la función **wordcount**, haga clic en **Obtener la dirección URL de la función**. Después, copie la dirección URL de la clave predeterminada en el portapapeles, la necesitará en el siguiente procedimiento.

## <a name="add-the-custom-skill-to-the-search-solution"></a>Adición de la aptitud personalizada a la solución de búsqueda

Ahora tiene que incluir la función como una aptitud personalizada en el conjunto de aptitudes de la solución de búsqueda y asignar los resultados que genera a un campo del índice. 

1. En Visual Studio Code, en la carpeta **23-custom-search-skill/update-search**, abra el archivo **update-skillset.json**. Este archivo contiene la definición JSON de un conjunto de aptitudes.
2. Revise la definición del conjunto de aptitudes. Incluye las mismas aptitudes que antes, así como una nueva aptitud de API web (**WebApiSkill**) denominada **get-top-words**.
3. Edite la definición de la aptitud **get-top-words** para establecer el valor de **uri** en la dirección URL de la función de Azure (la cual ha copiado en el Portapapeles en el procedimiento anterior); para ello, reemplace **YOUR-FUNCTION-APP-URL**.
4. En la parte superior de la definición del conjunto de aptitudes, en el elemento **cognitiveServices**, reemplace el marcador de posición **YOUR_COGNITIVE_SERVICES_KEY** por cualquiera de las claves de los recursos de Cognitive Services.

    *Puede encontrar las claves en la página **Claves y punto de conexión** del recurso de Cognitive Services en Azure Portal.*

5. Guarde y cierre el archivo JSON actualizado.
6. En la carpeta **update-search**, abra **update-index.json**. Este archivo contiene la definición JSON para el índice **margies-custom-index**, con un campo adicional denominado **top_words** en la parte inferior de la definición del índice.
7. Revise el archivo JSON del índice y ciérrelo sin realizar ningún cambio.
8. En la carpeta **update-search**, abra **update-indexer.json**. Este archivo contiene una definición JSON para **margies-custom-indexer**, con una asignación adicional para el campo **top_words**.
9. Revise el archivo JSON del indexador y ciérrelo sin realizar ningún cambio.
10. En la carpeta **update-search**, abra **update-search.cmd**. Este script por lotes usa la utilidad cURL para enviar las definiciones JSON actualizadas a la interfaz de REST del recurso de Azure Cognitive Search.
11. Reemplace los marcadores de posición de variables **YOUR_SEARCH_URL** y **YOUR_ADMIN_KEY** por la **dirección URL** y una de las **claves de administración** del recurso de Azure Cognitive Search.

    *Puede encontrar estos valores en las páginas **Información general** y **Claves** del recurso de Azure Cognitive Search en Azure Portal.*

12. Guarde el archivo por lotes actualizado.
13. Haga clic con el botón derecho en la carpeta **update-search** y seleccione **Abrir en terminal integrado**.
14. En el panel del terminal de la carpeta **update-search**, escriba el siguiente comando para ejecutar el script por lotes.

    ```
    update-search
    ```

15. Cuando se complete el script, en Azure Portal, en la página del recurso de Azure Cognitive Search, seleccione la página **Indizadores** y espere a que se complete el proceso de indexación.

    *Puede seleccionar **Actualizar** para supervisar el progreso de la operación de indexación. Puede tardar alrededor de un minuto en completarse.*

## <a name="search-the-index"></a>Búsqueda en el índice

Ahora que tiene un índice, puede realizar búsquedas en él.

1. En la parte superior del panel del recurso de Azure Cognitive Search, seleccione **Explorador de búsqueda**.
2. En el Explorador de búsqueda, en el cuadro **Cadena de consulta**, escriba la siguiente cadena de consulta y, a continuación, seleccione **Buscar**.

    ```
    search=Las Vegas&$select=url,top_words
    ```

    Esta consulta recupera los campos **url** y **top_words** de todos los documentos que mencionan *Las Vegas*.

## <a name="more-information"></a>Más información

Para obtener más información sobre cómo crear aptitudes personalizadas para Azure Cognitive Search, consulte la [documentación de Azure Cognitive Search](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface).
