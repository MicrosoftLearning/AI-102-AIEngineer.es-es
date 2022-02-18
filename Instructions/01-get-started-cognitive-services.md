---
lab:
  title: Introducción a Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 4baba38b03c6d7bb5fe04fa5e73bb606e970550b
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801357"
---
# <a name="get-started-with-cognitive-services"></a>Introducción a Cognitive Services

En este ejercicio, podrá empezar a trabajar con Cognitive Services mediante la creación de un recurso de **Cognitive Services** en la suscripción de Azure y su uso desde una aplicación cliente. El objetivo del ejercicio no es adquirir experiencia en ningún servicio determinado, sino familiarizarse con un patrón general de aprovisionamiento y trabajo con Cognitive Services como desarrollador.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="provision-a-cognitive-services-resource"></a>Aprovisionamiento de un recurso de Cognitive Services

Azure Cognitive Services son servicios basados en la nube que encapsulan funcionalidades de inteligencia artificial que se pueden incorporar a las aplicaciones. Puede aprovisionar recursos individuales de Cognitive Services para API específicas (por ejemplo, **Language** o **Computer Vision**). También puede aprovisionar un recurso general de **Cognitive Services** que proporcione acceso a varias API de Cognitive Services mediante una clave y punto de conexión únicos. En este caso, usará un único recurso de **Cognitive Services**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **+Crear un recurso**, busque *Cognitive Services* y cree un recurso de **Cognitive Services** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
3. Active las casillas necesarias y cree el recurso.
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Vaya al recurso y consulte su página **Claves y punto de conexión**. Esta página contiene la información que necesitará para conectarse al recurso y usarlo desde las aplicaciones que desarrolle. Concretamente:
    - Un *punto de conexión* HTTP al que las aplicaciones cliente pueden enviar solicitudes.
    - Dos *claves* que se pueden usar para la autenticación (las aplicaciones cliente pueden usar cualquiera de las claves para autenticarse).
    - La *ubicación* en la que se hospeda el recurso. Esta información es necesaria para las solicitudes a algunas API (pero no a todas).

## <a name="use-a-rest-interface"></a>Uso de la interfaz de REST

Las Cognitive Services APIs se basan en REST, por lo que se pueden consumir mediante el envío de solicitudes JSON mediante HTTP. En este ejemplo, explorará una aplicación de consola que usa la API de REST **Language** para realizar la detección de idioma, pero el principio básico es el mismo para todas las API que admite el recurso de Cognitive Services.

> **Nota**: En este ejercicio, puede elegir usar la API de REST de **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **01-getting-started** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Consulte el contenido de la carpeta **rest-client** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Cognitive Services. Guarde los cambios.
4. Tenga en cuenta que la carpeta **rest-client** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: rest-client.py

    Abra el archivo de código y revise el código que contiene, fijándose en los siguientes detalles:
    - Se importan varios espacios de nombres para habilitar la comunicación HTTP.
    - El código de la función **Main** recupera el punto de conexión y la clave del recurso de Cognitive Services, que se usarán para enviar solicitudes de REST al servicio Text Analytics.
    - El programa acepta la entrada del usuario y usa la función **GetLanguage** para llamar a la API de REST de detección de idioma de Text Analytics para que el punto de conexión de Cognitive Services detecte el idioma del texto introducido.
    - La solicitud enviada a la API consta de un objeto JSON que contiene los datos de entrada; en este caso, una colección de objetos de **documentos**, cada uno de los cuales tiene un **id.** y **texto**.
    - La clave del servicio se incluye en el encabezado de solicitud para autenticar la aplicación cliente.
    - La respuesta del servicio es un objeto JSON que la aplicación cliente puede analizar.
5. Haga clic con el botón derecho en la carpeta **rest-client** y abra un terminal integrado. A continuación, escriba el siguiente comando específico del lenguaje para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python rest-client.py
    ```

6. Cuando se le solicite, escriba algún texto y revise el idioma que detecta el servicio, que se devuelve en la respuesta JSON. Por ejemplo, pruebe a escribir "Hello", "Bonjour" y "Hola".
7. Cuando haya terminado de probar la aplicación, escriba "quit" (salir) para detener el programa.

## <a name="use-an-sdk"></a>Uso de un SDK

Puede escribir código que consuma las API de REST de Cognitive Services directamente, pero hay kits de desarrollo de software (SDK) para muchos lenguajes de programación populares, incluidos Microsoft C#, Python y Node.js. El uso de un SDK puede simplificar considerablemente el desarrollo de aplicaciones que consuman Cognitive Services.

1. En Visual Studio Code, en el panel **Explorador**, en la carpeta **01-getting-started** expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **sdk-client** y abra un terminal integrado. A continuación, instale el paquete del SDK de Text Analytics mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    ```

3. Consulte el contenido de la carpeta **sdk-client** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Cognitive Services. Guarde los cambios.
    
4. Tenga en cuenta que la carpeta **sdk-client** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: sdk-client.py

    Abra el archivo de código y revise el código que contiene, fijándose en los siguientes detalles:
    - Se importa el espacio de nombres del SDK que instaló.
    - El código de la función **Main** recupera el punto de conexión y la clave del recurso de Cognitive Services, que se usarán con el SDK a fin de crear un cliente para el servicio Text Analytics.
    - La función **GetLanguage** usa el SDK para crear un cliente para el servicio y, a continuación, usa el cliente para detectar el idioma del texto introducido.
5. Vuelva al terminal integrado de la carpeta **sdk-client** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```

6. Cuando se le solicite, escriba algún texto y revise el idioma que detecta el servicio. Por ejemplo, pruebe a escribir "Adiós", "Au revoir" y "Hasta la vista".
7. Cuando haya terminado de probar la aplicación, escriba "quit" (salir) para detener el programa.

> **Nota**: Es posible que algunos idiomas que requieren juegos de caracteres Unicode no se reconozcan en esta aplicación de consola sencilla.

## <a name="more-information"></a>Más información

Para obtener más información sobre Azure Cognitive Services, consulte la [documentación de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services).
