---
lab:
  title: Creación de una solución de Azure Cognitive Search
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: bba5786ed34cbbe806c74e2b2eec6286cb92554b
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801378"
---
# <a name="create-an-azure-cognitive-search-solution"></a>Creación de una solución de Azure Cognitive Search

Todas las organizaciones se basan en información para tomar decisiones, responder preguntas y operar de manera eficaz. El problema de la mayoría de las organizaciones no es una falta de información, sino el reto de buscar y extraer la información del conjunto masivo de documentos, bases de datos y otros orígenes en los que se almacena.

Por ejemplo, supongamos que *Margie's Travel* es una agencia de viajes especializada en organizar viajes a ciudades de todo el mundo. Con el tiempo, la empresa ha acumulado una enorme cantidad de información en documentos, como folletos, y reseñas de hoteles enviadas por los clientes. Estos datos son una valiosa fuente de información para los agentes de viajes y los clientes a medida que planifican los viajes, pero el volumen ingente de datos puede dificultar la búsqueda de información pertinente para responder preguntas específicas de los clientes.

Para abordar este desafío, Margie's Travel puede usar Azure Cognitive Search para implementar una solución en la que los documentos se indexen y enriquezcan mediante el uso de aptitudes cognitivas basadas en inteligencia artificial para facilitar la búsqueda.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="create-azure-resources"></a>Creación de recursos de Azure

La solución que creará para Margie's Travel requiere los siguientes recursos en su suscripción de Azure:

- Un recurso de **Azure Cognitive Search** que administre el proceso de indexación y de consulta.
- Un recurso de **Cognitive Services** que proporcione servicios de inteligencia artificial para las aptitudes que la solución de búsqueda puede usar para enriquecer los datos del origen de datos con información generada mediante inteligencia artificial.
- Una **cuenta de almacenamiento** con un contenedor de blobs en el que se almacenan los documentos en los que se va a buscar.

> **Importante**: Los recursos de Azure Cognitive Search y Cognitive Services deben estar en la misma ubicación.

### <a name="create-an-azure-cognitive-search-resource"></a>Creación de un recurso de Azure Cognitive Search

1. En un explorador web, abra Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **&#65291;Crear un recurso**, busque *Search* y cree un recurso de **Azure Cognitive Search** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *cree un nuevo grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Nombre del servicio**: *escriba un nombre único*
    - **Ubicación**: *seleccione una ubicación, pero tenga en cuenta que los recursos de Azure Cognitive Search y Cognitive Services deben estar en la misma ubicación*
    - **Plan de tarifa**: básico

3. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
4. Revise la página **Información general** del panel del recurso de Azure Cognitive Search en Azure Portal. Aquí puede usar una interfaz visual para crear, probar, administrar y supervisar los distintos componentes de una solución de búsqueda, incluidos los orígenes de datos, los índices, los indexadores y los conjuntos de aptitudes.

### <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso de **Cognitive Services**. La solución de búsqueda lo usará para enriquecer los datos del almacén de datos con información generada mediante inteligencia artificial.

1. Vuelva a la página principal de Azure Portal y seleccione el botón **&amp;#65291;Crear un recurso**, busque *Cognitive Services* y cree un recurso de **Cognitive Services** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *el mismo grupo de recursos que el recurso de Azure Cognitive Search*
    - **Región**: *la misma ubicación que el recurso de Azure Cognitive Search*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
2. Active las casillas necesarias y cree el recurso.
3. Espere a que se complete la implementación y, a continuación, consulte los detalles.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

1. Vuelva a la página principal de Azure Portal y seleccione el botón **&amp;#65291;Crear un recurso**, busque *cuenta de almacenamiento* y cree un recurso de **Cuenta de almacenamiento** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: **el mismo grupo de recursos que los recursos de Azure Cognitive Search y Cognitive Services*
    - **Nombre de la cuenta de almacenamiento**: *escriba un nombre único*
    - **Región**: *elija cualquier región disponible*
    - **Rendimiento**: Estándar
    - **Replication** (Replicación): Almacenamiento con redundancia local (LRS)
2. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
3. En la página **Información general**, fíjese en el **Id. de suscripción** que identifica la suscripción en la que se aprovisiona la cuenta de almacenamiento.
4. En la página **Claves de acceso**, tenga en cuenta que se han generado dos claves para la cuenta de almacenamiento. A continuación, seleccione **Mostrar claves** para ver las claves.

    > **Sugerencia**: Mantenga abierto el panel **Cuenta de almacenamiento**, ya que necesitará el id. de suscripción y una de las claves en el procedimiento siguiente.

## <a name="upload-documents-to-azure-storage"></a>Carga de documentos en Azure Storage

Ahora que tiene los recursos necesarios, puede cargar algunos documentos en su cuenta de Azure Storage.

1. En Visual Studio Code, en el panel **Explorador**, expanda la carpeta **22-create-a-search-solution** y seleccione **UploadDocs.cmd**.
2. Edite el archivo por lotes para reemplazar los marcadores de posición **YOUR_SUBSCRIPTION_ID**, **YOUR_AZURE_STORAGE_ACCOUNT_NAME** y **YOUR_AZURE_STORAGE_KEY** por los valores adecuados de id. de suscripción, nombre de la cuenta de almacenamiento de Azure y clave de la cuenta de almacenamiento de Azure de la cuenta de almacenamiento que creó anteriormente.
3. Guarde los cambios y, a continuación, haga clic con el botón derecho en la carpeta **22-create-a-search-solution** y abra un terminal integrado.
4. Escriba el siguiente comando para iniciar sesión en su suscripción de Azure mediante la CLI de Azure.

    ```
    az login
    ```

Se abrirá una pestaña del explorador web que le solicitará que inicie sesión en Azure. Hágalo y, a continuación, cierre la pestaña del explorador y vuelva a Visual Studio Code.

5. Escriba el comando siguiente para ejecutar el archivo por lotes. De esta forma, se creará un contenedor de blobs en la cuenta de almacenamiento y se cargarán los documentos de la carpeta **data** en él.

    ```
    UploadDocs
    ```

## <a name="index-the-documents"></a>Indexación de los documentos

Ahora que tiene los documentos en su lugar, puede indexarlos para crear una solución de búsqueda.

1. En Azure Portal, navegue hasta el recurso de Azure Cognitive Search. A continuación, en la página **Información general**, seleccione **Importar datos**.
2. En la página **Conexión a los datos**, en la lista **Origen de datos**, seleccione **Azure Blob Storage**. A continuación, complete los detalles del almacén de datos con los valores siguientes:
    - **Origen de datos**: Azure Blob Storage
    - **Nombre del origen de datos**: margies-data
    - **Datos que se extraerán**: contenido y metadatos
    - **Modo de análisis**: predeterminado
    - **Cadena de conexión**: *seleccione **Elegir una conexión existente**. A continuación, seleccione la cuenta de almacenamiento y, por último, seleccione el contenedor **margies** creado mediante el script UploadDocs.cmd.*
    - **Autenticación de identidad administrada**: ninguna
    - **Nombre del contenedor**: margies
    - **Carpeta de blobs**: *dejar en blanco*
    - **Descripción**: folletos y revisiones en el sitio web de Margie's Travel
3. Continúe con el paso siguiente (*Agregar aptitudes cognitivas*).
4. En la sección **Adjuntar Cognitive Services**, seleccione el recurso de Cognitive Services.
5. En la sección **Agregar enriquecimientos**, haga lo siguiente:
    - Cambie el **Nombre del conjunto de aptitudes** a **margies-skillset**.
    - Seleccione la opción **Habilitar OCR y combinar todo el texto en el campo merged_content**.
    - Asegúrese de que el **Campo de datos de origen** está establecido en **merged_content**.
    - Deje el **Nivel de granularidad de enriquecimiento** como **Campo de origen**, que establece todo el contenido del documento que se va a indexar; pero tenga en cuenta que puede cambiar esto para extraer información en niveles más detallados, como páginas u oraciones.
    - Seleccione los siguientes campos enriquecidos:

        | Conocimiento cognitivo | Parámetro | Nombre del campo |
        | --------------- | ---------- | ---------- |
        | Extracción de nombres de ubicaciones | | locations |
        | Extracción de frases clave | | keyphrases |
        | Detectar idioma | | language |
        | Generación de etiquetas a partir de imágenes | | imageTags |
        | Generación de descripciones a partir de imágenes | | imageCaption |

6. Compruebe las selecciones (puede ser difícil cambiarlas más adelante). A continuación, proceda con el siguiente paso (*Personalización del índice de destino*).
7. Cambie el **Nombre del índice** a **margies-index**.
8. Asegúrese de que el valor de **Clave** está establecido en **metadata_storage_path**. Deje en blanco el campo **Nombre del proveedor de sugerencias**; el campo **Modo de búsqueda** debe quedar con el valor predeterminado.
9. Realice los siguientes cambios en los campos de índice y deje todos los demás campos con su configuración predeterminada (**IMPORTANTE**: es posible que tenga que desplazarse a la derecha para ver toda la tabla):

    | Nombre del campo | Retrievable | Filtrable | Ordenable | Clasificable | Buscable |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | language | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | |

11. Compruebe las selecciones y preste especial atención a que las opciones **Recuperable**, **Filtrable**, **Ordenable**, **Clasificable** y **Buscable** estén seleccionadas para cada campo (puede ser difícil cambiarlas más adelante). A continuación, proceda con el siguiente paso (*Creación de un indexador*).
12. Cambie el **Nombre del indexador** a **margies-index**.
13. Deje la **Programación** establecida en **Una vez**.
14. Expanda las opciones **Avanzadas** y asegúrese de que la opción **Claves de codificación Base 64** está seleccionada (por lo general, la codificación de claves hace que el índice sea más eficaz).
15. Seleccione **Enviar** para crear un origen de datos, un conjunto de aptitudes, un índice y un indexador. El indexador se ejecuta automáticamente y ejecuta la canalización de indexación, que hace lo siguiente:
    1. Extrae los campos de metadatos del documento y el contenido del origen de datos.
    2. Ejecuta el conjunto de aptitudes cognitivas para generar campos enriquecidos adicionales.
    3. Asigna los campos extraídos al índice.
16. En la mitad inferior de la página **Información general** del recurso de Azure Cognitive Search, consulte la pestaña **Indizadores**, que debe mostrar el elemento **margies-indexer** recién creado. Espere unos minutos y haga clic en **&orarr; Actualizar** hasta que el campo **Estado** indique que se ha realizado correctamente.

## <a name="search-the-index"></a>Búsqueda en el índice

Ahora que tiene un índice, puede realizar búsquedas en él.

1. En la parte superior de la página **Información general** del recurso de Azure Cognitive Search, seleccione **Explorador de búsqueda**.
2. En el Explorador de búsqueda, en el cuadro **Cadena de consulta**, escriba `*` (un solo asterisco) y, a continuación, seleccione **Buscar**.

    Esta consulta recupera todos los documentos del índice en formato JSON. Examine los resultados y fíjese en los campos de cada documento, que incluyen el contenido, los metadatos y los datos enriquecidos del documento extraídos mediante las aptitudes cognitivas seleccionadas.

3. Modifique la cadena de consulta a `search=*&$count=true` y envíe la búsqueda.

    Esta vez, los resultados incluyen un campo **@odata.count** en la parte superior de los resultados que indica el número de documentos devueltos por la búsqueda.

4. Pruebe la siguiente cadena de consulta:

    ```
    search=*&$count=true&$select=metadata_storage_name,metadata_author,locations
    ```

    Esta vez, los resultados incluyen solo el nombre de archivo, el autor y las ubicaciones mencionadas en el contenido del documento. El nombre de archivo y el autor están en los campos **metadata_storage_name** y **metadata_author**, que se extrajeron del documento de origen. El campo **locations** se generó mediante una aptitud cognitiva.

5. Ahora pruebe la siguiente cadena de consulta:

    ```
    search="New York"&$count=true&$select=metadata_storage_name,keyphrases
    ```

    Con esta búsqueda se buscan documentos que mencionen "New York" (Nueva York) en cualquiera de los campos en los que se pueden realizar búsquedas y devuelve el nombre de archivo y las frases clave del documento.

6. Vamos a probar una cadena de consulta más:

    ```
    search="New York"&$count=true&$select=metadata_storage_name&$filter=metadata_author eq 'Reviewer'
    ```

    Esta consulta devuelve el nombre de archivo de los documentos creados por *Reviewer* (Revisor) que mencionan "New York" (Nueva York).

## <a name="explore-and-modify-definitions-of-search-components"></a>Exploración y modificación de definiciones de componentes de búsqueda

Los componentes de la solución de búsqueda se basan en definiciones JSON, que puede consultar y editar en Azure Portal.

Aunque puede usar el portal para crear y modificar las soluciones de búsqueda, a menudo es conveniente definir los objetos de búsqueda en JSON y usar la interfaz de REST de Azure Cognitive Services para crearlas y modificarlas.

### <a name="get-the-endpoint-and-key-for-your-azure-cognitive-search-resource"></a>Obtención del punto de conexión y la clave del recurso de Azure Cognitive Search

1. En Azure Portal, vuelva a la página **Información general** del recurso Azure Cognitive Search recursos y, en la sección superior de la página, busque la **dirección URL** del recurso (que es similar a **https://resource_name.search.windows.net** ) y cópiela en el Portapapeles.
2. En Visual Studio Code, en el panel Explorador, expanda la carpeta **22-create-a-search-solution** y la subcarpeta **modify-search** y seleccione el archivo **modify-search.cmd** para abrirlo. Usará este archivo de script para ejecutar los comandos *cURL* que envían el JSON a la interfaz de REST de Azure Cognitive Services.
3. En **modify-search.cmd**, reemplace el marcador de posición **YOUR_SEARCH_URL** por la dirección URL que copió en el Portapapeles.
4. En Azure Portal, consulte la página **Claves** del recurso de Azure Cognitive Search y copie la **Clave de administrador principal** en el Portapapeles.
5. En Visual Studio Code, reemplace el marcador de posición **YOUR_ADMIN_KEY** por la clave que copió en el Portapapeles.
6. Guarde los cambios en el archivo **modify-search.cmd** (pero no lo ejecute todavía).

### <a name="review-and-modify-the-skillset"></a>Revisión y modificación del conjunto de aptitudes

1. En Visual Studio Code, en la carpeta **modify-search**, abra **skillset.json**. Se muestra una definición JSON para **margies-skillset**.
2. En la parte superior de la definición del conjunto de aptitudes, observe el objeto **cognitiveServices**, que se usa para conectar el recurso de Cognitive Services al conjunto de aptitudes.
3. En Azure Portal, abra el recurso de Cognitive Services (<u>no</u> el recurso de Azure Cognitive Search) y consulte la página **Claves**. A continuación, copie el valor de **Clave 1** en el Portapapeles.
4. En Visual Studio Code, en **skillset.json**, reemplace el marcador de posición **YOUR_COGNITIVE_SERVICES_KEY** por la clave de Cognitive Services que copió en el Portapapeles.
5. Desplácese por el archivo JSON y observe que incluye definiciones para las aptitudes que creó mediante la interfaz de usuario de Azure Cognitive Search en Azure Portal. En la parte inferior de la lista de aptitudes, se ha agregado una aptitud adicional con la siguiente definición:

    ```
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

La nueva aptitud se denomina **get-sentiment** y, para cada nivel de **documento** de un documento, evaluará el texto que se encuentre en el campo **merged_content** del documento que se está indexando (que incluye el contenido de origen, así como cualquier texto extraído de las imágenes del contenido). Usa el **idioma** extraído del documento (el valor predeterminado es el inglés) y evalúa una etiqueta de la opinión del contenido. Los valores de la etiqueta de opinión pueden ser "positive", "negative", "neutral" o "mixed". A continuación, esta etiqueta se genera como un nuevo campo denominado **sentimentLabel**.

6. Guarde los cambios realizados en **skillset.json**.

### <a name="review-and-modify-the-index"></a>Revisión y modificación del índice

1. En Visual Studio Code, en la carpeta **modify-search**, abra **index.json**. Se muestra una definición JSON para **margies-index**.
2. Desplácese por el índice y consulte las definiciones de campo. Algunos campos se basan en metadatos y contenido del documento de origen y otros son los resultados de las aptitudes del conjunto de aptitudes.
3. Tenga en cuenta que, al final de la lista de campos que definió en Azure Portal, se han agregado dos campos adicionales:

    ```
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. El campo **sentiment** se usará para agregar la salida de la aptitud **get-sentiment** que se agregó al conjunto de aptitudes. El campo **url** se usará para agregar la dirección URL de cada documento indexado al índice, en función del valor de **metadata_storage_path** extraído del origen de datos. Tenga en cuenta que el índice ya incluye el campo **metadata_storage_path**, pero se usa como clave de índice. Dicho campo está codificado en Base 64, lo que hace que sea eficaz como clave, pero requiere que las aplicaciones cliente lo descodifiquen si desean usar el valor de dirección URL real como campo. Si se agrega un segundo campo para el valor sin codificar, se resuelve este problema.

### <a name="review-and-modify-the-indexer"></a>Revisión y modificación del indexador

1. En Visual Studio Code, en la carpeta **modify-search**, abra **indexer.json**. Se muestra una definición JSON para **margies-indexer**, que asigna los campos extraídos del contenido y los metadatos del documento (en la sección **fieldMappings**) y los valores extraídos mediante las aptitudes del conjunto de aptitudes (en la sección **outputFieldMappings**) a los campos del índice.
3. En la lista **fieldMappings**, observe la asignación del valor de **metadata_storage_path** al campo de la clave codificada en Base 64. Se creó cuando asignó el campo **metadata_storage_path** como clave y seleccionó la opción para codificar la clave en Azure Portal. Además, una nueva asignación asigna explícitamente el mismo valor al campo **url**, pero sin la codificación en Base 64:

    ```
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }
    
    ```

Todos los demás campos de contenido y metadatos del documento de origen se asignan de forma implícita a campos con el mismo nombre en el índice.

4. Revise la sección **ouputFieldMappings**, que asigna las salidas de las aptitudes del conjunto de aptitudes a los campos del índice. La mayoría de ellas reflejan las opciones seleccionadas en la interfaz de usuario, pero se ha agregado la siguiente asignación para asignar el valor de **sentimentLabel** extraído mediante la aptitud de opinión al campo **sentiment** que agregó al índice:

    ```
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### <a name="use-the-rest-api-to-update-the-search-solution"></a>Uso de la API de REST para actualizar la solución de búsqueda

1. Haga clic con el botón derecho en la carpeta **modify-search** y abra un terminal integrado.
2. En el panel de terminal de la carpeta **modify-search**, escriba el siguiente comando para ejecutar el script **modify-search.cmd**, que envía las definiciones JSON a la interfaz de REST e inicia la indexación.

    ```
    modify-search
    ```

3. Cuando el script haya finalizado, vuelva a la página **Información general** del recurso de Azure Cognitive Search en Azure Portal y consulte la página **Indizadores**. Periódicamente seleccione **Actualizar** para supervisar el progreso de la operación de indexación. Puede tardar alrededor de un minuto en completarse.

    *Puede haber algunas advertencias para algunos documentos que son demasiado grandes para evaluar la opinión. A menudo, el análisis de sentimiento se realiza en el nivel de página o frase en lugar de en el documento completo. No obstante, en este escenario, la mayoría de los documentos, especialmente las reseñas de hoteles, son lo suficientemente cortos como para evaluar las puntuaciones de opinión útiles de nivel de documento.*

### <a name="query-the-modified-index"></a>Consulta del índice modificado

1. En la parte superior del panel del recurso de Azure Cognitive Search, seleccione **Explorador de búsqueda**.
2. En el Explorador de búsqueda, en el cuadro **Cadena de consulta**, escriba la siguiente cadena de consulta y, a continuación, seleccione **Buscar**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    Esta consulta recupera los valores de **url**, **sentiment** y **keyphrases** de todos los documentos que mencionan *London*, cuyo autor es *Reviewer* y que tienen una etiqueta de **opinión** positiva (es decir, reseñas positivas que mencionan "London").

3. Cierre la página **Explorador de búsqueda** para volver a la página **Información general**.

## <a name="create-a-search-client-application"></a>Creación de una aplicación cliente de búsqueda

Ahora que tiene un índice útil, puede usarlo desde una aplicación cliente. Para ello, puede consumir la interfaz de REST, enviar solicitudes y recibir respuestas en formato JSON mediante HTTP. También puede usar el kit de desarrollo de software (SDK) para su lenguaje de programación preferido. En este ejercicio, usaremos el SDK.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

### <a name="get-the-endpoint-and-keys-for-your-search-resource"></a>Obtención del punto de conexión y las claves del recurso de búsqueda

1. En Azure Portal, en la página **Información general** del recurso de Azure Cognitive Search, tome nota del valor de **Url**, que debe ser similar a **https://*your_resource_name*.search.windows.net**. Este es el punto de conexión del recurso de búsqueda.
2. En la página **Claves**, observe que hay dos claves de **administración** y una sola clave de **consulta**. Una clave de *administración* se usa para crear y administrar los recursos de búsqueda. En cambio, las aplicaciones cliente que solo necesitan realizar consultas de búsqueda utilizan una clave de *consulta*.

    *Necesitará la clave de consulta y el punto de conexión para la aplicación cliente.*

### <a name="prepare-to-use-the-azure-cognitive-search-sdk"></a>Preparación para el uso del SDK de Azure Cognitive Search

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **22-create-a-search-solution** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **margies-travel** y abra un terminal integrado. A continuación, instale el paquete del SDK de Azure Cognitive Search mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**
    
    ```
    dotnet add package Azure.Search.Documents --version 11.1.1
    ```
    
    **Python**
    
    ```
    pip install azure-search-documents==11.0.0
    ```
    
3. Consulte el contenido de la carpeta **margies-travel** y observe que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y la **clave de consulta** del recurso de Azure Cognitive Search. Guarde los cambios.

### <a name="explore-code-to-search-an-index"></a>Exploración del código para buscar en un índice

La carpeta **margies-travel** contiene archivos de código para una aplicación web (una aplicación web *ASP.NET Razor* de Microsoft C# o una aplicación *Flask* en Python), que incluye la funcionalidad de búsqueda.

1. Abra el siguiente archivo de código en la aplicación web, en función del lenguaje de programación que haya elegido:
    - **C#** : Pages/Index.cshtml.cs
    - **Python**: app.py
2. Cerca de la parte superior del archivo de código, busque el comentario **Import search namespaces** (Importar espacios de nombres de búsqueda) y observe los espacios de nombres que se han importado para trabajar con el SDK de Azure Cognitive Search:
3. En la función **search_query**, busque el comentario **Create a search client** (Crear un cliente de búsqueda) y tenga en cuenta que el código crea un objeto **SearchClient** mediante el punto de conexión y la clave de consulta del recurso de Azure Cognitive Search:
4. En la función **search_query**, busque el comentario **Sumbit search query**(Enviar consulta de búsqueda) y revise el código para enviar una búsqueda del texto especificado con las siguientes opciones:
    - Un *modo de búsqueda* que requiere que se encuentren **todas** las palabras individuales del texto de búsqueda.
    - El número total de documentos encontrados mediante la búsqueda se incluye en los resultados.
    - Los resultados se filtran para incluir solo los documentos que coinciden con la expresión de filtro proporcionada.
    - Los resultados se ordenan según el criterio de ordenación especificado.
    - Cada valor discreto del campo **metadata_author** se devuelve como una *faceta* que se puede usar para mostrar valores predefinidos para el filtrado.
    - En los resultados se incluyen hasta tres extractos de los campos **merged_content** e **imageCaption** con los términos de búsqueda resaltados.
    - Los resultados incluyen solo los campos especificados.

### <a name="explore-code-to-render-search-results"></a>Exploración del código para representar los resultados de la búsqueda

La aplicación web ya incluye código para procesar y representar los resultados de la búsqueda.

1. Abra el siguiente archivo de código en la aplicación web, en función del lenguaje de programación que haya elegido:
    - **C#** : Pages/Index.cshtml
    - **Python**: templates/search.html
2. Examine el código, que representa la página en la que se muestran los resultados de la búsqueda. Observe lo siguiente:
    - La página comienza con un formulario de búsqueda que el usuario puede usar para enviar una nueva búsqueda (en la versión de Python de la aplicación, este formulario se define en la plantilla **base.html**), al que se hace referencia al principio de la página.
    - A continuación, se representa un segundo formulario, el cual permite al usuario refinar los resultados de la búsqueda. El código de este formulario hace lo siguiente:
        - Recupera y muestra el recuento de documentos de los resultados de la búsqueda.
        - Recupera los valores de faceta del campo **metadata_author** y los muestra como una lista de opciones para filtrar.
        - Crea una lista desplegable de opciones de ordenación para los resultados.
    - A continuación, el código itera por los resultados de la búsqueda y representa cada resultado de la manera siguiente:
        - Se muestra el campo **metadata_storage_name** (nombre de archivo) como un vínculo a la dirección del campo **url**.
        - Se muestra la *información destacada* de los términos de búsqueda que se encuentran en los campos **merged_content** e **imageCaption** para ayudar a mostrar los términos de búsqueda en contexto.
        - Se muestran los campos **metadata_author**, **metadata_storage_size**, **metadata_storage_last_modified** y **language**.
        - Muestra la etiqueta de **opinión** del documento. Puede ser positiva, negativa, neutra o mixta.
        - Se muestran los cinco primeros valores de **keyphrases** (si los hay).
        - Se muestran los cinco primeros valores de **locations** (si los hay).
        - Se muestran los cinco primeros valores de **imageTags** (si los hay).

### <a name="run-the-web-app"></a>Ejecución de la aplicación web

 1. Vuelva al terminal integrado de la carpeta **margies-travel** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. En el mensaje que se muestra cuando la aplicación se inicia correctamente, siga el vínculo a la aplicación web en ejecución ( *http://localhost:5000/* o *http://127.0.0.1:5000/* ) para abrir el sitio de Margies Travel en un explorador web.
3. En el sitio web de Margie's Travel, escriba **London hotel** (hotel en Londres) en el cuadro de búsqueda y haga clic en **Search** (Buscar).
4. Revise los resultados de la búsqueda. Estos incluyen el nombre de archivo (con un hipervínculo a la dirección URL del archivo), un extracto del contenido del archivo con los términos de búsqueda (*London* [Londres] y *hotel*) resaltados, además de otros atributos del archivo de los campos de índice.
5. Observe que la página de resultados incluye algunos elementos de la interfaz de usuario que le permiten refinar los resultados. Entre ellas se incluyen las siguientes:
    - Un *filtro* basado en un valor de faceta para el campo **metadata_author**. Muestra cómo puede usar campos *clasificables* para devolver una lista de *facetas*, que son campos con un pequeño conjunto de valores discretos que se pueden mostrar como posibles valores de filtro en la interfaz de usuario.
    - La capacidad de *ordenar* los resultados en función de un campo y una dirección de ordenación especificados (ascendente o descendente). El orden predeterminado se basa en la *relevancia*, que se calcula como un valor de **search.score()** basado en un *perfil de puntuación* que evalúa la frecuencia y la importancia de los términos de búsqueda en los campos de índice.
6. Seleccione el filtro **Reviewer** (Revisor) y la opción de ordenación **Positive to negative** (De positivo a negativo) y, a continuación, seleccione **Mejorar los resultados**.
7. Observe que los resultados se filtran para incluir solo revisiones y se ordenan en función de la etiqueta de opinión.
8. En el cuadro de **búsqueda**, escriba una nueva búsqueda de **quiet hotel in New York** (hotel silencioso en Nueva York) y revise los resultados.
9. Pruebe los siguientes términos de búsqueda:
    - **Tower of London** (Torre de Londres): observe que este término se identifica como una *frase clave* en algunos documentos.
    - **skyscraper** (rascacielos): observe que esta palabra no aparece en el contenido real de ningún documento, pero se encuentra en los *títulos de imagen* y las *etiquetas de imagen* que se generaron para las imágenes de algunos documentos.
    - **Mojave desert** (Desierto de Mojave): observe que este término se identifica como una *ubicación* en algunos documentos.
10. Cierre la pestaña del explorador que contiene el sitio web de Margie's Travel y vuelva a Visual Studio Code. Después, en el terminal de Python de la carpeta **margies-travel** (donde se está ejecutando la aplicación Flask o dotnet), presione Ctrl+C para detener la aplicación.

## <a name="more-information"></a>Más información

Para obtener más información sobre Azure Cognitive Search, consulte la [documentación de Azure Cognitive Search](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
