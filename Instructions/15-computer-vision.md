---
lab:
  title: Análisis de imágenes con Computer Vision
  module: Module 8 - Getting Started with Computer Vision
ms.openlocfilehash: f2ee18ff682d53e9fd554749ed2b9cbaa9b03611
ms.sourcegitcommit: 7191e53bc33cda92e710d957dde4478ee2496660
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/09/2022
ms.locfileid: "147041670"
---
# <a name="analyze-images-with-computer-vision"></a>Análisis de imágenes con Computer Vision

Computer Vision es una funcionalidad de inteligencia artificial que permite a los sistemas de software interpretar la entrada visual mediante el análisis de imágenes. En Microsoft Azure, **Computer Vision** de Cognitive Services proporciona modelos predefinidos para tareas comunes de Computer Vision, incluido el análisis de imágenes para sugerir títulos y etiquetas, la detección de objetos comunes, puntos de referencia, celebridades, marcas y la presencia de contenido para adultos. También puede usar el servicio Computer Vision para analizar el color y los formatos de la imagen, y para generar imágenes en miniatura "recortadas inteligentemente".

## <a name="clone-the-repository-for-this-course"></a>Clonación del repositorio para este curso

Si aún no ha clonado el repositorio de código **AI-102-AIEngineer** en el entorno en el que está trabajando en este laboratorio, siga estos pasos para hacerlo. De lo contrario, abra la carpeta clonada en Visual Studio Code.

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
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Keys and Endpoint** (Claves y punto de conexión). Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## <a name="prepare-to-use-the-computer-vision-sdk"></a>Preparación para el uso del SDK de Computer Vision

En este ejercicio, completará una aplicación cliente parcialmente implementada que usa el SDK de Computer Vision para analizar imágenes.

> **Nota**: Puede elegir usar el SDK para **C#** o **Python**. En los pasos siguientes, realice las acciones adecuadas para su lenguaje preferido.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **15-face-computer-vision** y expanda la carpeta **C-Sharp** o **Python** según sus preferencias de lenguaje.
2. Haga clic con el botón derecho en la carpeta **image-analysis** y abra un terminal integrado. A continuación, instale el paquete del SDK de Computer Vision mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. Consulte el contenido de la carpeta **image-analysis** y observe que contiene un archivo para las opciones de configuración:
    - **C#** : appsettings.json
    - **Python**: .env

    Abra el archivo de configuración y actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Cognitive Services. Guarde los cambios.
4. Observe que la carpeta **image-analysis** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: image-analysis.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Computer Vision:

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
    
## <a name="view-the-images-you-will-analyze"></a>Visualización de las imágenes que va a analizar

En este ejercicio, usará el servicio Computer Vision para analizar varias imágenes.

1. En Visual Studio Code, expanda la carpeta **image-analysis** y la carpeta **images** que contiene.
2. Seleccione cada uno de los archivos de imagen por turnos para verlos en Visual Studio Code.

## <a name="analyze-an-image-to-suggest-a-caption"></a>Análisis de una imagen para sugerir un título

Ahora ya puede usar el SDK para llamar al servicio Computer Vision y analizar una imagen.

1. En el archivo de código de la aplicación cliente (**Program.cs** o **image-analysis.py**), en la función **Main**, observe que se ha proporcionado el código para cargar los valores de configuración. A continuación, busque el comentario **Authenticate Computer Vision client** (Autenticar cliente de Computer Vision). A continuación, en este comentario, agregue el siguiente código específico del lenguaje para crear y autenticar un objeto de cliente de Computer Vision:

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

2. En la función **Main**, en el código que acaba de agregar, observe que el código especifica la ruta de acceso a un archivo de imagen y, a continuación, pasa la ruta de acceso de la imagen a otras dos funciones (**AnalyzeImage** y **GetThumbnail**). Estas funciones aún no están totalmente implementadas.

3. En la función **AnalyzeImage**, en el comentario **Specify features to be retrieved** (Especificar características que se recuperarán), agregue el código siguiente:

**C#**

```C#
// Specify features to be retrieved
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Description,
    VisualFeatureTypes.Tags,
    VisualFeatureTypes.Categories,
    VisualFeatureTypes.Brands,
    VisualFeatureTypes.Objects,
    VisualFeatureTypes.Adult
};
```

**Python**

```Python
# Specify features to be retrieved
features = [VisualFeatureTypes.description,
            VisualFeatureTypes.tags,
            VisualFeatureTypes.categories,
            VisualFeatureTypes.brands,
            VisualFeatureTypes.objects,
            VisualFeatureTypes.adult]
```
    
4. En la función **AnalyzeImage**, en el comentario **Get image analysis** (Obtener análisis de imágenes), agregue el código siguiente (incluidos los comentarios que indican dónde agregará más código con posterioridad):

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // get image captions
    foreach (var caption in analysis.Description.Captions)
    {
        Console.WriteLine($"Description: {caption.Text} (confidence: {caption.Confidence.ToString("P")})");
    }

    // Get image tags


    // Get image categories


    // Get brands in the image


    // Get objects in the image


    // Get moderation ratings
    

}            
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

# Get image description
for caption in analysis.description.captions:
    print("Description: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get image categories 


# Get brands in the image


# Get objects in the image


# Get moderation ratings

```
    
5. Guarde los cambios y vuelva al terminal integrado de la carpeta **image-analysis** y escriba el siguiente comando para ejecutar el programa con el argumento **images/street.jpg**:

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Observe la salida, que debe incluir un título sugerido para la imagen **street.jpg**.
7. Vuelva a ejecutar el programa, esta vez con el argumento **images/building.jpg** para ver el título que se genera para la imagen **building.jpg**.
8. Repita el paso anterior para generar un título para el archivo **images/person.jpg**.

## <a name="get-suggested-tags-for-an-image"></a>Obtención de etiquetas sugeridas para una imagen

A veces puede ser útil identificar *etiquetas* relevantes que proporcionan pistas sobre el contenido de una imagen.

1. En la función **AnalyzeImage**, en el comentario **Get image tags** (Obtener etiquetas de imagen), agregue el código siguiente:

**C#**

```C
// Get image tags
if (analysis.Tags.Count > 0)
{
    Console.WriteLine("Tags:");
    foreach (var tag in analysis.Tags)
    {
        Console.WriteLine($" -{tag.Name} (confidence: {tag.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image tags
if (len(analysis.tags) > 0):
    print("Tags: ")
    for tag in analysis.tags:
        print(" -'{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe que, además del título de la imagen, se muestra una lista de etiquetas sugeridas.

## <a name="get-image-categories"></a>Obtención de categorías de imagen

El servicio Computer Vision puede sugerir *categorías* para imágenes y, dentro de cada categoría, puede identificar puntos de referencia conocidos.

1. En la función **AnalyzeImage**, en el comentario **Get image categories** (Obtener categorías de imagen), agregue el código siguiente:

**C#**

```C
// Get image categories
List<LandmarksModel> landmarks = new List<LandmarksModel> {};
Console.WriteLine("Categories:");
foreach (var category in analysis.Categories)
{
    // Print the category
    Console.WriteLine($" -{category.Name} (confidence: {category.Score.ToString("P")})");

    // Get landmarks in this category
    if (category.Detail?.Landmarks != null)
    {
        foreach (LandmarksModel landmark in category.Detail.Landmarks)
        {
            if (!landmarks.Any(item => item.Name == landmark.Name))
            {
                landmarks.Add(landmark);
            }
        }
    }
}

// If there were landmarks, list them
if (landmarks.Count > 0)
{
    Console.WriteLine("Landmarks:");
    foreach(LandmarksModel landmark in landmarks)
    {
        Console.WriteLine($" -{landmark.Name} (confidence: {landmark.Confidence.ToString("P")})");
    }
}

```

**Python**

```Python
# Get image categories
if (len(analysis.categories) > 0):
    print("Categories:")
    landmarks = []
    for category in analysis.categories:
        # Print the category
        print(" -'{}' (confidence: {:.2f}%)".format(category.name, category.score * 100))
        if category.detail:
            # Get landmarks in this category
            if category.detail.landmarks:
                for landmark in category.detail.landmarks:
                    if landmark not in landmarks:
                        landmarks.append(landmark)

    # If there were landmarks, list them
    if len(landmarks) > 0:
        print("Landmarks:")
        for landmark in landmarks:
            print(" -'{}' (confidence: {:.2f}%)".format(landmark.name, landmark.confidence * 100))

```
    
2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe que, además de las etiquetas y el título de la imagen, se muestra una lista de categorías sugeridas junto con los puntos de referencia reconocidos (en particular en la imagen **building.jpg**).

## <a name="get-brands-in-an-image"></a>Obtención de las marcas de una imagen

Algunas marcas se pueden reconocer visualmente por los logotipos, incluso cuando no se muestra el nombre de la marca. El servicio Computer Vision está entrenado para identificar miles de marcas conocidas.

1. En la función **AnalyzeImage**, en el comentario **Get brands in the image** (Obtener marcas en la imagen), agregue el código siguiente:

**C#**

```C
// Get brands in the image
if (analysis.Brands.Count > 0)
{
    Console.WriteLine("Brands:");
    foreach (var brand in analysis.Brands)
    {
        Console.WriteLine($" -{brand.Name} (confidence: {brand.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get brands in the image
if (len(analysis.brands) > 0):
    print("Brands: ")
    for brand in analysis.brands:
        print(" -'{}' (confidence: {:.2f}%)".format(brand.name, brand.confidence * 100))
```
    
2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe las marcas que se identifican (especialmente, en la imagen **person.jpg**).

## <a name="detect-and-locate-objects-in-an-image"></a>Detección y localización de objetos en una imagen

La *detección de objetos* es una forma específica de Computer Vision en la que se identifican los objetos individuales dentro de una imagen y su ubicación se indica mediante un rectángulo delimitador.

1. En la función **AnalyzeImage**, en el comentario **Get objects in the image** (Obtener objetos en la imagen), agregue el código siguiente:

**C#**

```C
// Get objects in the image
if (analysis.Objects.Count > 0)
{
    Console.WriteLine("Objects in image:");

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.Black);

    foreach (var detectedObject in analysis.Objects)
    {
        // Print object name
        Console.WriteLine($" -{detectedObject.ObjectProperty} (confidence: {detectedObject.Confidence.ToString("P")})");

        // Draw object bounding box
        var r = detectedObject.Rectangle;
        Rectangle rect = new Rectangle(r.X, r.Y, r.W, r.H);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.ObjectProperty,font,brush,r.X, r.Y);

    }
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file);   
}
```

**Python**

```Python
# Get objects in the image
if len(analysis.objects) > 0:
    print("Objects in image:")

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 8))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    color = 'cyan'
    for detected_object in analysis.objects:
        # Print object name
        print(" -{} (confidence: {:.2f}%)".format(detected_object.object_property, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.rectangle
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.object_property,(r.x, r.y), backgroundcolor=color)
    # Save annotated image
    plt.imshow(image)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```
    
2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe los objetos que se detectan. Después de cada ejecución, consulte el archivo **objects.jpg** que se genera en la misma carpeta que el archivo de código para ver las caras anotadas.

## <a name="get-moderation-ratings-for-an-image"></a>Obtención de clasificaciones de moderación para una imagen

Es posible que algunas imágenes no sean adecuadas para todos los públicos y es posible que deba aplicar cierta moderación para identificar imágenes que son para adultos o violentas por naturaleza.

1. En la función **AnalyzeImage**, en el comentario **Get moderation ratings** (Obtener clasificaciones de moderación), agregue el código siguiente:

**C#**

```C
// Get moderation ratings
string ratings = $"Ratings:\n -Adult: {analysis.Adult.IsAdultContent}\n -Racy: {analysis.Adult.IsRacyContent}\n -Gore: {analysis.Adult.IsGoryContent}";
Console.WriteLine(ratings);
```

**Python**

```Python
# Get moderation ratings
ratings = 'Ratings:\n -Adult: {}\n -Racy: {}\n -Gore: {}'.format(analysis.adult.is_adult_content,
                                                                    analysis.adult.is_racy_content,
                                                                    analysis.adult.is_gory_content)
print(ratings)
```
    
2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images**, y observe las clasificación de cada imagen.

> **Nota**: En las tareas anteriores, usó un único método para analizar la imagen y, a continuación, agregó código incrementalmente para analizar y mostrar los resultados. El SDK también proporciona métodos individuales para sugerir títulos, identificar etiquetas, detectar objetos, entre otros, lo que significa que puede usar el método más adecuado para devolver solo la información que necesita, lo que reduce el tamaño de la carga de datos que se debe devolver. Consulte la [documentación del SDK de .NET](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet) o la [documentación del SDK de Python](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/computervision?view=azure-python) para más detalles.

## <a name="generate-a-thumbnail-image"></a>Generación imágenes miniatura

En algunos casos, es posible que tenga que crear una versión más pequeña de una imagen denominada *miniatura*, y recortarla para incluir el asunto visual principal dentro de nuevas dimensiones de imagen.

1. En el archivo de código, busque la función **GetThumbnail** y, en el comentario **Generate a thumbnail** (Generar una miniatura), agregue el código siguiente:

**C#**

```C
// Generate a thumbnail
using (var imageData = File.OpenRead(imageFile))
{
    // Get thumbnail data
    var thumbnailStream = await cvClient.GenerateThumbnailInStreamAsync(100, 100,imageData, true);

    // Save thumbnail image
    string thumbnailFileName = "thumbnail.png";
    using (Stream thumbnailFile = File.Create(thumbnailFileName))
    {
        thumbnailStream.CopyTo(thumbnailFile);
    }

    Console.WriteLine($"Thumbnail saved in {thumbnailFileName}");
}
```

**Python**

```Python
# Generate a thumbnail
with open(image_file, mode="rb") as image_data:
    # Get thumbnail data
    thumbnail_stream = cv_client.generate_thumbnail_in_stream(100, 100, image_data, True)

# Save thumbnail image
thumbnail_file_name = 'thumbnail.png'
with open(thumbnail_file_name, "wb") as thumbnail_file:
    for chunk in thumbnail_stream:
        thumbnail_file.write(chunk)

print('Thumbnail saved in.', thumbnail_file_name)
```
    
2. Guarde los cambios y ejecute el programa una vez para cada uno de los archivos de imagen de la carpeta **images** y abra el archivo **thumbnail.jpg** que se genera en la misma carpeta que el archivo de código para cada imagen.

## <a name="more-information"></a>Más información

En este ejercicio, ha explorado algunas de las funcionalidades de manipulación y análisis de imágenes del servicio Computer Vision. El servicio también incluye funcionalidades para leer texto, detectar caras y otras tareas de Computer Vision.

Para obtener más información sobre cómo usar el servicio **Computer Vision**, consulte la [documentación de Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/).
