---
lab:
  title: Detección, análisis y reconocimiento de caras
  module: Module 10 - Detecting, Analyzing, and Recognizing Faces
ms.openlocfilehash: b9565f41eb67b916278508c729860a3471a9e0bd
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625918"
---
# <a name="detect-analyze-and-recognize-faces"></a>Detección, análisis y reconocimiento de caras

La capacidad de detectar, analizar y reconocer caras de personas es una funcionalidad básica de la inteligencia artificial. En este ejercicio, explorará dos funcionalidades de Azure Cognitive Services que puede usar para trabajar con caras de imágenes: el servicio **Computer Vision** y el servicio **Face**.

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no lo ha hecho, debe clonar el repositorio de código para este curso:

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús+Ctrl+P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/AI-102-AIEngineer` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## <a name="provision-a-cognitive-services-resource"></a>Aprovisionamiento de un recurso de Cognitive Services

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso de **Cognitive Services**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Seleccione el botón **&amp;#65291;Crear un recurso**, busque *Cognitive Services* y cree un recurso de **Cognitive Services** con la siguiente configuración:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos (si usa una suscripción restringida, es posible que no tenga permiso para crear un nuevo grupo de recursos; use el proporcionado)*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: estándar S0
3. Active las casillas necesarias y cree el recurso.
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Claves y punto de conexión**. Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Preparación para el uso del SDK de Computer Vision

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Computer Vision para analizar las caras de una imagen.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **19-face** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **computer-vision** y abra un terminal integrado. A continuación, instale el paquete del SDK de Computer Vision mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. Consulte el contenido de la carpeta **computer-vision** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

4. Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Cognitive Services. Guarde los cambios.

5. Tenga en cuenta que la carpeta **computer-vision** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: detect-faces.py

6. Abra el archivo de código y, en la parte superior, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces** (Importar espacios de nombres). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Computer Vision:

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
    from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
    from msrest.authentication import CognitiveServicesCredentials
    ```

## <a name="view-the-image-you-will-analyze"></a>Visualización de la imagen que se va a analizar

En este ejercicio, usará el servicio Computer Vision para analizar una imagen de personas.

1. En Visual Studio Code, expanda la carpeta **computer-vision** y la carpeta **images** que contiene.
2. Seleccione la imagen **people.jpg** para verla.

## <a name="detect-faces-in-an-image"></a>Detectar caras en una imagen

Ahora ya puede usar el SDK para llamar al servicio Computer Vision y detectar caras en una imagen.

1. En el archivo de código de la aplicación cliente (**Program.cs** o **detect-faces.py**), en la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Computer Vision client** (Autenticar cliente de Computer Vision). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Computer Vision:

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

2. En la función **Main**, en el código que acaba de agregar, observe que el código especifica la ruta de acceso a un archivo de imagen y, a continuación, pasa la ruta de acceso de la imagen a una función denominada **AnalyzeFaces**. Esta función aún no está totalmente implementada.

3. En la función **AnalyzeFaces**, debajo del comentario **Specify features to be retrieved (faces)** (Especificar características que se recuperarán [caras]), agregue el código siguiente:

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. En la función **AnalyzeFaces**, debajo del comentario **Get image analysis** (Obtener análisis de imagen), agregue el código siguiente:

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person aged approximately {face.Age}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person aged approximately {}'.format(face.age)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. Guarde los cambios y vuelva al terminal integrado de la carpeta **computer-vision** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. Observe la salida, que debe indicar el número de caras detectadas.
7. Consulte el archivo **detected_faces.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas. En este caso, el código ha usado los atributos de la cara para estimar la edad de cada persona de la imagen y las coordenadas del rectángulo delimitador para dibujar un rectángulo alrededor de cada cara.

## <a name="prepare-to-use-the-face-sdk"></a>Preparación para el uso del SDK de Face

Aunque el servicio **Computer Vision** ofrece detección de caras básica (junto con muchas otras funcionalidades de análisis de imágenes), el servicio **Face** proporciona una funcionalidad más completa de reconocimiento y análisis facial.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **19-face** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **face-api** y abra un terminal integrado. A continuación, instale el paquete del SDK de Face mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. Consulte el contenido de la carpeta **face-api** y fíjese en que contiene un archivo para los valores de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

4. Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Cognitive Services. Guarde los cambios.

5. Tenga en cuenta que la carpeta **face-api** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: analyze-faces.py

6. Abra el archivo de código y, en la parte superior, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces** (Importar espacios de nombres). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Computer Vision:

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. En la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Face client** (Autenticar cliente de Face). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto **FaceClient**:

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. En la función **Main**, en el código que acaba de agregar, tenga en cuenta que el código muestra un menú que le permite llamar a funciones en el código para explorar las funcionalidades del servicio Face. Implementará estas funciones en lo que queda de este ejercicio.

## <a name="detect-and-analyze-faces"></a>Detección y análisis de caras

Una de las funcionalidades más fundamentales del servicio Face es la detección de caras de una imagen y la determinación de sus atributos, como la edad, las expresiones emocionales, el color del cabello, la presencia de gafas, etc.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **1**. Este código llama a la función **DetectFaces** y pasa la ruta de acceso a un archivo de imagen.
2. Busque la función **DetectFaces** en el archivo de código y, debajo del comentario **Specify facial features to be retrieved** (Especificar características faciales que se recuperarán), agregue el código siguiente:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Age,
        FaceAttributeType.Emotion,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.age,
                FaceAttributeType.emotion,
                FaceAttributeType.glasses]
    ```

3. En la función **DetectFaces**, en el código que acaba de agregar, busque el comentario **Get faces** (Obtener caras) y agregue el código siguiente:

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            // Get face properties
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Age: {face.FaceAttributes.Age}");
            Console.WriteLine($" - Emotions:");
            foreach (var emotion in face.FaceAttributes.Emotion.ToRankedList())
            {
                Console.WriteLine($"   - {emotion}");
            }

            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            age = 'age unknown' if 'age' not in detected_attributes.keys() else int(detected_attributes['age'])
            print(' - Age: {}'.format(age))

            if 'emotion' in detected_attributes:
                print(' - Emotions:')
                for emotion_name in detected_attributes['emotion']:
                    print('   - {}: {}'.format(emotion_name, detected_attributes['emotion'][emotion_name]))
            
            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. Examine el código que agregó a la función **DetectFaces**. Analiza un archivo de imagen y detecta las caras que contiene, incluidos los atributos de edad y emociones y la presencia de gafas. Se muestran los detalles de cada cara, incluido un identificador de cara único que se asigna a cada cara. La ubicación de las caras se indica en la imagen mediante un rectángulo delimitador.
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **face-api** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    *La salida de C# puede mostrar advertencias sobre las funciones asincrónicas que ahora usan el operador **await**. No obstante, puede ignorarlas.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Cuando se le solicite, escriba **1** y observe la salida, que debe incluir el id. y los atributos de cada cara detectada.
7. Consulte el archivo **detected_faces.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas.

## <a name="compare-faces"></a>Comparación de caras

Una tarea común consiste en comparar caras y buscar caras similares. En este escenario, no es necesario *identificar* a la persona de las imágenes, sino que simplemente se debe determinar si varias imágenes muestran a la *misma* persona (o al menos a alguien con un aspecto similar). 

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **2**. Este código llama a la función **CompareFaces** y pasa la ruta de acceso a dos archivos de imagen (**person1.jpg** y **people.jpg**).
2. Busque la función **CompareFaces** en el archivo de código y, en el código existente que imprime un mensaje en la consola, agregue el código siguiente:

**C#**

```C
// Determine if the face in image 1 is also in image 2
DetectedFace image_i_face;
using (var image1Data = File.OpenRead(image1))
{    
    // Get the first face in image 1
    var image1_faces = await faceClient.Face.DetectWithStreamAsync(image1Data);
    if (image1_faces.Count > 0)
    {
        image_i_face = image1_faces[0];
        Image img1 = Image.FromFile(image1);
        Graphics graphics = Graphics.FromImage(img1);
        Pen pen = new Pen(Color.LightGreen, 3);
        var r = image_i_face.FaceRectangle;
        Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        String output_file = "face_to_match.jpg";
        img1.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file); 

        //Get all the faces in image 2
        using (var image2Data = File.OpenRead(image2))
        {    
            var image2Faces = await faceClient.Face.DetectWithStreamAsync(image2Data);

            // Get faces
            if (image2Faces.Count > 0)
            {

                var image2FaceIds = image2Faces.Select(f => f.FaceId).ToList<Guid?>();
                var similarFaces = await faceClient.Face.FindSimilarAsync((Guid)image_i_face.FaceId,faceIds:image2FaceIds);
                var similarFaceIds = similarFaces.Select(f => f.FaceId).ToList<Guid?>();

                // Prepare image for drawing
                Image img2 = Image.FromFile(image2);
                Graphics graphics2 = Graphics.FromImage(img2);
                Pen pen2 = new Pen(Color.LightGreen, 3);
                Font font2 = new Font("Arial", 4);
                SolidBrush brush2 = new SolidBrush(Color.Black);

                // Draw and annotate each face
                foreach (var face in image2Faces)
                {
                    if (similarFaceIds.Contains(face.FaceId))
                    {
                        // Draw and annotate face
                        var r2 = face.FaceRectangle;
                        Rectangle rect2 = new Rectangle(r2.Left, r2.Top, r2.Width, r2.Height);
                        graphics2.DrawRectangle(pen2, rect2);
                        string annotation = "Match!";
                        graphics2.DrawString(annotation,font2,brush2,r2.Left, r2.Top);
                    }
                }

                // Save annotated image
                String output_file2 = "matched_faces.jpg";
                img2.Save(output_file2);
                Console.WriteLine(" Results saved in " + output_file2);   
            }
        }

    }
}
```

**Python**

```Python
# Determine if the face in image 1 is also in image 2
with open(image_1, mode="rb") as image_data:
    # Get the first face in image 1
    image_1_faces = face_client.face.detect_with_stream(image=image_data)
    image_1_face = image_1_faces[0]

    # Highlight the face in the image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_1)
    draw = ImageDraw.Draw(image)
    color = 'lightgreen'
    r = image_1_face.face_rectangle
    bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
    draw = ImageDraw.Draw(image)
    draw.rectangle(bounding_box, outline=color, width=5)
    plt.imshow(image)
    outputfile = 'face_to_match.jpg'
    fig.savefig(outputfile)

# Get all the faces in image 2
with open(image_2, mode="rb") as image_data:
    image_2_faces = face_client.face.detect_with_stream(image=image_data)
    image_2_face_ids = list(map(lambda face: face.face_id, image_2_faces))

    # Find faces in image 2 that are similar to the one in image 1
    similar_faces = face_client.face.find_similar(face_id=image_1_face.face_id, face_ids=image_2_face_ids)
    similar_face_ids = list(map(lambda face: face.face_id, similar_faces))

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_2)
    draw = ImageDraw.Draw(image)

    # Draw and annotate matching faces
    for face in image_2_faces:
        if face.face_id in similar_face_ids:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline='lightgreen', width=10)
            plt.annotate('Match!',(r.left, r.top + r.height + 15), backgroundcolor='white')

    # Save annotated image
    plt.imshow(image)
    outputfile = 'matched_faces.jpg'
    fig.savefig(outputfile)
```

3. Examine el código que agregó a la función **CompareFaces**. Busca la primera cara en la imagen 1 y la anota en un nuevo archivo de imagen denominado **face_to_match.jpg**. A continuación, busca todas las caras en la imagen 2 y usa sus id. de cara para buscar las que son similares a las de la imagen 1. Las caras similares se anotan y se guardan en una nueva imagen denominada **matched_faces.jpg**.
4. Guarde los cambios y vuelva al terminal integrado de la carpeta **face-api** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    *La salida de C# puede mostrar advertencias sobre las funciones asincrónicas que ahora usan el operador **await**. No obstante, puede ignorarlas.*

    **Python**

    ```
    python analyze-faces.py
    ```
    
5. Cuando se le solicite, escriba **2** y observe la salida. A continuación, consulte los archivos **face_to_match.jpg** y **matched_faces.jpg** que se generan en la misma carpeta que el archivo de código para ver las caras que coinciden.
6. Edite el código de la función **Main** para la opción de menú **2** a fin de comparar **person2.jpg** con **people.jpg** y, a continuación, vuelva a ejecutar el programa y consulte los resultados.

## <a name="train-a-facial-recognition-model"></a>Entrenamiento de un modelo de reconocimiento facial

Puede haber escenarios en los que necesite mantener un modelo de personas específicas cuyas caras puedan ser reconocidas por una aplicación de inteligencia artificial. Por ejemplo, para facilitar un sistema de seguridad biométrica que use el reconocimiento facial para comprobar la identidad de los empleados en un edificio seguro.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **3**. Este código llama a la función **TrainModel** y pasa el nombre y el id. de un nuevo objeto **PersonGroup** que se registrará en el recurso de Cognitive Services, así como una lista de nombres de empleados.
2. En la carpeta **face-api/images**, observe que hay carpetas con los mismos nombres que los empleados. Cada carpeta contiene varias imágenes del empleado con nombre.
3. Busque la función **TrainModel** en el archivo de código y, en el código existente que imprime un mensaje en la consola, agregue el código siguiente:

**C#**

```C
// Delete group if it already exists
var groups = await faceClient.PersonGroup.ListAsync();
foreach(var group in groups)
{
    if (group.PersonGroupId == groupId)
    {
        await faceClient.PersonGroup.DeleteAsync(groupId);
    }
}

// Create the group
await faceClient.PersonGroup.CreateAsync(groupId, groupName);
Console.WriteLine("Group created!");

// Add each person to the group
Console.Write("Adding people to the group...");
foreach(var personName in imageFolders)
{
    // Add the person
    var person = await faceClient.PersonGroupPerson.CreateAsync(groupId, personName);

    // Add multiple photo's of the person
    string[] images = Directory.GetFiles("images/" + personName);
    foreach(var image in images)
    {
        using (var imageData = File.OpenRead(image))
        { 
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(groupId, person.PersonId, imageData);
        }
    }

}

    // Train the model
Console.WriteLine("Training model...");
await faceClient.PersonGroup.TrainAsync(groupId);

// Get the list of people in the group
Console.WriteLine("Facial recognition model trained with the following people:");
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    Console.WriteLine($"-{person.Name}");
}
```

**Python**

```Python
# Delete group if it already exists
groups = face_client.person_group.list()
for group in groups:
    if group.person_group_id == group_id:
        face_client.person_group.delete(group_id)

# Create the group
face_client.person_group.create(group_id, group_name)
print ('Group created!')

# Add each person to the group
print('Adding people to the group...')
for person_name in image_folders:
    # Add the person
    person = face_client.person_group_person.create(group_id, person_name)

    # Add multiple photo's of the person
    folder = os.path.join('images', person_name)
    person_pics = os.listdir(folder)
    for pic in person_pics:
        img_path = os.path.join(folder, pic)
        img_stream = open(img_path, "rb")
        face_client.person_group_person.add_face_from_stream(group_id, person.person_id, img_stream)

# Train the model
print('Training model...')
face_client.person_group.train(group_id)

# Get the list of people in the group
print('Facial recognition model trained with the following people:')
people = face_client.person_group_person.list(group_id)
for person in people:
    print('-', person.name)
```

4. Examine el código que agregó a la función **TrainModel**. Realiza las siguientes tareas:
    - Obtiene una lista de objetos **PersonGroup** registrados en el servicio y elimina el especificado si existe.
    - Crea un grupo con el id. y el nombre especificados.
    - Agrega una persona al grupo para cada nombre especificado e incluye las diferentes imágenes de cada persona.
    - Entrena un modelo de reconocimiento facial basado en las personas con nombre del grupo y las imágenes de sus caras.
    - Recupera una lista de las personas con nombre del grupo y las muestra.
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **face-api** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    *La salida de C# puede mostrar advertencias sobre las funciones asincrónicas que ahora usan el operador **await**. No obstante, puede ignorarlas.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Cuando se le solicite, escriba **3** y observe la salida. Fíjese en que el objeto **PersonGroup** creado incluye dos personas.

## <a name="recognize-faces"></a>Reconocimiento de caras

Ahora que ha definido un objeto **PeopleGroup** y entrenado un modelo de reconocimiento facial, puede usarlo para reconocer a individuos con nombre en una imagen.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **4**. Este código llama a la función **RecognizeFaces** y pasa la ruta de acceso a un archivo de imagen (**people.jpg**) y el id. del objeto **PeopleGroup** que se usará para la identificación facial.
2. Busque la función **RecognizeFaces** en el archivo de código y, en el código existente que imprime un mensaje en la consola, agregue el código siguiente:

**C#**

```C
// Detect faces in the image
using (var imageData = File.OpenRead(imageFile))
{    
    var detectedFaces = await faceClient.Face.DetectWithStreamAsync(imageData);

    // Get faces
    if (detectedFaces.Count > 0)
    {
        
        // Get a list of face IDs
        var faceIds = detectedFaces.Select(f => f.FaceId).ToList<Guid?>();

        // Identify the faces in the people group
        var recognizedFaces = await faceClient.Face.IdentifyAsync(faceIds, groupId);

        // Get names for recognized faces
        var faceNames = new Dictionary<Guid?, string>();
        if (recognizedFaces.Count> 0)
        {
            foreach(var face in recognizedFaces)
            {
                var person = await faceClient.PersonGroupPerson.GetAsync(groupId, face.Candidates[0].PersonId);
                Console.WriteLine($"-{person.Name}");
                faceNames.Add(face.FaceId, person.Name);

            }
        }

        
        // Annotate faces in image
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen penYes = new Pen(Color.LightGreen, 3);
        Pen penNo = new Pen(Color.Magenta, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Cyan);
        foreach (var face in detectedFaces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            if (faceNames.ContainsKey(face.FaceId))
            {
                // If the face is recognized, annotate in green with the name
                graphics.DrawRectangle(penYes, rect);
                string personName = faceNames[face.FaceId];
                graphics.DrawString(personName,font,brush,r.Left, r.Top);
            }
            else
            {
                // Otherwise, just annotate the unrecognized face in magenta
                graphics.DrawRectangle(penNo, rect);
            }
        }

        // Save annotated image
        String output_file = "recognized_faces.jpg";
        image.Save(output_file);
        Console.WriteLine("Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Detect faces in the image
with open(image_file, mode="rb") as image_data:

    # Get faces
    detected_faces = face_client.face.detect_with_stream(image=image_data)

    # Get a list of face IDs
    face_ids = list(map(lambda face: face.face_id, detected_faces))

    # Identify the faces in the people group
    recognized_faces = face_client.face.identify(face_ids, group_id)

    # Get names for recognized faces
    face_names = {}
    if len(recognized_faces) > 0:
        print(len(recognized_faces), 'faces recognized.')
        for face in recognized_faces:
            person_name = face_client.person_group_person.get(group_id, face.candidates[0].person_id).name
            print('-', person_name)
            face_names[face.face_id] = person_name

    # Annotate faces in image
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    for face in detected_faces:
        r = face.face_rectangle
        bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
        draw = ImageDraw.Draw(image)
        if face.face_id in face_names:
            # If the face is recognized, annotate in green with the name
            draw.rectangle(bounding_box, outline='lightgreen', width=3)
            plt.annotate(face_names[face.face_id],
                        (r.left, r.top + r.height + 15), backgroundcolor='white')
        else:
            # Otherwise, just annotate the unrecognized face in magenta
            draw.rectangle(bounding_box, outline='magenta', width=3)

    # Save annotated image
    plt.imshow(image)
    outputfile = 'recognized_faces.jpg'
    fig.savefig(outputfile)

    print('\nResults saved in', outputfile)
```
    
3. Examine el código que agregó a la función **RecognizeFaces**. Busca las caras de la imagen y crea una lista de sus id. A continuación, usa el grupo de personas para intentar identificar las caras en la lista de id. de cara. Las caras reconocidas se anotan con el nombre de la persona identificada y los resultados se guardan en **recognized_faces.jpg**.
4. Guarde los cambios y vuelva al terminal integrado de la carpeta **face-api** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    *La salida de C# puede mostrar advertencias sobre las funciones asincrónicas que ahora usan el operador **await**. No obstante, puede ignorarlas.*

    **Python**

    ```
    python analyze-faces.py
    ```

5. Cuando se le solicite, escriba **4** y observe la salida. A continuación, consulte el archivo **recognized_faces.jpg** que se genera en la misma carpeta que el archivo de código para ver las personas identificadas.
6. Edite el código de la función **Main** para la opción de menú **4** a fin de reconocer las caras de **people2.jpg** y, a continuación, vuelva a ejecutar el programa y consulte los resultados. Se deben reconocer las mismas personas.

## <a name="verify-a-face"></a>Verificación de una cara

El reconocimiento facial se usa a menudo para la verificación de identidad. Con el servicio Face, puede verificar una cara de una imagen mediante la comparación con otra cara o con la de una persona registrada en un objeto **PersonGroup**.

1. En el archivo de código de la aplicación, en la función **Main**, examine el código que se ejecuta si el usuario selecciona la opción de menú **5**. Este código llama a la función **VerifyFace** y pasa la ruta de acceso a un archivo de imagen (**person1.jpg**), un nombre y el id. del objeto **PeopleGroup** que se usarán para la identificación facial.
2. Busque la función **VerifyFace** en el archivo de código y, debajo el comentario **Get the ID of the person from the people group** (Obtener el id. de la persona del grupo de personas) situado encima del código que imprime el resultado, agregue el código siguiente:

**C#**

```C
// Get the ID of the person from the people group
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    if (person.Name == personName)
    {
        Guid personId = person.PersonId;

        // Get the first face in the image
        using (var imageData = File.OpenRead(personImage))
        {    
            var faces = await faceClient.Face.DetectWithStreamAsync(imageData);
            if (faces.Count > 0)
            {
                Guid faceId = (Guid)faces[0].FaceId;

                //We have a face and an ID. Do they match?
                var verification = await faceClient.Face.VerifyFaceToPersonAsync(faceId, personId, groupId);
                if (verification.IsIdentical)
                {
                    result = "Verified";
                }
            }
        }
    }
}
```

**Python**

```Python
# Get the ID of the person from the people group
people = face_client.person_group_person.list(group_id)
for person in people:
    if person.name == person_name:
        person_id = person.person_id

        # Get the first face in the image
        with open(person_image, mode="rb") as image_data:
            faces = face_client.face.detect_with_stream(image=image_data)
            if len(faces) > 0:
                face_id = faces[0].face_id

                # We have a face and an ID. Do they match?
                verification = face_client.face.verify_face_to_person(face_id, person_id, group_id)
                if verification.is_identical:
                    result = 'Verified'
```

3. Examine el código que agregó a la función **VerifyFace**. Busca el id. de la persona del grupo con el nombre especificado. Si la persona existe, el código obtiene el id. de cara de la primera cara de la imagen. Por último, si hay una cara en la imagen, el código la comprueba con el id. de la persona especificada.
4. Guarde los cambios y vuelva al terminal integrado de la carpeta **face-api** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python analyze-faces.py
    ```

5. Cuando se le solicite, escriba **5** y observe el resultado.
6. Edite el código de la función **Main** de la opción de menú **5** a fin de experimentar con diferentes combinaciones de nombres y las imágenes **person1.jpg** y **person2.jpg**.

## <a name="more-information"></a>Más información

Para obtener más información sobre cómo usar el servicio **Computer Vision** para la detección de caras, consulte la [documentación de Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Para obtener más información sobre el servicio **Face**, consulte la [documentación de Face](https://docs.microsoft.com/azure/cognitive-services/face/).
