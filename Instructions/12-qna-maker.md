---
lab:
  title: Crear una solución de QnA
  module: Module 6 - Building a QnA Solution
ms.openlocfilehash: 15b1b41d16f389392315f31c3daba81bc18101c9
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625980"
---
# <a name="create-a-qna-solution"></a>Crear una solución de QnA

Uno de los escenarios de conversación más comunes es proporcionar soporte técnico a través de knowledge bases de preguntas más frecuentes (P+F). Muchas organizaciones publican preguntas más frecuentes como documentos o páginas web, lo que funciona bien para un pequeño conjunto de pares de preguntas y respuestas, pero los documentos grandes pueden ser difíciles y lentos de buscar.

QnA Maker es un servicio cognitivo que permite crear una knowledge base de pares de preguntas y respuestas que se pueden consultar mediante la entrada de lenguaje natural y que se usa normalmente como un recurso que un bot puede usar para buscar respuestas a las preguntas enviadas por los usuarios.

En este laboratorio, usaremos Managed QnA Maker, que es una característica dentro de Text Analytics. 

## <a name="create-a-text-analytics-resource"></a>Creación de un recurso de Text Analytics 

Para crear y hospedar una knowledge base mediante Managed QnA Maker, necesita un recurso de Text Analytics en la suscripción de Azure.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **Crear un recurso**, busque *Text Analytics* y cree un recurso de **Text Analytics**. 
3. Haga clic en **Seleccionar** en el bloque **Respuesta personalizada de preguntas (versión preliminar)** . Después, haga clic en **Continuar** para crear el recurso. Deberá especificar la siguiente configuración:
    
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier ubicación disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: Estándar S.
    - **Ubicación de Azure Search**\*: *elija una ubicación en la misma región global que el recurso de QnA Maker*.
    - **Plan de tarifa de Azure Search**: Gratis (F) (*Si este plan no está disponible, seleccione Básico (B)* )
    - **Términos legales**: _Aceptar_ 
    - **Aviso de IA responsable**: _Aceptar_
    
    \*Respuestas a preguntas personalizadas usa Azure Search para indexar y consultar la knowledge base de preguntas y respuestas.

4. Espere a que se complete la implementación y, a continuación, vea los detalles de la implementación.

## <a name="create-a-knowledge-base"></a>Creación de una base de conocimientos

Para crear una knowledge base en el recurso de Text Analytics, puede usar el portal de QnA Maker. En este caso, creará una knowledge base que contiene preguntas y respuestas sobre [Microsoft Learn](https://docs.microsoft.com/learn).

1. En una nueva pestaña del explorador, vaya al portal de QnA Maker en `https://qnamaker.ai` y regístrese con la cuenta Microsoft asociada a su suscripción de Azure.
2. En la parte superior del portal, seleccione **Creación de una base de conocimiento**.
3. Ya ha creado un recurso de QnA Maker, por lo que puede omitir el paso 1. En la sección **Paso 2**, seleccione la siguiente configuración:
    - **Id. de directorio de Microsoft Azure**: directorio de Azure que contiene la suscripción.
    - **Nombre de suscripción de Azure**: su suscripción de Azure.
    - **Servicio Azure QnA**: el recurso de Text Analytics que creó anteriormente.
    - **Idioma**: inglés (*de forma predeterminada, esta opción solo está disponible para la primera knowledge base que crea*).
4. En la sección **Paso 3**, escriba **Preguntas frecuentes sobre Learn** como nombre de la knowledge base.

    Puede crear una knowledge base desde cero, pero es habitual empezar importando preguntas y respuestas desde una página o documento de preguntas frecuentes existente.

5. En la sección **Paso 4**:
    - En el cuadro **URL**, escriba `https://docs.microsoft.com/en-us/learn/support/faq` y haga clic en **Agregar dirección URL**.
    - En la sección **Charla**, seleccione **Descriptiva**.
6. En la sección **Paso 5**, seleccione **Create your KB** (Crear la knowledge base) y espere a que se cree la knowledge base.

## <a name="modify-the-knowledge-base"></a>Modificar una knowledge base

La knowledge base se ha rellenado con pares de preguntas y respuestas de las preguntas más frecuentes de Microsoft Learn, complementadas con un conjunto de pares de preguntas y respuestas de *charla* conversacional. Puede ampliar la knowledge base agregando pares de preguntas y respuestas adicionales.

1. En la knowledge base, seleccione **+Adición de un par de QnA**.
2. En el cuadro **Pregunta**, escriba `What is Microsoft certification?`
3. Seleccione **+ Add alternative phrasing** (+ Adición de expresiones alternativas) y escriba `How can I demonstrate my Microsoft technology skills?`.
4. En el cuadro **Respuesta**, escriba `The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.`

    En algunos casos, tiene sentido permitir que el usuario realice un seguimiento de la respuesta para crear una conversación *multiturno* que permita al usuario refinar iterativamente la pregunta para llegar a la respuesta que necesita.

5. En la respuesta que escribió para la pregunta de certificación, seleccione **+ Add follow-up prompt** (Adición de solicitud de seguimiento).
6. En el cuadro de diálogo **Solicitud de seguimiento**, escriba la siguiente configuración:
    - **Texto para mostrar**: `Learn more about certification.`
    - **Link to QnA**\* (Vincular a QnA): `You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - **Solo contexto**: seleccionado. *Esta opción garantiza que la respuesta solo se devuelve en el contexto de una pregunta de seguimiento a partir de la pregunta de certificación original.*

    \*Al escribir en el cuadro **Link to QnA** (Vincular a QnA) se buscan las respuestas existentes en la knowledge base. Cuando no se encuentra ninguna coincidencia, se establece el valor predeterminado y crea un nuevo par de pregunta y respuesta. Tenga en cuenta que el texto que escriba aquí está en formato Markdown.

## <a name="train-and-test-the-knowledge-base"></a>Entrenamiento y prueba de la base de conocimiento

Ahora que tiene una base de conocimiento, puede probarla en el portal de QnA Maker.

1. En la parte superior derecha de la página, haga clic en **Save and train** (Guardar y entrenar) para entrenar la base de conocimiento.
2. Una vez completado el entrenamiento, haga clic en **&larr;Probar** para abrir el panel de pruebas.
3. En el panel de prueba, en la parte superior, *anule la selección* del cuadro que indica *Mostrar respuesta corta*. A continuación, en la parte inferior, escriba el mensaje *Hello*. Debería devolver una respuesta adecuada.
4. En el panel de prueba, en la parte inferior, escriba el mensaje *What is Microsoft Learn?* . Se debe devolver una respuesta adecuada de las preguntas más frecuentes.
5. Escriba el mensaje *That makes me happy!* Debería devolverse una respuesta de charla adecuada.
6. Escriba el mensaje *Tell me about certification*. Debería devolverse la respuesta que creó junto con un botón de solicitud de seguimiento.
7. Seleccione el botón de seguimiento **Más información sobre la certificación**. Debería devolverse la respuesta de seguimiento con un vínculo a la página de certificación.
8. Cuando haya terminado de probar el la base de conocimiento, haga clic en **&rarr; Probar** para cerrar el panel de prueba.

## <a name="publish-the-knowledge-base"></a>Publicación de la base de conocimiento

La base de conocimiento proporciona un servicio back-end que las aplicaciones cliente pueden usar para responder preguntas. Ahora ya está listo para publicar la knowledge base y acceder a su interfaz REST desde un cliente.

1. En la parte superior de la página de QnA Maker, haga clic en **Publicar**. A continuación, en la página **Preguntas frecuentes sobre Learn**, haga clic en **Publicar**.
2. Una vez completada la publicación, vea el código de ejemplo que se proporciona para usar el punto de conexión REST de la knowledge base. Hay un ejemplo para *Postman* y un ejemplo para *Curl*.
3. Vea la pestaña **Curl** y copie el código de ejemplo.
4. Inicie Visual Studio Code y abra un panel de terminal.
5. Pegue el código que copió en el terminal y, a continuación, edítelo para reemplazar **&lt;la pregunta&gt;** por **What is a learning path?** .
6. Escriba el comando y vea la respuesta JSON que se devuelve desde la knowledge base.

## <a name="create-a-bot-for-the-knowledge-base"></a>Creación de un bot para la base de conocimiento

Normalmente, las aplicaciones cliente que se usan para recuperar respuestas de una knowledge base son bots.

1. En la página que contiene la confirmación de publicación y el código Curl de ejemplo, seleccione **Crear bot**. Se abrirá Azure Portal en una nueva pestaña del explorador para que pueda crear un bot de aplicación web en su suscripción de Azure.
2. En Azure Portal, cree un bot de aplicación web con la siguiente configuración (la mayoría de las opciones se rellenarán previamente):

    *Si faltan algunos valores, actualice el explorador.*  

  - **Identificador de bot**: *nombre único para el bot*.
  - **Suscripción**: *suscripción de Azure*
  - **Grupo de recursos**: *grupo de recursos que contiene el recurso de QnA Maker*.
  - **Ubicación**: *la misma que la del servicio Text Analytics*.
  - **Plan de tarifa**: F0.
  - **Nombre de aplicación**: *el mismo que el **Identificador de bot** con *.azurewebsites.net* agregado automáticamente
  - **Lenguaje del SDK**: *elija C# o Node.js*.
  - **Clave de autenticación de QnA**: *se debe establecer automáticamente como la clave de autenticación de la base de conocimiento de QnA*.
  - **Plan o ubicación de App Service**: *se puede establecer automáticamente en un plan y una ubicación adecuados si existe uno. Si no es así, cree un nuevo plan*
  - **Application Insights**: desactivado.
  - **Id. y contraseña de la aplicación de Microsoft**: cree automáticamente el identificador y la contraseña de la aplicación.
3. Espere a que se cree el bot (el icono de notificación de la parte superior derecha, que parece una campana, se moverá mientras espera). A continuación, en el mensaje que informa sobre la finalización de la implementación, haga clic en **Ir al recurso** (otra opción es hacer clic en **Grupos de recursos** en la página principal, abrir el grupo de recursos en el que creó el bot de aplicación web y hacer clic en él).
4. En la hoja de su bot, vaya a la página **Test in Web Chat** y espere hasta que el bot muestre el mensaje **Hello and welcome!** (puede tardar unos segundos en inicializarse).
5. Use la interfaz de chat de prueba para asegurarse de que el bot responde a las preguntas de la base de conocimiento según lo previsto. Por ejemplo, pruebe a enviar *What is Microsoft certification?* .

## <a name="more-information"></a>Más información

Para obtener más información sobre QnA Maker, consulte la [documentación QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/).