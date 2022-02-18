---
lab:
  title: Administración de la seguridad de Azure Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: dcab47cf20f54d6bcbed9a3e40081b703fc2d5ba
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801336"
---
# <a name="manage-cognitive-services-security"></a>Administración de la seguridad de Azure Cognitive Services

La seguridad es una consideración crítica para cualquier aplicación y, como desarrollador, debe asegurarse de que el acceso a recursos como Cognitive Services está restringido solo a aquellos usuarios que lo requieren.

El acceso a Cognitive Services normalmente se controla mediante claves de autenticación, que se generan al crear inicialmente un recurso de Cognitive Services.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

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
4. Espere a que se complete la implementación y, a continuación, vea los detalles de la implementación.

## <a name="manage-authentication-keys"></a>Administrar las claves de autenticación

Al crear el recurso de Cognitive Services, se generaron dos claves de autenticación. Puede administrarlas en Azure Portal o mediante la interfaz de la línea de comandos (CLI) de Azure.

1. En Azure Portal, vaya al recurso de Cognitive Services y consulte su página **Keys and Endpoint** (Claves y punto de conexión). Esta página contiene la información que necesitará para conectarse al recurso y usarlo desde las aplicaciones que desarrolle. Concretamente:
    - Un *punto de conexión* HTTP al que las aplicaciones cliente puedan enviar solicitudes.
    - Dos *claves* que se puedan usar para la autenticación (las aplicaciones cliente pueden usar cualquiera de las claves. Una práctica común es usar una para el desarrollo y la otra para producción. Puede volver a generar fácilmente la clave de desarrollo después de que los desarrolladores terminen su trabajo para evitar el acceso continuado).
    - La *ubicación* en la que se hospeda el recurso. Esta información es necesaria para las solicitudes a algunas API (pero no a todas).
2. En Visual Studio Code, haga clic con el botón derecho en la carpeta **02-cognitive-security** y abra un terminal integrado. A continuación, escriba el siguiente comando para iniciar sesión en su suscripción de Azure mediante la CLI de Azure.

    ```
    az login
    ```

    Se abrirá una pestaña del explorador web que le solicitará que inicie sesión en Azure. Hágalo y, a continuación, cierre la pestaña del explorador y vuelva a Visual Studio Code.

    > **Sugerencia**: Si tiene varias suscripciones, deberá asegurarse de que está trabajando en la que contiene el recurso de Cognitive Services.  Use este comando para determinar la suscripción actual: su identificador único es el valor de **id** del archivo JSON que se devuelve.

    > **Advertencia**: Si se produce un error de verificación de certificado para `az login`, intente esperar unos minutos e inténtelo de nuevo.
    >
    > ```
    > az account show
    > ```
    >
    > Si necesita cambiar la suscripción, ejecute este comando y cambie *&lt;Your_Subscription_Id&gt;* por el id. de suscripción correcto.
    >
    > ```
    > az account set --subscription <Your_Subscription_Id>
    > ```
    >
    > Como alternativa, puede especificar explícitamente el id. de suscripción como un parámetro *--subscription* en cada comando de la CLI de Azure siguiente.

3. Ahora puede usar el comando siguiente para obtener la lista de claves de Cognitive Services y reemplazar *&lt;resourceName&gt;* por el nombre del recurso de Cognitive Services y *&lt; resourceGroup&gt;* por el nombre del grupo de recursos en el que lo creó.

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

El comando devuelve una lista de las claves del recurso de Cognitive Services: hay dos claves, denominadas **key1** y **key2**.

4. Para probar el servicio de Cognitive Services, puede usar **curl**: una herramienta de línea de comandos para las solicitudes HTTP. En la carpeta **02-cognitive-security**, abra **rest-test.cmd** y edite el comando **curl** que contiene (que se muestra a continuación); reemplace *&lt;yourEndpoint&gt;* y *&lt;yourKey&gt;* por el URI del punto de conexión y la clave **Key1** para usar la API de Text Analytics en el recurso de Cognitive Services.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.0/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':[{'id':1,'text':'hello'}]}"
    ```

5. Guarde los cambios y, a continuación, en el terminal integrado de la carpeta **02-cognitive-security**, ejecute el siguiente comando:

    ```
    rest-test
    ```

El comando devuelve un documento JSON que contiene información sobre el idioma detectado en los datos de entrada (que debe ser inglés).

6. Si una clave se ve comprometida o los desarrolladores que la tienen ya no requieren acceso, puede volver a generarla en el portal o mediante la CLI de Azure. Ejecute el siguiente comando para volver a generar la clave **key1** (reemplace los valores de *&lt;resourceName&gt;* y *&lt;resourceGroup&gt;* de su recurso).

    ```
    az cognitiveservices account keys regenerate --name <resourceName> --resource-group <resourceGroup> --key-name key1
    ```

Se devuelve la lista de claves del recurso de Cognitive Services: observe que **key1** ha cambiado desde la última vez que recuperó las claves.

7. Vuelva a ejecutar el comando **rest-test** con la clave antigua (puede usar la clave **^** para recorrer los comandos anteriores) y verifique que ahora se produce un error.
8. Edite el comando *curl* en **rest-test.cmd**; reemplace la clave por el nuevo valor de **key1** y guarde los cambios. A continuación, vuelva a ejecutar el comando **rest-test** y compruebe que se ejecuta correctamente.

> **Sugerencia**: En este ejercicio, ha usado los nombres completos de los parámetros de la CLI de Azure, como **--resource-group**.  También puede usar alternativas más cortas, como **-g**, para que los comandos sean menos detallados (pero un poco más difíciles de entender).  En la [referencia de comandos de la CLI de Cognitive Services](https://docs.microsoft.com/cli/azure/cognitiveservices?view=azure-cli-latest) se enumeran las opciones de parámetros para cada comando de la CLI de Cognitive Services.

## <a name="secure-key-access-with-azure-key-vault"></a>Proteger el acceso a las claves con Azure Key Vault

Puede desarrollar aplicaciones que consuman Cognitive Services mediante una clave para la autenticación. Sin embargo, esto significa que el código de la aplicación debe poder obtener la clave. Una opción es almacenar la clave en una variable de entorno o en un archivo de configuración donde se implemente la aplicación, pero este enfoque hace que la clave sea vulnerable al acceso no autorizado. Un enfoque mejor al desarrollar aplicaciones en Azure es almacenar la clave de forma segura en Azure Key Vault y proporcionar acceso a la clave a través de una *identidad administrada* (es decir, una cuenta de usuario usada por la propia aplicación).

### <a name="create-a-key-vault-and-add-a-secret"></a>Creación de un almacén de claves y adición de un secreto

En primer lugar, debe crear un almacén de claves y agregar un *secreto* para la clave de Cognitive Services.

1. Anote el valor de **key1** del recurso de Cognitive Services (o cópielo en el Portapapeles).
2. En la Azure Portal, en la página **Inicio**, seleccione el botón **&#65291;Crear un recurso**, busque *Key Vault* y cree un recurso de **Key Vault** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *el mismo grupo de recursos que el recurso de Cognitive Services*
    - **Nombre del almacén de claves**: *escriba un nombre único*.
    - **Región**: *la misma región que el recurso de Cognitive Service*
    - **Plan de tarifa**: estándar
3. Espere a que se complete la implementación y, a continuación, vaya al recurso del almacén de claves.
4. En el panel de navegación izquierdo, seleccione **Secretos** (en la sección Configuración).
5. Seleccione **+ Generate/Import** (+ Generar/Importar) y agregue un nuevo secreto con la siguiente configuración:
    - **Opciones de carga**: manual
    - **Nombre**: Cognitive-Services-Key *(es importante que coincida exactamente, porque más adelante ejecutará código que recuperará el secreto basado en este nombre)* .
    - **Valor**: *la clave de Cognitive Services **key1***

### <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Para acceder al secreto del almacén de claves, la aplicación debe usar una entidad de servicio que tenga acceso al secreto. Usará la interfaz de la línea de comandos (CLI) de Azure para crear la entidad de servicio, encontrar su id. de objeto y conceder acceso al secreto en Azure Key Vault.

1. Vuelva a Visual Studio Code y, en el terminal integrado de la carpeta **02-cognitive-security**, ejecute el siguiente comando de la CLI de Azure; reemplace *&lt;spName &gt;* por un nombre adecuado para una identidad de aplicación (por ejemplo, *ai-app*). Reemplace también *&lt;subscriptionId&gt;* y *&lt;resourceGroup&gt;* por los valores correctos para el id. de suscripción y el grupo de recursos que contienen los recursos de Cognitive Services y del almacén de claves:

    > **Sugerencia**: Si no está seguro del id. de suscripción, use el comando **az account show** para recuperar la información de la suscripción (el id. de suscripción es el atributo **id** de la salida).

    ```
    az ad sp create-for-rbac -n "api://<spName>" --role owner --scopes subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    ```

La salida de este comando incluye información sobre la nueva entidad de servicio. Debe tener un aspecto similar a este:

    ```
    {
        "appId": "abcd12345efghi67890jklmn",
        "displayName": "ai-app",
        "name": "http://ai-app",
        "password": "1a2b3c4d5e6f7g8h9i0j",
        "tenant": "1234abcd5678fghi90jklm"
    }
    ```

Tome nota de los valores de **appId**,**password** y **tenant**: los necesitará más adelante (si cierra este terminal, no podrá recuperar la contraseña, por lo que es importante anotar los valores ahora; puede pegar la salida en un nuevo archivo de texto en Visual Studio Code para asegurarse de poder encontrar los valores que necesita más adelante).

2. Para obtener el **id. de objeto** de la entidad de servicio, ejecute el siguiente comando de la CLI de Azure; reemplace *&lt;appId&gt;* por el valor del id. de aplicación de la entidad de servicio.

    ```
    az ad sp show --id <appId> --query objectId --out tsv
    ```

3. Para asignar permiso a fin de que la nueva entidad de servicio acceda a los secretos en Key Vault, ejecute el siguiente comando de la CLI de Azure; reemplace *&lt;keyVaultName&gt;* por el nombre del recurso de Azure Key Vault y *&lt;objectId&gt;* por el valor del id. de objeto de la entidad de servicio.

    ```
    az keyvault set-policy -n <keyVaultName> --object-id <objectId> --secret-permissions get list
    ```

### <a name="use-the-service-principal-in-an-application"></a>Uso de la entidad de servicio en una aplicación

Ahora ya puede usar la identidad de la entidad de servicio en una aplicación, de modo que esta pueda acceder a la clave de Cognitive Services en el almacén de claves y usarla para conectarse al recurso de Cognitive Services.

> **Nota**: En este ejercicio, almacenaremos las credenciales de la entidad de servicio en la configuración de la aplicación y las usaremos para autenticar una identidad **ClientSecretCredential** en el código de la aplicación. Esto está bien para desarrollo y pruebas, pero, en una aplicación de producción real, un administrador asignaría una *identidad administrada* a la aplicación para que use la identidad de la entidad de servicio para acceder a los recursos, sin almacenar en caché ni guardar la contraseña.

1. En Visual Studio Code, expanda la carpeta **02-cognitive-security** y la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **keyvault-client** y abra un terminal integrado. A continuación, instale los paquetes que necesitará para usar Azure Key Vault y Text Analytics API en el recurso de Cognitive Services mediante la ejecución del comando adecuado para su preferencia de lenguaje:

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    dotnet add package Azure.Identity --version 1.5.0
    dotnet add package Azure.Security.KeyVault.Secrets --version 4.2.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    pip install azure-identity==1.5.0
    pip install azure-keyvault-secrets==4.2.0
    ```

3. Consulte el contenido de la carpeta **keyvault-client** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar los valores siguientes:
    
    - El **punto de conexión** del recurso de Cognitive Services
    - El nombre del recurso de **Azure Key Vault**
    - El **inquilino** de la entidad de servicio
    - El **appId** de la entidad de servicio
    - La **contraseña** de la entidad de servicio

     Guarde los cambios.
4. Observe que la carpeta **keyvault-client** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: keyvault-client.py

    Abra el archivo de código y revise el código que contiene, fijándose en los siguientes detalles:
    - Se importa el espacio de nombres del SDK que instaló.
    - El código de la función **Main** recupera los valores de configuración de la aplicación y, a continuación, usa las credenciales de la entidad de servicio para obtener la clave de Cognitive Services del almacén de claves.
    - La función **GetLanguage** usa el SDK para crear un cliente para el servicio y, a continuación, usa el cliente para detectar el idioma del texto introducido.
5. Vuelva al terminal integrado de la carpeta **keyvault-client** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python keyvault-client.py
    ```

6. Cuando se le solicite, escriba algún texto y revise el idioma que detecta el servicio. Por ejemplo, pruebe a escribir "Hello", "Bonjour" y "Hola".
7. Cuando haya terminado de probar la aplicación, escriba "quit" (salir) para detener el programa.

## <a name="more-information"></a>Más información

Para obtener más información sobre la protección de Cognitive Services, consulte la [documentación sobre seguridad de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security).
