---
lab:
  title: Traducir voz
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 54bc0d9f942455e981437ecf5fe4a9de828c1cfb
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625953"
---
# <a name="translate-speech"></a>Traducir voz

El servicio de **Voz** incluye una API **Speech Translation** que puede usar para traducir la entrada hablada. Por ejemplo, supongamos que quiere desarrollar una aplicación de traductor que los usuarios puedan usar al viajar a lugares donde no hablan el idioma local. Podrían decir frases como "¿Dónde está la estación?" o "Necesito encontrar un restaurante" en su propio idioma y hacer que los traduzca al idioma local.

**Nota**: Este ejercicio requiere que use un equipo con altavoces o auriculares. Para obtener la mejor experiencia, también se requiere un micrófono. Algunos entornos virtuales hospedados pueden capturar audio del micrófono local, pero si esto no funciona (o no tiene micrófono), puede usar un archivo de audio proporcionado para la entrada de voz. Siga las instrucciones detenidamente, ya que deberá elegir diferentes opciones en función de si usa un micrófono o el archivo de audio.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si ya ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, ábralo en Visual Studio Code; en caso contrario, siga estos pasos para clonarlo ahora.

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
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará una de las claves y la ubicación en la que se aprovisiona el servicio desde esta página en el procedimiento siguiente.

## <a name="prepare-to-use-the-speech-translation-service"></a>Preparación para usar el servicio Speech Translation

En este ejercicio, completará una aplicación cliente implementada parcialmente que usa el SDK de Voz para reconocer, traducir y sintetizar voz.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **08-speech-translation** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **translator** y abra un terminal integrado. A continuación, instale el paquete del SDK de Voz mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

3. Consulte el contenido de la carpeta **translator** y fíjese en que contiene un archivo para las opciones de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para incluir una **clave** de autenticación para el recurso de Cognitive Services, y la **ubicación** en la que se implementa. Guarde los cambios.
4. Tenga en cuenta que la carpeta **translator** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: translator.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico según el lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Voz:

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. En la función **Main**, fíjese en que ya se ha proporcionado código para cargar la clave de Cognitive Services y la región del archivo de configuración. Debe usar estas variables para crear **SpeechTranslationConfig** para el recurso de Cognitive Services, que usará para traducir la entrada hablada. Agregue el código siguiente en el comentario **Configurar traducción**:

    **C#**
    
    ```C#
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```
    
    **Python**
    
    ```Python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(cog_key, cog_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

6. Usará **SpeechTranslationConfig** para traducir voz a texto, pero también usará **SpeechConfig** para sintetizar las traducciones en voz. Agregue el código siguiente en el comentario **Configurar voz**:

    **C#**
    
    ```C#
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    ```
    
    **Python**
    
    ```Python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    ```

7. Guarde los cambios y vuelva al terminal integrado de la carpeta **translator** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

8. Si usa C#, puede omitir las advertencias sobre el uso del operador **await** en métodos asincrónicos; lo corregiremos más adelante. El código debe mostrar un mensaje que indica que está listo para traducir desde en-US. Presione ENTRAR para finalizar el programa.

## <a name="implement-speech-translation"></a>Implementación de la traducción de voz

Ahora que tiene **SpeechTranslationConfig** para el servicio de voz en el recurso de Cognitive Services, puede usar la API **Speech Translation** para reconocer y traducir voz.

### <a name="if-you-have-a-working-microphone"></a>Si tiene un micrófono que funcione

1. En la función **Main** del programa, tenga en cuenta que el código usa la función **Translate** para traducir la entrada hablada.
2. En la función **Translate**, en el comentario **Traducir voz**, agregue el código siguiente para crear un cliente **TranslationRecognizer** que se pueda usar para reconocer y traducir la voz mediante el micrófono predeterminado del sistema para la entrada.

    **C#**
    
    ```C#
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **Nota**: El código de la aplicación traduce la entrada a los tres idiomas en una sola llamada. Solo se muestra la traducción del idioma específico, pero puede recuperar cualquiera de las traducciones especificando el código de idioma de destino en la colección **traducciones** del resultado.

3. Ahora vaya directamente a la sección **Ejecución del programa** que aparece a continuación.

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

3. En la función **Main** del programa, tenga en cuenta que el código usa la función **Translate** para traducir la entrada hablada. A continuación, en la función **Translate**, en el comentario **Traducir voz**, agregue el código siguiente para crear un cliente **TranslationRecognizer** que se pueda usar para reconocer y traducir la voz de un archivo.

    **C#**
    
    ```C#
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **Nota**: El código de la aplicación traduce la entrada a los tres idiomas en una sola llamada. Solo se muestra la traducción del idioma específico, pero puede recuperar cualquiera de las traducciones especificando el código de idioma de destino en la colección **traducciones** del resultado.

### <a name="run-the-program"></a>Ejecución del programa

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **translator** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

2. Cuando se le solicite, escriba un código de idioma válido(*fr*, *es* o *hi*) y, si usa un micrófono, hable claramente y diga "¿dónde está la estación?" o alguna otra frase que pueda usar al viajar. El programa debe transcribir la entrada hablada y traducirla al idioma especificado (francés, español o hindi). Repita este proceso e intente cada idioma admitido por la aplicación. Cuando haya terminado, presione ENTRAR para finalizar el programa.

    > **Nota**: TranslationRecognizer le da unos 5 segundos para hablar. Si no detecta ninguna entrada hablada, genera un resultado "Sin coincidencia".
    >
    > Es posible que la traducción a Hindi no siempre se muestre correctamente en la ventana de la consola debido a problemas de codificación de caracteres.

## <a name="synthesize-the-translation-to-speech"></a>Síntesis de la traducción a voz

Hasta ahora, la aplicación traduce la entrada hablada en texto, lo que puede ser suficiente si necesita pedir ayuda a alguien mientras viaja. Sin embargo, sería mejor que la traducción se pronunciara en voz alta en una voz adecuada.

1. En la función **Translate**, en el comentario **Síntesis de la traducción**, agregue el código siguiente para usar un cliente **SpeechSynthesizer** para sintetizar la traducción como voz a través del altavoz predeterminado:

    **C#**
    
    ```C#
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. Guarde los cambios y vuelva al terminal integrado de la carpeta **translator** y escriba el siguiente comando para ejecutar el programa:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

3. Cuando se le solicite, escriba un código de idioma válido *(fr*, *es* o *hi*) y, a continuación, hable claramente en el micrófono y diga una frase que podría usar al viajar. El programa debería transcribir la entrada hablada y responder con una traducción hablada. Repita este proceso e intente cada idioma admitido por la aplicación. Cuando haya terminado, presione ENTRAR para finalizar el programa.

    > **Nota** *En este ejemplo, ha usado **SpeechTranslationConfig** para traducir voz a texto y, a continuación, ha usado **SpeechConfig** para sintetizar la traducción como voz. De hecho, puede usar **SpeechTranslationConfig** para sintetizar la traducción directamente, pero esto solo funciona cuando se traduce a un solo idioma y da como resultado una secuencia de audio que normalmente se guarda como un archivo en lugar de enviarse directamente a un altavoz.*

## <a name="more-information"></a>Más información

Para obtener más información sobre el uso de la API **Speech Translation**, consulte la [documentación de traducción de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-translation).
