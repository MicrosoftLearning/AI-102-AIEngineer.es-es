---
lab:
  title: Lectura de texto en imágenes
  module: Module 11 - Reading Text in Images and Documents
ms.openlocfilehash: 1199e4e4f44a98fc5f900fa1ad021384b56f0c2b
ms.sourcegitcommit: e242452e8125a2622093980048f1e2cacb8b893d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2022
ms.locfileid: "145757494"
---
# <a name="read-text-in-images"></a>Lectura de texto en imágenes

El reconocimiento óptico de caracteres (OCR) es un subconjunto de Computer Vision que se ocupa de leer texto en imágenes y documentos. El servicio **Computer Vision** proporciona dos API para leer texto, que explorará en este ejercicio.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no lo ha hecho, debe clonar el repositorio de código para este curso:

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
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Keys and Endpoint** (Claves y punto de conexión). Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Preparación para el uso del SDK de Computer Vision

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Computer Vision para leer texto.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **20-ocr** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **read-text** y abra un terminal integrado. A continuación, instale el paquete del SDK de Computer Vision mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```

3. Consulte el contenido de la carpeta **read-text** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Cognitive Services. Guarde los cambios.
4. Tenga en cuenta que la carpeta **read-text** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: read-text.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces** (Importar espacios de nombres). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Computer Vision:

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. En el archivo de código de la aplicación cliente, en la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Computer Vision client** (Autenticar cliente de Computer Vision). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Computer Vision:

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```
    
## <a name="use-the-ocr-api"></a>Uso de la API de OCR

La API de **OCR** es una API de reconocimiento óptico de caracteres optimizada para leer cantidades pequeñas y medianas de texto impreso de imágenes en formato *.jpg*, *.png*, *.gif* y *.bmp*. Admite una amplia gama de idiomas y, además de leer texto en la imagen, puede determinar la orientación de cada región de texto y devolver información sobre el ángulo de rotación del texto en relación con la imagen.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **1**. Este código llama a la función **GetTextOcr** y pasa la ruta de acceso a un archivo de imagen.
2. En la carpeta **read-text/images**, abra **Lincoln.jpg** para ver la imagen que procesará el código.
3. De nuevo en el archivo de código, busque la función **GetTextOcr** y, en el código existente que imprime un mensaje en la consola, agregue el código siguiente:

**C#**

```C#
// Use OCR API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var ocrResults = await cvClient.RecognizePrintedTextInStreamAsync(detectOrientation:false, image:imageData);

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Magenta, 3);

    foreach(var region in ocrResults.Regions)
    {
        foreach(var line in region.Lines)
        {
            // Show the position of the line of text
            int[] dims = line.BoundingBox.Split(",").Select(int.Parse).ToArray();
            Rectangle rect = new Rectangle(dims[0], dims[1], dims[2], dims[3]);
            graphics.DrawRectangle(pen, rect);

            // Read the words in the line of text
            string lineText = "";
            foreach(var word in line.Words)
            {
                lineText += word.Text + " ";
            }
            Console.WriteLine(lineText.Trim());
        }
    }

    // Save the image with the text locations highlighted
    String output_file = "ocr_results.jpg";
    image.Save(output_file);
    Console.WriteLine("Results saved in " + output_file);
}
```

**Python**

```Python
# Use OCR API to read text in image
with open(image_file, mode="rb") as image_data:
    ocr_results = cv_client.recognize_printed_text_in_stream(image_data)

# Prepare image for drawing
fig = plt.figure(figsize=(7, 7))
img = Image.open(image_file)
draw = ImageDraw.Draw(img)

# Process the text line by line
for region in ocr_results.regions:
    for line in region.lines:

        # Show the position of the line of text
        l,t,w,h = list(map(int, line.bounding_box.split(',')))
        draw.rectangle(((l,t), (l+w, t+h)), outline='magenta', width=5)

        # Read the words in the line of text
        line_text = ''
        for word in line.words:
            line_text += word.text + ' '
        print(line_text.rstrip())

# Save the image with the text locations highlighted
plt.axis('off')
plt.imshow(img)
outputfile = 'ocr_results.jpg'
fig.savefig(outputfile)
print('Results saved in', outputfile)
```

4. Examine el código que agregó a la función **GetTextOcr**. Detecta las regiones de texto impreso de un archivo de imagen y, para cada región, extrae las líneas de texto y resalta la posición en la imagen. A continuación, extrae las palabras de cada línea y las imprime.
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **read-text** y escriba el siguiente comando para ejecutar el programa:

**C#**

```
dotnet run
```

*La salida de C# puede mostrar advertencias sobre las funciones asincrónicas que ahora usan el operador **await**. No obstante, puede ignorarlas.*

**Python**

```
python read-text.py
```

6. Cuando se le solicite, escriba **1** y observe la salida, que es el texto extraído de la imagen.
7. Consulte el archivo **ocr_results.jpg** que se genera en la misma carpeta que el archivo de código para ver las líneas de texto anotadas de la imagen.

## <a name="use-the-read-api"></a>Uso de Read API

**Read** API usa un modelo de reconocimiento de texto más reciente que la API de OCR y funciona mejor para imágenes más grandes que contienen una gran cantidad de texto. También admite la extracción de texto de archivos *.pdf* y puede reconocer tanto texto impreso como texto manuscrito en varios idiomas.

**Read** API usa un modelo de operaciones asincrónicas, en el que se envía una solicitud para iniciar el reconocimiento de texto y el id. de operación devuelto por la solicitud se pueden usar posteriormente para comprobar el progreso y recuperar los resultados.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **2**. Este código llama a la función **GetTextRead** y pasa la ruta de acceso a un archivo de documento PDF.
2. En la carpeta **read-text/images**, haga clic con el botón derecho en **Rome.pdf** y seleccione **Mostrar en el Explorador de archivos**. A continuación, en el Explorador de archivos, abra el archivo PDF para verlo.
3. De nuevo en el archivo de código en Visual Studio Code, busque la función **GetTextRead** y, en el código existente que imprime un mensaje en la consola, agregue el código siguiente:

**C#**

```C#
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfuly, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
            }
        }
    }
}  
```

**Python**

```Python
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfuly, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
```
    
4. Examine el código que agregó a la función **GetTextRead**. Envía una solicitud para una operación de lectura y, a continuación, comprueba repetidamente el estado hasta que la operación se completa. Si se ha realizado correctamente, el código procesa los resultados mediante la iteración por cada página y, a continuación, por cada línea.
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **read-text** y escriba el siguiente comando para ejecutar el programa:

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. Cuando se le solicite, escriba **2** y observe la salida, que es el texto extraído del documento.

## <a name="read-handwritten-text"></a>Leer texto manuscrito

Además del texto impreso, **Read** API puede extraer texto manuscrito en inglés.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **3**. Este código llama a la función **GetTextRead** y pasa la ruta de acceso a un archivo de imagen.
2. En la carpeta **read-text/images**, abra **Note.jpg** para ver la imagen que procesará el código.
3. En el terminal integrado de la carpeta **read-text**, escriba el siguiente comando para ejecutar el programa:

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. Cuando se le solicite, escriba **3** y observe la salida, que es el texto extraído del documento.

## <a name="more-information"></a>Más información

Para obtener más información sobre cómo usar el servicio **Computer Vision** para la lectura de texto, consulte la [documentación de Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text).
