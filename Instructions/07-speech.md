---
lab:
  title: Reconocimiento y síntesis de voz
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 88f5d500dc5adad83bdd476a278329792e1b2e45
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625884"
---
# <a name="recognize-and-synthesize-speech"></a>Reconocimiento y síntesis de voz

El servicio **Voz** es un servicio de Azure Cognitive Services que proporciona la funcionalidad relacionada con la voz, que incluye:

- Una API de *conversión de voz a texto* que permite implementar el reconocimiento de voz (conversión de texto oral audible en texto).
- Una API de *conversión de texto a voz* que permite implementar la síntesis de voz (conversión de texto en voz audible).

En este ejercicio, usará estas dos API para implementar una aplicación de reloj de voz.

**Nota**: Este ejercicio requiere que use un equipo con altavoces o auriculares. Para obtener la mejor experiencia, también se requiere un micrófono. Algunos entornos virtuales hospedados pueden capturar audio del micrófono local, pero si esto no funciona (o no tiene micrófono), puede usar un archivo de audio proporcionado para la entrada de voz. Siga las instrucciones detenidamente, ya que deberá elegir diferentes opciones en función de si usa un micrófono o el archivo de audio.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

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

## <a name="prepare-to-use-the-speech-service"></a>Preparación para usar el servicio Voz

En este ejercicio, completará una aplicación cliente implementada parcialmente que usa el SDK de Voz para reconocer y sintetizar voz.

**Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **07-speech** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **speaking-clock** y abra un terminal integrado. A continuación, instale el paquete del SDK de Voz mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

3. Consulte el contenido de la carpeta **speaking-clock** y observe que contiene un archivo para las opciones de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para incluir una **clave** de autenticación para el recurso de Cognitive Services, y la **ubicación** en la que se implementa. Guarde los cambios.
4. Tenga en cuenta que la carpeta **speaking-clock** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: speaking-clock.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico según el lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Voz:

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. En la función **Main**, fíjese en que ya se ha proporcionado código para cargar la clave de Cognitive Services y la región del archivo de configuración. Debe usar estas variables para crear **SpeechConfig** para el recurso de Cognitive Services. Agregue el código siguiente en el comentario **Configure speech service** (Configurar el servicio de voz):

    **C#**
    
    ```C#
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```
    
    **Python**
    
    ```Python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

6. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

7. Si usa C#, puede omitir las advertencias sobre el uso del operador **await** en métodos asincrónicos; lo corregiremos más adelante. El código debe mostrar la región del recurso del servicio de voz que usará la aplicación.

## <a name="recognize-speech"></a>Reconocer la voz

Ahora que tiene **SpeechConfig** para el servicio de voz en el recurso de Cognitive Services, puede usar la API de **conversión de voz en texto** para reconocer la voz y transcribirla a texto.

### <a name="if-you-have-a-working-microphone"></a>Si tiene un micrófono que funcione

1. En la función **Main** del programa, observe que el código usa la función **TranscribeCommand** para aceptar la entrada hablada.
2. En la función **TranscribeCommand**, en el comentario **Configurar Reconocimiento de voz**, agregue el código adecuado siguiente para crear un cliente **SpeechRecognizer** que se pueda usar para reconocer y transcribir la voz mediante el micrófono predeterminado del sistema:

    **C#**
    
    ```C#
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```
    
    **Python**
    
    ```Python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

3. Ahora vaya directamente a la sección **Adición de código para procesar el comando transcrito** a continuación.

### <a name="alternatively-use-audio-input-from-a-file"></a>Como alternativa, use la entrada de audio de un archivo

1. En la ventana de terminal, escriba el siguiente comando para instalar una biblioteca que pueda usar para reproducir el archivo de audio:

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

2. En el archivo de código del programa, en las importaciones del espacio de nombres existente, agregue el código siguiente para importar la biblioteca que acaba de instalar:

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

3. En la función **Main**, observe que el código usa la función **TranscribeCommand** para aceptar la entrada hablada. A continuación, en la función **TranscribeCommand**, en el comentario **Configurar Reconocimiento de voz**, agregue el código adecuado siguiente para crear un cliente **SpeechRecognizer** que se pueda usar para reconocer y transcribir la voz de un archivo de audio:

    **C#**

    ```C#
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech recognition
    audioFile = 'time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

### <a name="add-code-to-process-the-transcribed-command"></a>Adición de código para procesar el comando transcrito

1. En la función **TranscribeCommand**, en el comentario **Process speech input** (Procesar entrada de voz), agregue el código siguiente para escuchar la entrada hablada, y tenga cuidado de no reemplazar el código al final de la función que devuelve el comando:

    **C#**
    
    ```C#
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**
    
    ```Python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Si usa un micrófono, hable con claridad y diga "¿qué hora es?". El programa debe transcribir la entrada hablada y mostrar la hora (en función de la hora local del equipo donde se ejecuta el código, que puede que no sea la hora correcta donde se encuentra).

    SpeechRecognizer proporciona unos 5 segundos para hablar. Si no detecta ninguna entrada hablada, genera un resultado "Sin coincidencia".

    Si SpeechRecognizer encuentra un error, genera el resultado "Cancelado". A continuación, el código de la aplicación mostrará el mensaje de error. La causa más probable es una clave o región incorrectas en el archivo de configuración.

## <a name="synthesize-speech"></a>Sintetizar voz

La aplicación de reloj de voz acepta la entrada hablada, pero en realidad no habla. Vamos a corregirlo agregando código para sintetizar la voz.

1. En la función **Main** del programa, observe que el código usa la función **TellTime** para decir al usuario la hora actual.
2. En la función **TellTime**, en el comentario **Configure speech synthesis** (Configurar síntesis de voz), agregue el código siguiente para crear un cliente **SpeechSynthesizer** que se pueda usar para generar salida de voz:

    **C#**
    
    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```
    
    > **Nota**: *La configuración de audio predeterminada usa el dispositivo de audio del sistema predeterminado para la salida, por lo que no es necesario proporcionar explícitamente **audioConfig**. Si necesita redirigir la salida de audio a un archivo, puede usar **AudioConfig** con una ruta de acceso de archivo para hacerlo.*

3. En la función **TellTime**, en el comentario **Synthesize spoken output** (Sintetizar salida de voz), agregue el código siguiente para generar la salida hablada y tenga cuidado de no reemplazar el código al final de la función que imprime la respuesta:

    **C#**
    
    ```C#
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

4. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

5. Cuando se le solicite, hable con claridad hacia el micrófono y diga "¿qué hora es?". El programa debe hablar, indicando la hora.

## <a name="use-a-different-voice"></a>Uso de una voz distinta

La aplicación de reloj de voz usa una voz predeterminada, que se puede cambiar. El servicio Voz admite una variedad de voces *estándar*, así como voces *neuronales* más parecidas a las humanas. También puede crear voces *personalizadas*.

> **Nota**: Para obtener una lista de voces neuronales y estándar, consulte [Compatibilidad de idioma y voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#text-to-speech) en la documentación del servicio Voz.

1. En la función **TellTime**, en el comentario **Configure speech synthesis** (Configurar síntesis de voz), modifique el código tal como se indica a continuación para especificar una voz alternativa antes de crear un cliente **SpeechSynthesizer**:

   **C#**

    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural"; // add this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-RyanNeural' # add this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Cuando se le solicite, hable con claridad hacia el micrófono y diga "¿qué hora es?". El programa debe hablar en la voz especificada, indicando la hora.

## <a name="use-speech-synthesis-markup-language"></a>Uso de Lenguaje de marcado de síntesis de voz

El lenguaje de marcado de síntesis de voz (SSML) permite personalizar la forma en que se sintetiza la voz mediante un formato basado en XML.

1. En la función **TellTime**, reemplace todo el código actual en el comentario **Synthesize spoken output** (Sintentizar salida de voz) por el código siguiente (deje el código del comentario **Print the response** [Imprimir la respuesta]):

   **C#**

    ```C#
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**
    
    ```Python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Cuando se le solicite, hable con claridad hacia el micrófono y diga "¿qué hora es?". El programa debe hablar con la voz especificada en el SSML (reemplazando a la voz especificada en SpeechConfig), indicando la hora y, después de una pausa, le indicará que es el momento de finalizar este laboratorio, que lo es.

## <a name="more-information"></a>Más información

Para más información sobre el uso de las API de **conversión de voz en texto** y de **conversión de texto a voz**, consulte la [documentación de conversión de voz en texto](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-to-text) y la [documentación de conversión de texto a voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-text-to-speech).
