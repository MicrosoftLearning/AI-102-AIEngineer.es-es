---
lab:
  title: Creación de un almacén de conocimiento con Azure Cognitive Search
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 8d7008ee592d4a76f24ca26a4b192c7dd5cbe9ea
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625880"
---
# <a name="create-a-knowledge-store-with-azure-cognitive-search"></a>Creación de un almacén de conocimiento con Azure Cognitive Search

Azure Cognitive Search usa una canalización de enriquecimiento de aptitudes cognitivas para extraer campos generados mediante inteligencia artificial de documentos e incluirlos en un índice de búsqueda. Aunque el índice se podría considerar el resultado principal de un proceso de indexación, los datos enriquecidos que contiene también podrían resultar útiles de otras maneras. Por ejemplo:

- Dado que el índice es esencialmente una colección de objetos JSON, cada uno de los cuales representa un registro indexado, podría ser útil exportar los objetos como archivos JSON para su integración en un proceso de orquestación de datos mediante herramientas como Azure Data Factory.
- Puede que quiera normalizar los registros del índice en un esquema relacional de tablas para fines de análisis y para la generación de informes con herramientas como Microsoft Power BI.
- Después de extraer las imágenes insertadas de los documentos durante el proceso de indexación, es posible que quiera guardar esas imágenes como archivos.

En este ejercicio, implementará un almacén de conocimiento para *Margie's Travel*, una agencia de viajes ficticia que usa la información de folletos y reseñas de hoteles para ayudar a los clientes a planificar sus viajes.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="create-azure-resources"></a>Creación de recursos de Azure

> **Nota**: Si ha completado previamente el ejercicio **[Creación de una solución de Azure Cognitive Search](22-azure-search.md)** y todavía tiene esos recursos de Azure en su suscripción, puede omitir esta sección y empezar en la sección **Creación de una solución de búsqueda**. De lo contrario, siga los pasos que se indican a continuación para aprovisionar los recursos de Azure necesarios.

1. En un explorador web, abra Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Consulte los **Grupos de recursos** de su suscripción.
3. Si usa una suscripción restringida en la que se le ha proporcionado un grupo de recursos, seleccione el grupo de recursos para ver sus propiedades. En caso contrario, cree un nuevo grupo de recursos con el nombre que prefiera y vaya a él cuando se haya creado.
4. En la página **Información general** del grupo de recursos, tome nota de los valores de **Id. de suscripción** y **Ubicación**. Los necesitará, junto con el nombre del grupo de recursos, en los pasos posteriores.
5. En Visual Studio Code, expanda la carpeta **24-knowledge-store** y seleccione **setup.cmd**. Usará este script por lotes para ejecutar los comandos de la interfaz de la línea de comandos (CLI) de Azure necesarios para crear los recursos de Azure que necesita.
6. Haga clic con el botón derecho en la carpeta **24-knowledge-store** y seleccione **Open in Integrated Terminal** (Abrir en terminal integrado).
7. En el panel de terminal, escriba el siguiente comando para establecer una conexión autenticada con su suscripción de Azure.

    ```
    az login --output none
    ```

8. Cuando se le solicite, inicie sesión en su suscripción de Azure. Después, vuelva a Visual Studio Code y espere a que se complete el proceso de inicio de sesión.
9. Ejecute el siguiente comando para enumerar las ubicaciones de Azure.

    ```
    az account list-locations -o table
    ```

10. En la salida, busque el valor de **Nombre** que corresponde a la ubicación del grupo de recursos (por ejemplo, para *Este de EE. UU.* , el nombre correspondiente es *eastus*).
11. En el script **setup.cmd**, modifique las declaraciones de las variables **subscription_id**, **resource_group** y **location** con los valores adecuados del id. de suscripción, el nombre del grupo de recursos y el nombre de la ubicación. A continuación, guarde los cambios.
12. En el terminal de la carpeta **24-create-knowledge-store**, escriba el siguiente comando para ejecutar el script:

    ```
    setup
    ```
    > **Nota**: El módulo de la CLI de búsqueda está en versión preliminar y puede dejar de responder en el proceso *- En ejecución ..* . Si sucede esto durante más de dos minutos, presione Ctrl+C para cancelar la operación de larga duración y, a continuación, seleccione **N** cuando se le pregunte si desea finalizar el script. Debería completarse correctamente.
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
- Un **conjunto de aptitudes** que defina una canalización de enriquecimiento de aptitudes para extraer campos generados mediante inteligencia artificial de los documentos. El conjunto de aptitudes también define las *proyecciones* que se generarán en el *almacén de conocimiento*.
- Un **índice** que defina un conjunto de registros de documentos en los que se pueden realizar búsquedas.
- Un **indexador** que extraiga los documentos del origen de datos, aplique el conjunto de aptitudes y rellene el índice. El proceso de indexación también conserva las proyecciones definidas en el conjunto de aptitudes en el almacén de conocimiento.

En este ejercicio, usará la interfaz de REST de Azure Cognitive Search para crear estos componentes mediante el envío de solicitudes JSON.

### <a name="prepare-json-for-rest-operations"></a>Preparación de JSON para operaciones REST

Usará la interfaz de REST para enviar definiciones JSON para los componentes de Azure Cognitive Search.

1. En Visual Studio Code, en la carpeta **24-knowledge-store**, expanda la carpeta **create-search** y seleccione **data_source.json**. Este archivo contiene una definición JSON para un origen de datos denominado **margies-knowledge-data**.
2. Reemplace el marcador de posición **YOUR_CONNECTION_STRING** por la cadena de conexión de la cuenta de almacenamiento de Azure, que debe ser similar a la siguiente:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Puede encontrar la cadena de conexión en la página **Claves de acceso** de su cuenta de almacenamiento en Azure Portal.*

3. Guarde y cierre el archivo JSON actualizado.
4. En la carpeta **create-search,** , abra **skillset.json**. Este archivo contiene una definición JSON para un conjunto de aptitudes denominado **margies-knowledge-skillset**.
5. En la parte superior de la definición del conjunto de aptitudes, en el elemento **cognitiveServices**, reemplace el marcador de posición **YOUR_COGNITIVE_SERVICES_KEY** por cualquiera de las claves de los recursos de Cognitive Services.

    *Puede encontrar las claves en la página **Keys and Endpoint** (Claves y punto de conexión) del recurso de Cognitive Services en Azure Portal.*

6. Al final de la colección de aptitudes del conjunto de aptitudes, busque la aptitud **Microsoft.Skills.Util.ShaperSkill** denominada **define-projection**. Esta aptitud define una estructura JSON para los datos enriquecidos que se usarán para las proyecciones que la canalización conservará en el almacén de conocimiento para cada documento procesado por el indexador.
7. Al final del archivo de aptitudes, puede observar que el conjunto de aptitudes también incluye una definición de **knowledgeStore**, que contiene una cadena de conexión para la cuenta de almacenamiento de Azure en la que se va a crear el almacén de conocimiento y también una colección de **proyecciones**. Este conjunto de aptitudes incluye tres *grupos de proyecciones*:
    - Un grupo que contiene una proyección de *objetos* basada en la salida de **knowledge_projection** de la aptitud de conformador en el conjunto de aptitudes.
    - Un grupo que contiene una proyección de *archivos* basada en la colección **normalized_images** de datos de imágenes extraídos de los documentos.
    - Un grupo que contiene las siguientes proyecciones de *tablas*:
        - **KeyPhrases**: contiene una columna de clave generada automáticamente y una columna **keyPhrase** asignada a la salida de la colección **knowledge_projection/key_phrases/** de la aptitud de conformador.
        - **Locations**: contiene una columna de clave generada automáticamente y una columna **location** asignada a la salida de la colección **knowledge_projection/locations/** de la aptitud de conformador.
        - **ImageTags**: contiene una columna de clave generada automáticamente y una columna **tag** asignada a la salida de la colección **knowledge_projection/image_tags/** de la aptitud de conformador.
        - **Docs**: contiene una columna de clave generada automáticamente y todos los valores de salida de **knowledge_projection** de la aptitud de conformador que todavía no están asignados a una tabla.
8. Reemplace el marcador de posición **YOUR_CONNECTION_STRING** para el valor **storageConnectionString** por la cadena de conexión de la cuenta de almacenamiento.
9. Guarde y cierre el archivo JSON actualizado.
10. En la carpeta **create-search,** , abra **index.json**. Este archivo contiene una definición JSON para un índice denominado **margies-knowledge-index**.
11. Revise el JSON del índice y ciérrelo sin realizar ningún cambio.
12. En la carpeta **create-search,** , abra **indexer.json**. Este archivo contiene una definición JSON para un indexador denominado **margies-knowledge-indexer**.
13. Revise el JSON del indexador y ciérrelo sin realizar ningún cambio.

### <a name="submit-rest-requests"></a>Envío de solicitudes REST

Ahora que ha preparado los objetos JSON que definen los componentes de la solución de búsqueda, puede enviar los documentos JSON a la interfaz REST para crearlos.

1. En la carpeta **create-search,** , abra **create-search.cmd**. Este script por lotes usa la utilidad cURL para enviar las definiciones JSON a la interfaz de REST del recurso de Azure Cognitive Search.
2. Reemplace los marcadores de posición de variables **YOUR_SEARCH_URL** y **YOUR_ADMIN_KEY** por la **dirección URL** y una de las **claves de administración** del recurso de Azure Cognitive Search.

    *Puede encontrar estos valores en las páginas **Información general** y **Claves** del recurso de Azure Cognitive Search en Azure Portal.*

3. Guarde el archivo por lotes actualizado.
4. Haga clic con el botón derecho en la carpeta **create-search** y seleccione **Open in Integrated Terminal** (Abrir en terminal integrado).
5. En el panel del terminal de la carpeta **create-search**, escriba el siguiente comando para ejecutar el script por lotes.

    ```
    create-search
    ```

6. Cuando se complete el script, en Azure Portal, en la página del recurso de Azure Cognitive Search, seleccione la página **Indizadores** y espere a que se complete el proceso de indexación.

    *Puede seleccionar **Actualizar** para supervisar el progreso de la operación de indexación. Puede tardar alrededor de un minuto en completarse.*

    > **Sugerencia**: Si se produce un error en el script, compruebe los marcadores de posición que agregó en los archivos **data_source.json** y **skillset.json**, así como el archivo **create-search.cmd**. Después de corregir los errores, puede que tenga que usar la interfaz de usuario de Azure Portal para eliminar los componentes que se crearon en el recurso de búsqueda antes de volver a ejecutar el script.

## <a name="view-the-knowledge-store"></a>Visualización del almacén de conocimiento

Después de ejecutar un indexador que usa un conjunto de aptitudes para crear un almacén de conocimiento, los datos enriquecidos extraídos mediante el proceso de indexación se conservan en las proyecciones del almacén de conocimiento.

### <a name="view-object-projections"></a>Visualización de las proyecciones de objetos

Las proyecciones de *objetos* definidas en el conjunto de aptitudes de Margie's Travel contienen un archivo JSON para cada documento indexado. Estos archivos se almacenan en un contenedor de blobs en la cuenta de Azure Storage especificada en la definición del conjunto de aptitudes.

1. En la Azure Portal, visualice la cuenta de Azure Storage que creó anteriormente.
2. Seleccione la pestaña **Explorador de Storage** (en el panel de la izquierda) para ver la cuenta de almacenamiento en la interfaz del explorador de almacenamiento en Azure Portal.
2. Expanda **CONTENEDORES DE BLOBS** para ver los contenedores de la cuenta de almacenamiento. Además del contenedor **margies** en el que se almacenan los datos de origen, debería haber dos nuevos contenedores: **margies-images** y **margies-knowledge**. Estos contenedores se han creado mediante el proceso de indexación.
3. Seleccione el contenedor **margies-knowledge**. Debe contener una carpeta para cada documento indexado.
4. Abra cualquiera de las carpetas y, después, abra el archivo **knowledge-projection.json** que contiene. Cada archivo JSON contiene una representación de un documento indexado, incluidos los datos enriquecidos extraídos por el conjunto de aptitudes, tal como se muestra aquí.

```
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

La capacidad de crear proyecciones de *objetos* como esta le permite generar objetos de datos enriquecidos que se pueden incorporar en una solución de análisis de datos empresariales; por ejemplo, mediante la introducción de los archivos JSON en una canalización de Azure Data Factory para su posterior procesamiento o su carga en un almacenamiento de datos.

### <a name="view-file-projections"></a>Visualización de las proyecciones de archivos

Las proyecciones de *archivos* definidas en el conjunto de aptitudes crean archivos JPEG para cada imagen extraída de los documentos durante el proceso de indexación.

1. En la interfaz del explorador de almacenamiento en Azure Portal, seleccione el contenedor de blobs **margies-images**. Este contenedor contiene una carpeta para cada documento que incluye imágenes.
2. Abra cualquiera de las carpetas y vea su contenido (cada una contiene al menos un archivo \*.jpg).
3. Abra cualquiera de los archivos de imagen para comprobar que contienen imágenes extraídas de los documentos.

Esta capacidad de generar proyecciones de *archivos* hace que la indexación sea una manera eficaz para extraer imágenes insertadas de un gran volumen de documentos.

### <a name="view-table-projections"></a>Visualización de las proyecciones de tablas

Las proyecciones de *tablas* definidas en el conjunto de aptitudes forman un esquema relacional de datos enriquecidos.

1. En la interfaz del explorador de almacenamiento en Azure Portal, expanda **TABLAS**.
2. Seleccione la tabla **Docs** para ver sus columnas. Se incluyen algunas columnas estándar de las tablas de Azure Storage, para ocultarlas, modifique las **opciones de columna** y seleccione solo las columnas siguientes:
    - **document_id**: la columna de clave generada automáticamente por el proceso de indexación.
    - **file_id**: la dirección URL del archivo codificado.
    - **file_name**: el nombre de archivo extraído de los metadatos del documento.
    - **language**: el idioma en el que está escrito el documento.
    - **sentiment**: la puntuación de opinión calculada para el documento.
    - **url**: la dirección URL del blob del documento en Azure Storage.
3. Vea las otras tablas que se han creado mediante el proceso de indexación:
    - **ImageTags**: contiene una fila por cada etiqueta de imagen y la columna **document_id** para el documento en el que aparece la etiqueta.
    - **KeyPhrases**: contiene una fila por cada frase clave y la columna **document_id** para el documento en el que aparece la frase.
    - **Locations**: contiene una fila para cada ubicación y la columna **document_id** para el documento en el que aparece la ubicación.

La capacidad de crear proyecciones de *tablas* le permite crear soluciones de análisis y generación de informes que consultan el esquema relacional; por ejemplo, con Microsoft Power BI. Las columnas de clave generadas automáticamente se pueden usar para combinar las tablas en las consultas; por ejemplo, para devolver todas las ubicaciones mencionadas en un documento específico.

## <a name="more-information"></a>Más información

Para obtener más información sobre cómo crear almacenes de conocimiento para Azure Cognitive Search, consulte la [documentación de Azure Cognitive Search](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro).
