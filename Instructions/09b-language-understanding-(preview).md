---
lab:
  title: Creación de un modelo de reconocimiento del lenguaje con el servicio Language (versión preliminar)
ms.openlocfilehash: 0f9055a31e42b98ddb75a35eb115ba6c8b23a3ef
ms.sourcegitcommit: 20572bfc85061bb8947fa1a5290dad8f8a71ffc8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/11/2021
ms.locfileid: "134680861"
---
# <a name="create-a-language-understanding-model-with-the-language-service-preview"></a>Creación de un modelo de reconocimiento del lenguaje con el servicio Language (versión preliminar)

> **Nota**: La característica de reconocimiento del lenguaje conversacional del servicio Language está actualmente en versión preliminar y sujeta a cambios. En algunos casos, se puede producir un error en el entrenamiento del modelo; si esto sucede, inténtelo de nuevo.  

El servicio Language permite definir un modelo de *reconocimiento del lenguaje conversacional* que las aplicaciones pueden usar para interpretar la entrada de lenguaje natural de los usuarios, predecir su *intención* (lo que quieren lograr) e identificar las *entidades* a las que se debe aplicar la intención.

Por ejemplo, es posible que se espere que un modelo de reconocimiento del lenguaje para una aplicación de reloj procese la entrada como lo siguiente:

*¿Qué hora es en Londres?*

Este tipo de entrada es un ejemplo de una *expresión* (algo que un usuario podría decir o escribir), para la que la *intención* deseada es obtener la hora en una ubicación específica (una *entidad*); en este caso, Londres.

> **Nota:** La tarea del modelo de reconocimiento del lenguaje es predecir la intención del usuario e identificar las entidades a las que se aplica la intención. <u>No</u> es tarea suya ejecutar realmente las acciones necesarias para satisfacer la intención. Por ejemplo, la aplicación de reloj puede usar una aplicación de lenguaje para distinguir que el usuario quiere saber la hora en Londres. No obstante, la propia aplicación cliente debe implementar la lógica para determinar la hora correcta y mostrarla al usuario.

## <a name="create-a-language-service-resource"></a>Creación de un recurso del servicio Language

Para crear un modelo de reconocimiento del lenguaje conversacional, necesita un recurso del **servicio Language** en una región admitida. En el momento de escribir este texto, solo se admiten las regiones Oeste de EE. UU. 2 y Oeste de Europa.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Use el botón **&#65291;Crear un recurso**, busque *language* y cree un recurso del **servicio Language** con los valores siguientes:

    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: Oeste de EE. UU. 2 u Oeste de Europa
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: Gratis (F0) (*Si este plan no está disponible, seleccione Estándar (S)*)
    - **Términos legales**: _Aceptar_ 
    - **Aviso de IA responsable**: _Aceptar_
3. Espere a que se complete la implementación y, a continuación, consulte los detalles.

## <a name="create-a-conversational-language-understanding-project"></a>Creación de un proyecto de reconocimiento del lenguaje conversacional

Ahora que ha creado un recurso de creación, puede usarlo para crear un proyecto de reconocimiento del lenguaje conversacional.

1. En una pestaña nueva del explorador, abra el portal de Language Studio en `https://language.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Si se le pide que elija un recurso de Language, seleccione la configuración siguiente:
    - **Directorio de Azure**: directorio de Azure que contiene la suscripción.
    - **Suscripción de Azure**: su suscripción a Azure.
    - **Language resource** (Recurso de Language): el recurso de lenguaje que ha creado antes.
3. Si <u>no</u> se le pide que elija un recurso de Language, puede que ya se le haya asignado otro; en tal caso, haga lo siguiente:
    1. En la barra de la parte superior de la página, haga clic en el botón **Configuración (&#9881;)**.
    2. En la página **Configuración,** vea la pestaña **Recursos**.
    3. Seleccione el recurso de Language que acaba de crear y haga clic en **Switch resource** (Cambiar recurso).
    4. En la parte superior de la página, haga clic en **Language Studio** para volver a la página principal de Language Studio.
4. En la parte superior del portal, en el menú **Crear nuevo**, seleccione **Comprensión del lenguaje de conversación**.
5. En el cuadro de diálogo **Crear un proyecto**, en la página **Elegir tipo de proyecto**, seleccione **Conversación** y haga clic en **Siguiente**.
6. En la página **Enter basic information** (Escribir información básica), escriba los detalles siguientes y haga clic en **Siguiente**:
    - **Nombre**: `Clock`
    - **Descripción**: `Natural language clock`
    - **Utterances primary language** (Idioma principal de las expresiones): inglés
    - **¿Habilitar varios idiomas en el proyecto?** : *no seleccionado*
7. En la página **Revisar y finalizar**, haga clic en **Crear**.

## <a name="create-intents"></a>Crear intenciones

Lo primero que haremos en el nuevo proyecto es definir algunas intenciones.

> **Sugerencia**: Al trabajar en el proyecto, si se muestran algunas sugerencias, léalas y haga clic en **Entendido** para descartarlas, o bien en **Omitir todo**.

1. En la página **Esquema de compilación**, en la pestaña **Intenciones**, seleccione **&#65291; Agregar** para agregar una nueva intención denominada **GetTime**.
2. Haga clic en la nueva intención **GetTime** para editarla y agregue las siguientes expresiones como entrada de usuario de ejemplo:

    `what is the time?`

    `what's the time?`

    `what time is it?`

    `tell me the time`

3. Después de agregar estas expresiones, haga clic en **Guardar cambios** y vuelva a la página **Compilar esquema**.

4. Agregue otra intención denominada **GetDay** con las siguientes expresiones:

    `what day is it?`

    `what's the day?`

    `what is the day today?`

    `what day of the week is it?`

5. Después de agregar estas expresiones y guardarlas, vuelva a la página **Esquema de compilación** y agregue otra intención denominada **GetDate** con las siguientes expresiones:

    `what date is it?`

    `what's the date?`

    `what is the date today?`

    `what's today's date?`

6. Después de agregar estas expresiones, guárdelas y borre el filtro **GetDate** en la página de expresiones para ver las expresiones de todas las intenciones.

## <a name="train-and-test-the-model"></a>Entrenamiento y prueba del modelo

Ahora que ha agregado algunas intenciones, vamos a entrenar el modelo de lenguaje y ver si puede predecirlas correctamente a partir de la entrada del usuario.

1. En el panel de la izquierda, seleccione la página **Entrenar modelo** y seleccione la opción para entrenar un nuevo modelo. Use el nombre **Clock** y asegúrese de que selecciona la opción para ejecutar la evaluación cuando se selecciona el entrenamiento. Haga clic en **Entrenar** para entrenar el modelo.
2. Una vez completado el entrenamiento (lo que puede tardar algún tiempo), consulte la página **Ver detalles del modelo** y seleccione el modelo **Clock**. A continuación, vea las métricas de evaluación generales y por intención (*precisión*, *coincidencia* y *puntuación F1*) y la *matriz de confusión* generadas por la evaluación que se realizó durante el entrenamiento. Tenga en cuenta que, debido al número reducido de expresiones de ejemplo, puede que no todas las intenciones se incluyan en los resultados.

    >**Nota**: Para obtener más información sobre las métricas de evaluación, consulte la [documentación](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/concepts/evaluation-metrics).

3. En la página **Implementar modelo**, seleccione el modelo **Clock** e impleméntelo. Esto puede llevar algo de tiempo.
4. Una vez implementado el modelo, en la página **Probar modelo**, seleccione el modelo **Clock**.
5. Escriba el texto siguiente y, después, haga clic en **Ejecutar la prueba**:

    `what's the time now?`

    Revise el resultado que se devuelve y observe que incluye la intención pronosticada (que debe ser **GetTime**) y una puntuación de confianza que indica la probabilidad que el modelo calculó para la dicha intención. En la pestaña JSON se muestra la confianza comparativa para cada posible intención (la que tiene la puntuación de confianza más alta es la intención pronosticada).

6. Borre el cuadro de texto y, a continuación, ejecute otra prueba con el texto siguiente:

    `tell me the time`

    De nuevo, revise la intención pronosticada y la puntuación de confianza.

7. Pruebe con este texto:

    `what's the day today?`

    Esperamos que el modelo prediga la intención **GetDay**.

## <a name="add-entities"></a>Agregar entidades

Hasta ahora ha definido algunas expresiones simples que se asignan a intenciones. La mayoría de las aplicaciones reales incluyen expresiones más complejas de las que se deben extraer entidades de datos específicas para obtener más contexto para la intención.

### <a name="add-a-learned-entity"></a>Adición de una entidad *aprendida*

El tipo más común de entidad es la una entidad *aprendida*, en la que el modelo aprende a identificar valores de entidad en función de ejemplos.

1. En Language Studio, vuelva a la página **Esquema de compilación** y, en la pestaña **Entidades**, seleccione **&#65291; Agregar** para agregar una nueva entidad.
2. En el cuadro de diálogo **Agregar una entidad**, escriba el nombre de la entidad **Location** y asegúrese de **Aprendida** está seleccionado. A continuación, haga clic en **Agregar entidad**.
3. Una vez creada la entidad **Location**, vuelva a la página **Esquema de compilación** y, en la pestaña **Intenciones**, seleccione la intención **GetTime**.
4. Escriba la siguiente expresión de ejemplo nueva:

    `what time is it in London?`

5. Cuando se haya agregado la expresión, seleccione la palabra ***London** _ y, en la lista desplegable que aparece, seleccione _ *Location** para indicar que "London" es un ejemplo de ubicación.
6. Agregue otra expresión de ejemplo:

    `Tell me the time in Paris?`

7. Cuando se haya agregado la expresión, seleccione la palabra ***Paris** _ y asígnela a la entidad _ *Location**.
8. Agregue otra expresión de ejemplo:

    `what's the time in New York?`

9. Cuando se haya agregado la expresión, seleccione las palabras ***New York** _ y asígnelas a la entidad _ *Location**.

10. Haga clic en **Guardar cambios** para guardar las nuevas expresiones.

### <a name="add-a-list-entity"></a>Agregar una entidad *list*

En algunos casos, los valores válidos para una entidad se pueden restringir a una lista de términos y sinónimos específicos, que pueden ayudar a la aplicación a identificar instancias de la entidad en expresiones.

1. En Language Studio, vuelva a la página **Esquema de compilación** y, en la pestaña **Entidades**, seleccione **&#65291; Agregar** para agregar una nueva entidad.
2. En el cuadro de diálogo **Agregar una entidad**, escriba el nombre de entidad **Weekday** y seleccione el tipo de entidad **Lista**. A continuación, haga clic en **Agregar entidad**.
3. En la página de la entidad **Weekday**, en la sección **Lista**, haga clic en **&#65291; Agregar nueva lista**. A continuación, escriba los siguientes valor y sinónimo y haga clic en **Guardar**:

    | Value | Sinónimos|
    |-------------------|---------|
    | sunday | sun |

4. Repita el paso anterior para agregar los siguientes componentes de lista:

    | Value | Sinónimos|
    |-------------------|---------|
    | Lunes | Lu. |
    | Martes | Tue, Tues |
    | Miércoles | Wed, Weds |
    | Jueves | Thur, Thurs |
    | Viernes | Vi. |
    | Sábado | Sá. |

5. Vuelva a la página **Esquema de compilación** y, a continuación, en la pestaña **Intenciones**, seleccione la intención **GetDate**.
6. Escriba la siguiente expresión de ejemplo nueva:

    `what date was it on Saturday?`

7. Tras agregar la expresión, seleccione la palabra ***Saturday** _ y, en la lista desplegable que aparece, seleccione _*Weekday**.
8. Agregue otra expresión de ejemplo:

    `what date will it be on Friday?`

9. Cuando se haya agregado la expresión, asigne **Friday** a la entidad **Weekday**.

10. Agregue otra expresión de ejemplo:

    `what will the be on Thurs?`

11. Cuando se haya agregado la expresión, asigne **Thurs** a la entidad **Weekday**.

12. Haga clic en **Guardar cambios** para guardar las nuevas expresiones.

### <a name="add-a-prebuilt-entity"></a>Adición de una entidad *precompilada*

El servicio Language proporciona un conjunto de entidades *precompiladas* que se usan normalmente en aplicaciones conversacionales.

1. En Language Studio, vuelva a la página **Esquema de compilación** y, en la pestaña **Entidades**, seleccione **&#65291; Agregar** para agregar una nueva entidad.
2. En el cuadro de diálogo **Agregar una entidad**, escriba el nombre de entidad **Date** y seleccione el tipo de entidad **Precompilada**. A continuación, haga clic en **Agregar entidad**.
3. En la página de la entidad **Date**, en la sección **Precompiladas**, haga  **clic en &#65291; Agregar nueva precompilada.
4. En la lista **Seleccionar precompilada**, seleccione **DateTime** y, a continuación, haga clic en **Guardar**.
5. Vuelva a la página **Esquema de compilación** y, a continuación, en la pestaña **Intenciones**, seleccione la intención **GetDay**.
6. Escriba la siguiente expresión de ejemplo nueva:

    `what day was 01/01/1901?`

7. Tras agregar la expresión, seleccione la palabra ***01/01/1901** _ y, en la lista desplegable que aparece, seleccione _*Date**.
8. Agregue otra expresión de ejemplo:

    `what day will it be on Dec 31st 2099?`

9. Cuando se haya agregado la expresión, asigne **Dec 31st 2099** a la entidad **Date**.

10. Haga clic en **Guardar cambios** para guardar las nuevas expresiones.

### <a name="retrain-the-model"></a>Nuevo entrenamiento del modelo

Ahora que ha modificado el esquema, debe volver a entrenar y probar el modo.

1. En la página **Entrenar modelo**, seleccione la opción para sobrescribir un modelo existente y especifique el modelo **Clock**. Asegúrese de que la opción para ejecutar la evaluación durante el entrenamiento esté seleccionada y haga clic en **Entrenar** para entrenar el modelo. Confirme que quiere sobrescribir el modelo existente.
2. Una vez completado el entrenamiento (lo que puede tardar algún tiempo), consulte la página **Ver detalles del modelo** y seleccione el modelo **Clock**. A continuación, vea las métricas de evaluación generales, por entidad y por intención (*precisión*, *coincidencia* y *puntuación F1*) y la *matriz de confusión* generadas por la evaluación que se realizó durante el entrenamiento. Tenga en cuenta que, debido al número reducido de expresiones de ejemplo, puede que no todas las intenciones se incluyan en los resultados.
3. En la página **Implementar modelo**, seleccione el modelo **Clock** e impleméntelo. Esto puede llevar algo de tiempo.
4. Tras implementar la aplicación, en la página **Probar modelo**, seleccione el modelo **Clock** y pruébelo con el texto siguiente:

    `what's the time in Edinburgh?`

5. Revise el resultado que se devuelve, que debería predecir la intención **GetTime** y una entidad **Location** con el valor de texto "Edinburgh".
6. Pruebe las siguientes expresiones:

    `what time is it in Tokyo?`
    
    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## <a name="use-the-model-from-a-client-app"></a>Uso del modelo desde una aplicación cliente

En un proyecto real, refinaría de forma iterativa las intenciones y las entidades, y volvería a entrenar y a probar la aplicación hasta que estuviera satisfecho con el rendimiento predictivo. Después, cuando lo haya probado y dé por bueno su rendimiento predictivo, puede usarlo en una aplicación cliente mediante una llamada a su interfaz REST. En este ejercicio, usará la utilidad *curl* para llamar al punto de conexión REST del modelo.

1. En Language Studio, en la página **Implementar modelo**, seleccione el modelo **Clock**. Después, haga clic en **Obtener dirección URL de predicción**.
2. En el cuadro de diálogo **Obtener dirección URL de predicción**, observe que la dirección URL del punto de conexión de predicción se muestra junto con una solicitud de ejemplo con el comando **curl**, que envía una solicitud HTTP POST al punto de conexión. En la solicitud se especifica la clave para el recurso de Language en el encabezado y se incluyen una consulta y un idioma.
3. Copie la solicitud de ejemplo y péguela en el editor de texto que prefiera (por ejemplo, el Bloc de notas).
4. Reemplace los marcadores de posición siguientes:
    - **YOUR_QUERY_HERE**: *What's the time in Sydney*
    - **QUERY_LANGUAGE_HERE**: *EN*

    El comando debe ser similar al código siguiente:

    ```
    curl -X POST "https://some-name.cognitiveservices.azure.com/language/:analyze-conversations?projectName=Clock&deploymentName=production&api-version=2021-11-01-preview" -H "Ocp-Apim-Subscription-Key: 0ab1c23de4f56..."  -H "Apim-Request-Id: 9zy8x76wv5u43...." -H "Content-Type: application/json" -d "{\"verbose\":true,\"query\":\"What's the time in Sydney?\",\"language\":\"EN\"}"
    ```

5. Abra un símbolo del sistema (Windows) o shell de Bash (Linux/Mac).
6. Copie y pegue el comando curl editado en la interfaz de la línea de comandos y ejecútelo.
7. Vea el JSON generado; debe incluir la intención y las entidades predichas, tal como se muestra a continuación:

    ```
    {"query":"What's the time in Sydney?","prediction":{"topIntent":"GetTime","projectKind":"conversation","intents":[{"category":"GetTime","confidenceScore":0.9998859},{"category":"GetDate","confidenceScore":9.8372206E-05},{"category":"GetDay","confidenceScore":1.5763446E-05}],"entities":[{"category":"Location","text":"Sydney","offset":19,"length":6,"confidenceScore":1}]}}
    ```

8. Revise la respuesta JSON devuelta por la aplicación, que debería indicar la intención de puntuación superior pronosticada para la entrada (que debería ser **GetTime**).
9. Cambie la consulta del comando curl a `What's today's date?` y, a continuación, ejecute y revise el JSON generado.
10. Pruebe las siguientes consultas:

    `What day will Jan 1st 2050 be?`

    `What time is it in Glasgow?`

    `What date will next Monday be?`

## <a name="export-the-project"></a>Exportación del proyecto

Puede usar Language Studio para desarrollar y probar el modelo de reconocimiento del lenguaje, pero, en un proceso de desarrollo de software para DevOps, debe mantener una definición controlada por el origen del proyecto que se pueda incluir en las canalizaciones de integración y entrega continuas (CI/CD). Aunque *puede* usar la API de REST de Language en scripts de código para crear y entrenar el modelo, una manera más sencilla es usar el portal para crear el esquema del modelo y exportarlo como un archivo *.json* que se pueda importar y volver a entrenar en otra instancia del servicio Language. Este enfoque le permite usar las ventajas de productividad de la interfaz visual de Language Studio, al tiempo que mantiene la portabilidad y reproducibilidad del modelo.

1. En Language Studio, en la página **Proyectos**, seleccione el proyecto **Clock (Conversation)** .
2. Haga clic en el botón **&#x2913; Exportar**.
3. Guarde el archivo **Clock.json** que se genera en el lugar que prefiera.
4. Abra el archivo descargado en su editor de código favorito (por ejemplo, Visual Studio Code) para revisar la definición JSON del proyecto.

## <a name="more-information"></a>Más información

Para obtener más información sobre el uso del servicio **Language** para crear soluciones de reconocimiento del lenguaje conversacional, consulte la [documentación del servicio Language](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/overview).
