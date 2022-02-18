---
lab:
  title: Creación de una solución de respuesta a preguntas
  module: Module 6 - Building a QnA Solution
ms.openlocfilehash: 786d4c30b4b4f85b5c85a7a9b8d500a497bb832d
ms.sourcegitcommit: 6c1ad9a67d6caadcfb7edb4f58e314eab306f720
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/09/2021
ms.locfileid: "134463605"
---
# <a name="create-a-question-answering-solution"></a>Creación de una solución de respuesta a preguntas

Uno de los escenarios de conversación más comunes es proporcionar soporte técnico a través de knowledge bases de preguntas más frecuentes (P+F). Muchas organizaciones publican preguntas más frecuentes como documentos o páginas web, lo que funciona bien para un pequeño conjunto de pares de preguntas y respuestas, pero los documentos grandes pueden ser difíciles y lentos de buscar.

El servicio **Language** incluye una capacidad de *respuesta a preguntas* que permite crear una knowledge base de pares de preguntas y respuestas consultables mediante la entrada de lenguaje natural. Normalmente, esta capacidad se usa como un recurso que un bot puede emplear para buscar respuestas a las preguntas enviadas por los usuarios.

> **Nota**: La capacidad de respuesta a preguntas del servicio Language es una versión más reciente del servicio QnA Maker, que todavía está disponible como servicio independiente.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="create-a-language-resource"></a>Creación de un recurso de Language

Si quiere crear y hospedar una knowledge base para responder a preguntas, necesita tener un recurso del **servicio Language** en su suscripción de Azure.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Use el botón **&#65291;Crear un recurso**, busque *Language* y cree un recurso del **servicio Language**.
3. Haga clic en **Seleccionar** en el bloque **Respuesta personalizada a preguntas**. Después, haga clic en **Continuar** para crear el recurso. Deberá especificar la siguiente configuración:
    
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier ubicación disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: Estándar S.
    - **Ubicación de Azure Search**\*: *elija una ubicación en la misma región global que el recurso de Language*.
    - **Plan de tarifa de Azure Search**: Gratis (F) (*Si este plan no está disponible, seleccione Básico (B)* )
    - **Términos legales**: _Aceptar_ 
    - **Aviso de IA responsable**: _Aceptar_
    
    \*Respuestas a preguntas personalizadas usa Azure Search para indexar y consultar la knowledge base de preguntas y respuestas.

4. Espere a que se complete la implementación y, a continuación, vea los detalles de la implementación.

## <a name="create-a-question-answering-project"></a>Creación de un proyecto de respuesta a preguntas

Si quiere crear una knowledge base para responder a preguntas en el recurso de Language, puede usar el portal de Language Studio para crear un proyecto de respuesta a preguntas. En este caso, creará una knowledge base que contiene preguntas y respuestas sobre [Microsoft Learn](https://docs.microsoft.com/learn).

1. En una pestaña nueva del explorador, vaya el portal de *Language Studio* (`https://language.azure.com`) e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Si se le pide que elija un recurso de Language, seleccione la configuración siguiente:
    - **Directorio de Azure**: directorio de Azure que contiene la suscripción.
    - **Suscripción de Azure**: su suscripción a Azure.
    - **Language resource** (Recurso de Language): el recurso de lenguaje que ha creado antes.
3. Si <u>no</u> se le pide que elija un recurso de Language, puede deberse a que tiene varios recursos de Language en la suscripción, en cuyo caso:
    1. En la barra de la parte superior de la página, haga clic en el botón **Configuración (&#9881;)**.
    2. En la página **Configuración,** vea la pestaña **Recursos**.
    3. Seleccione el recurso de Language que acaba de crear y haga clic en **Switch resource** (Cambiar recurso).
    4. En la parte superior de la página, haga clic en **Language Studio** para volver a la página principal de Language Studio.
3. En la parte superior del portal, en el menú **Crear nuevo**, seleccione **Respuesta a preguntas personalizada**.
4. En el Asistente para ***creación de un proyecto**, en la página **Elegir valor de idioma**, seleccione la opción para establecer el idioma de todos los proyectos de este recurso y elija **Inglés**. A continuación, haga clic en **Siguiente**.
5. En la página **Escribir información básica**, escriba los detalles siguientes y haga clic en **Siguiente**:
    - **Nombre**: LearnFAQ
    - **Descripción**: Preguntas más frecuentes sobre Microsoft Learn
    - **Respuesta predeterminada cuando no se devuelve ninguna respuesta**: No se entiende la pregunta.
6. En la página **Revisar y finalizar**, haga clic en **Crear proyecto**.

## <a name="add-a-sources-to-the-knowledge-base"></a>Adición de orígenes a la knowledge base

Puede crear una knowledge base desde cero, pero es habitual empezar importando preguntas y respuestas desde una página o documento de preguntas frecuentes existente. En este caso, importará datos desde una página web de preguntas más frecuentes existente para Microsoft Learn. También importará algunas preguntas y respuestas predefinidas de tipo charla para admitir intercambios de conversacionales comunes.

1. En la página **Administrar orígenes** del proyecto de respuesta a preguntas, en la lista **&#9547; Agregar origen**, seleccione **Direcciones URL**. A continuación, en el cuadro de diálogo **Agregar direcciones URL**, haga clic en **&#9547; Agregar dirección URL** y agregue la siguiente dirección URL. Después, haga clic en **Agregar todo** para agregarla a la knowledge base:
    - **Nombre**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
2. En la página **Administrar orígenes** del proyecto de respuesta a preguntas, en la lista **&#9547; Agregar origen**, seleccione **Charla**. En el cuadro de diálogo **Agregar charla**, seleccione **Amistosa** y haga clic en **Agregar charla**.

## <a name="edit-the-knowledge-base"></a>Edición de la base de conocimiento

La knowledge base se ha rellenado con pares de preguntas y respuestas de las preguntas más frecuentes de Microsoft Learn, complementadas con un conjunto de pares de preguntas y respuestas de *charla* conversacional. Puede ampliar la knowledge base agregando pares de preguntas y respuestas adicionales.

1. En el proyecto **LearnFAQ** de Language Studio, seleccione la página **Editar knowledge base** para ver los pares de pregunta y respuesta existentes. Si se muestran algunas sugerencias, léalas y haga clic en **Entendido** para descartarlas, o bien en **Omitir todo**.
2. En la knowledge base, seleccione **&#65291; Agregar par de pregunta**.
3. En el cuadro **Pregunta**, escriba `What is Microsoft certification?` y presione **Entrar****.
4. Seleccione **&#65291; Agregar expresiones alternativas**, escriba `How can I demonstrate my Microsoft technology skills?` y presione **Entrar**.
5. En el cuadro **Respuesta**, escriba `The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.` y presione **Entrar**. Haga clic en **Enviar** para agregar la pregunta, incluida la expresión alternativa, y la respuesta a la knowledge base.

    En algunos casos, tiene sentido permitir que el usuario realice un seguimiento de la respuesta para crear una conversación *multiturno* que permita al usuario refinar iterativamente la pregunta para llegar a la respuesta que necesita.

6. En la respuesta que escribió para la pregunta de certificación, seleccione **&#65291; Agregar solicitudes de seguimiento**.
7. En el cuadro de diálogo **Solicitud de seguimiento**, indique la siguiente configuración y haga clic en **Agregar solicitud**:
    - **Texto que se muestra en el símbolo del sistema al usuario**: Más información sobre la certificación
    - Seleccione **Crear vínculo a nuevo par** y escriba este texto: `You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - **Mostrar solo en el flujo contextual**: seleccionado *Esta opción garantiza que la respuesta solo se devuelve en el contexto de una pregunta de seguimiento a partir de la pregunta de certificación original.*

## <a name="train-and-test-the-knowledge-base"></a>Entrenamiento y prueba de la base de conocimiento

Ahora que tiene una base de conocimiento, puede probarla en el portal de QnA Maker.

1. En la parte superior de la página, haga clic en **Guardar cambios**.
2. Una vez guardados los cambios, haga clic en **Probar** para abrir el panel de pruebas.
3. En el panel de prueba, en la parte superior, *anule la selección* de la opción para mostrar respuestas cortas. A continuación, en la parte inferior, escriba el mensaje `Hello`. Debería devolver una respuesta adecuada.
4. En el panel de prueba, en la parte inferior, escriba el mensaje `What is Microsoft Learn?`. Se debe devolver una respuesta adecuada de las preguntas más frecuentes.
5. Escriba el mensaje `Thanks!`. Debería devolverse una respuesta de charla adecuada.
6. Escriba el mensaje `Tell me about certification`. Debería devolverse la respuesta que creó junto con un vínculo de solicitud de seguimiento.
7. Seleccione el vínculo de seguimiento **Más información sobre la certificación**. Debería devolverse la respuesta de seguimiento con un vínculo a la página de certificación.
8. Cuando haya terminado de probar la knowledge base, cierre el panel de prueba.

## <a name="deploy-and-test-the-knowledge-base"></a>Implementación y prueba de la knowledge base

La base de conocimiento proporciona un servicio back-end que las aplicaciones cliente pueden usar para responder preguntas. Ahora ya está listo para publicar la knowledge base y acceder a su interfaz REST desde un cliente.

1. En el proyecto **LearnFAQ** de Language Studio, seleccione la página **Implementar knowledge base**.
2. Haga clic en **Implementar** en la parte superior de la página. A continuación, haga clic en **Implementar** para confirmar que quiere implementar la knowledge base.
3. Una vez completada la implementación, haga clic en **Obtener dirección URL de predicción** para ver el punto de conexión REST de la knowledge base y cópielo en el Portapapeles, pero no cierre el cuadro de diálogo todavía.
4. En Visual Studio Code, en la carpeta **12-qna**, abra **ask-question.cmd**. Este script usa *Curl* para llamar a la interfaz REST de un punto de conexión de respuesta a preguntas.
5. En el script, reemplace *YOUR_PREDICTION_ENDPOINT* por el punto de conexión de predicción que copió (asegurándose de que esté entre comillas).
6. Vuelva al explorador y, en el cuadro de diálogo **Obtener dirección URL de predicción**, observe que la solicitud de ejemplo incluye un valor para el parámetro **Ocp-Apim-Subscription-Key** similar a *ab12c345de678fg9hijk01lmno2pqrs34*. Esta es la clave de autorización para el recurso. Cópielo en el Portapapeles y, a continuación, haga clic en **Entendido** para cerrar el cuadro de diálogo.
7. Vuelva a Visual Studio Code y, en el script **ask-question.cmd**, reemplace *YOUR_KEY* por la clave que copió (asegurándose de que esté entre comillas).
8. Observe que el comando Curl del script envía un parámetro **question** con el valor **What is a Learning Path?** .
9. Compruebe que todo el script sea similar al código siguiente y guarde el archivo.

    ```
    @echo off
    SETLOCAL ENABLEDELAYEDEXPANSION

    rem Set variables
    set prediction_url="https://some-name.cognitiveservices.azure.com/language/......."
    set key="ab12c345de678fg9hijk01lmno2pqrs34"

    curl -X POST !prediction_url! -H "Ocp-Apim-Subscription-Key: !key!" -H "Content-Type: application/json" -d "{'question': 'What is a learning Path?' }"
    ```

10. En Visual Studio Code, en el panel Explorador, haga clic con el botón derecho en el script **ask-question.cmd** y seleccione **Abrir en terminal integrado**.
11. En el panel del terminal, escriba el comando `ask-question.cmd` para ejecutar el script y ver la respuesta JSON que devuelve el servicio, que debería contener una respuesta adecuada a la pregunta *What is a learning path?* .

## <a name="create-a-bot-for-the-knowledge-base"></a>Creación de un bot para la base de conocimiento

Normalmente, las aplicaciones cliente que se usan para recuperar respuestas de una knowledge base son bots.

1. Vuelva a Language Studio en el explorador y, en la página **Implementar knowledge base**, seleccione **Crear bot**. Se abrirá Azure Portal en una nueva pestaña del explorador para que pueda crear un bot de aplicación web en su suscripción de Azure. Si se le pide que inicie sesión, hágalo.
2. En Azure Portal, cree un bot de aplicación web con la siguiente configuración (la mayoría de las opciones se rellenarán previamente):

    *Si faltan algunos valores, actualice el explorador.*  

  - **Identificador de bot**: *nombre único para el bot*.
  - **Suscripción**: *suscripción de Azure*
  - **Grupo de recursos**: *grupo de recursos que contiene el recurso de Language*
  - **Ubicación**: *la misma que la del servicio Text Analytics*.
  - **Plan de tarifa**: F0.
  - **Nombre de aplicación**: *el mismo que el **identificador de bot** con un id. único y *.azurewebsites.net* anexado automáticamente
  - **Lenguaje del SDK**: *elija C# o Node.js*.
  - **Clave de autenticación de QnA**: *se debe establecer automáticamente como la clave de autenticación de la base de conocimiento de QnA*.
  - **Plan o ubicación de App Service**: *se puede establecer automáticamente en un plan y una ubicación adecuados si existe uno. Si no es así, cree un nuevo plan*
  - **Application Insights**: desactivado.
  - **Id. y contraseña de la aplicación de Microsoft**: cree automáticamente el identificador y la contraseña de la aplicación.
3. Espere a que se cree el bot. A continuación, haga clic en **Ir al recurso**. Otra opción es hacer clic en **Grupos de recursos** en la página principal, abrir el grupo de recursos en el que creó el bot de aplicación web y hacer clic en este último.
4. En la hoja de su bot, vaya a la página **Test in Web Chat** y espere hasta que el bot muestre el mensaje **Hello and welcome!** (puede tardar unos segundos en inicializarse).
5. Use la interfaz de chat de prueba para asegurarse de que el bot responde a las preguntas de la base de conocimiento según lo previsto. Por ejemplo, pruebe a enviar `What is Microsoft certification?`.

## <a name="more-information"></a>Más información

Para obtener más información sobre las respuestas a preguntas en el servicio Language, consulte la [documentación del servicio Language](https://docs.microsoft.com/en-us/azure/cognitive-services/language-service/question-answering/overview).
