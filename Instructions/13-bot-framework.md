---
lab:
  title: Creación de un bot con el SDK de Bot Framework
  module: Module 7 - Conversational AI and the Azure Bot Service
ms.openlocfilehash: ab51a1f11f4eee99838634d83b5d9261a8c20ecb
ms.sourcegitcommit: 480c5898009ea964025fdecb57900aefeeac81fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/07/2022
ms.locfileid: "147019888"
---
# <a name="create-a-bot-with-the-bot-framework-sdk"></a>Creación de un bot con el SDK de Bot Framework

Los *bots* son agentes de software que pueden participar en diálogos de conversación con usuarios humanos. El Microsoft Bot Framework proporciona una plataforma completa para compilar bots que se pueden entregar como servicios en la nube a través de Azure Bot Service.

En este ejercicio, usará el SDK de Microsoft Bot Framework para crear e implementar un bot.

## <a name="before-you-start"></a>Antes de comenzar

Para empezar, preparemos el entorno para el desarrollo de bots.

### <a name="update-the-bot-framework-emulator"></a>Actualización de Bot Framework Emulator

Va a usar el SDK de Bot Framework para crear el bot y Bot Framework Emulator para probarlo. Bot Framework Emulator se actualiza periódicamente, así que vamos a asegurarnos de que tiene instalada la versión más reciente.

> **Nota**: Las actualizaciones pueden incluir cambios en la interfaz de usuario que afectan a las instrucciones de este ejercicio.

1. Inicie **Bot Framework Emulator** y, si se le pide que instale una actualización, hágalo para el usuario que ha iniciado sesión actualmente. Si no se le solicita automáticamente, use la opción **Buscar actualización** en el menú **Ayuda** para buscar actualizaciones.
2. Después de instalar cualquier actualización disponible, cierre Bot Framework Emulator hasta que lo vuelva a necesitar más adelante.

> **Importante**: A veces la actualización no se descarga; estamos investigándolo. Si no hay progreso en un par de minutos después de intentar actualizar, puede descartar la descarga y usar la versión instalada actualmente del emulador.

### <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="create-a-bot"></a>Creación de un bot

Puede usar el SDK de Bot Framework para crear un bot basado en una plantilla y, a continuación, personalizar el código para satisfacer sus requisitos específicos.

> **Nota**: En este ejercicio, puede elegir usar **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **13-bot-framework** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta del idioma elegido y abra un terminal integrado.
3. En el terminal, ejecute los siguientes comandos para instalar las plantillas de bot y los paquetes que necesita:

**C#**

```C#
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```Python
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. Una vez instaladas las plantillas y los paquetes, ejecute el siguiente comando para crear un bot basado en la plantilla *EchoBot*:

**C#**

```C#
dotnet new echobot -n TimeBot
```

**Python**

```Python
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Si usa Python, cuando se lo solicite, escriba los detalles siguientes:
- **bot_name**: TimeBot
- **bot_description**: A bot for our times
    
5. En el panel de terminal, escriba los siguientes comandos para cambiar el directorio actual a la carpeta **TimeBot** en la lista de archivos de código que se han generado para el bot:

    ```Code
    cd TimeBot
    dir
    ```

## <a name="test-the-bot-in-the-bot-framework-emulator"></a>Probar el bot en Bot Framework Emulator

Ha creado un bot basado en la plantilla *EchoBot*. Ahora puede ejecutarlo localmente y probarlo mediante Bot Framework Emulator (que debería haberse instalado en el sistema).

1. En el panel del terminal, asegúrese de que el directorio actual es la carpeta **TimeBot** que contiene los archivos de código del bot y, a continuación, escriba el siguiente comando para iniciar el bot localmente.

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```
    
Cuando se inicie el bot, tenga en cuenta que se muestra el punto de conexión en el que se está ejecutando. Debería ser similar a **http://localhost:3978** .

2. Inicie Bot Framework Emulator y abra el bot especificando el punto de conexión con la ruta de acceso **/api/messages** anexada, de la siguiente manera:

    `http://localhost:3978/api/messages`

3. Una vez abierta la conversación en un panel de **Chat en directo**, espere el mensaje *Hello and welcome!* .
4. Escriba un mensaje, como, p. ej., *Hello*, y vea la respuesta del bot, que debería devolver el mensaje que escribió.
5. Cierre Bot Framework Emulator y vuelva a Visual Studio Code y, a continuación, en la ventana del terminal, escriba **Ctrl + C** para detener el bot.

## <a name="modify-the-bot-code"></a>Modificar el código del bot

Ha creado un bot que devuelve la entrada de usuario. No es especialmente útil, pero sirve para ilustrar el flujo básico de un diálogo conversacional. Una conversación con un bot consta de una secuencia de *actividades* en las que se usa texto, gráficos o *tarjetas* de interfaz de usuario para intercambiar información. El bot comienza la conversación con un saludo, que es el resultado de una actividad de *actualización de la conversación* que se desencadena cuando un usuario inicializa una sesión de chat con el bot. A continuación, la conversación consta de una secuencia de otras actividades en las que el usuario y el bot intercambian *mensajes*.

1. En Visual Studio Code, abra el siguiente archivo de código para el bot:
    - **C#** : TimeBot/Bots/EchoBot.cs
    - **Python**: TimeBot/bot.py

    Tenga en cuenta que el código de este archivo consta de funciones de *controlador de actividad*; una para la actividad de actualización de conversación *Miembro agregado* (cuando alguien se une a la sesión de chat), y otra para la actividad *Mensaje* (cuando se recibe un mensaje). La conversación se basa en el concepto de *turnos*, en el que cada turno representa una interacción en la que el bot recibe, procesa y responde a una actividad. El *contexto del turno* se usa para realizar un seguimiento de la información sobre la actividad que se está procesando en el turno actual.

2. En la parte superior del archivo de código, agregue la siguiente instrucción de importación del espacio de nombres:

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. Modifique la función de controlador de actividad para que la actividad *Mensaje* coincida con el código siguiente:

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. Guarde los cambios y, a continuación, en el panel del terminal, asegúrese de que el directorio actual es la carpeta **TimeBot** que contiene los archivos de código del bot y, a continuación, escriba el siguiente comando para iniciar el bot localmente.

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```

Igual que antes, cuando se inicie el bot, tenga en cuenta que se muestra el punto de conexión en el que se está ejecutando.

5. Inicie Bot Framework Emulator y abra el bot especificando el punto de conexión con la ruta de acceso **/api/messages** anexada, de la siguiente manera:

    `http://localhost:3978/api/messages`

6. Una vez abierta la conversación en un panel de **Chat en directo**, espere el mensaje *Hello and welcome!* .
7. Escriba un mensaje, como, p. ej., *Hello*, y vea la respuesta del bot, que debería ser *Ask me what the time is*.
8. Escriba *What is the time?* y vea la respuesta.

    El bot ahora responde a la consulta "What is the time?" mostrando la hora local en la que se ejecuta el bot. Para cualquier otra consulta, solicita al usuario que le pregunte qué hora es. Se trata de un bot muy limitado, que se podría mejorar mediante la integración con el servicio Language Understanding y código personalizado adicional, pero sirve como ejemplo práctico de cómo puede crear una solución con el SDK de Bot Framework mediante la extensión de un bot creado a partir de una plantilla.

9. Cierre Bot Framework Emulator y vuelva a Visual Studio Code y, a continuación, en la ventana del terminal, escriba **Ctrl + C** para detener el bot.

## <a name="more-information"></a>Más información

Para obtener más información sobre Bot Framework, consulte la [documentación de Bot Framework](https://docs.microsoft.com/azure/bot-service/index-bf-sdk).
