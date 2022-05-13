---
lab:
  title: Uso de contenedores de Cognitive Services
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 244ab1ef3754e668d64996dece9711682651691d
ms.sourcegitcommit: 29a684646784fe4f7370343b6c005728a953770d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2022
ms.locfileid: "144557844"
---
# <a name="use-a-cognitive-services-container"></a>Uso de contenedores de Cognitive Services

El uso de Cognitive Services hospedados en Azure permite a los desarrolladores de aplicaciones centrarse en la infraestructura de su propio código, a la vez que se benefician de los servicios escalables administrados por Microsoft. Sin embargo, en muchos escenarios, las organizaciones requieren más control sobre su infraestructura de servicio y los datos que se pasan entre los servicios.

Muchas de las API de Cognitive Services se pueden empaquetar e implementar en un *contenedor*, lo que permite a las organizaciones hospedar Cognitive Services en su propia infraestructura; por ejemplo, en servidores de Docker locales, Azure Container Instances o clústeres de Azure Kubernetes Services. Cognitive Services en contenedores deben comunicarse con una cuenta de Cognitive Services basada en Azure para admitir la facturación; pero los datos de la aplicación no se pasan al servicio back-end y las organizaciones tienen un mayor control sobre la configuración de implementación de sus contenedores, lo que permite soluciones personalizadas para la autenticación, la escalabilidad y otras consideraciones.

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
4. Espere a que se complete la implementación y, a continuación, consulte los detalles.
5. Cuando se haya implementado el recurso, vaya a él y vea su página **Keys and Endpoint** (Claves y punto de conexión). Necesitará el punto de conexión y una de las claves de esta página en el procedimiento siguiente.

## <a name="deploy-and-run-a-text-analytics-container"></a>Implementación y ejecución de un contenedor de Text Analytics

Muchas API de Cognitive Services de uso frecuente están disponibles en imágenes de contenedor. Para obtener una lista completa, consulte la [documentación de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#container-availability-in-azure-cognitive-services). En este ejercicio, usará la imagen de contenedor para la API de *detección de idioma* de Text Analytics, pero los principios son los mismos para todas las imágenes disponibles.

1. En Azure Portal, en la página **Inicio**, seleccione el botón **+Crear un recurso**, busque *Container Instances* y cree un recurso de **Container Instances** con la siguiente configuración:

    - **Aspectos básicos**:
        - **Suscripción**: *suscripción de Azure*
        - **Grupo de recursos**: *elija el grupo de recursos que contiene el recurso de Cognitive Services*.
        - **Nombre de contenedor**: *escriba un nombre único*.
        - **Región**: *elija cualquier región disponible*
        - **Origen de imagen:** Otro registro
        - **Tipo de imagen**: pública
        - **Imagen**: `mcr.microsoft.com/azure-cognitive-services/textanalytics/language:1.1.013570001-amd64`
        - **Tipo de SO**: Linux
        - **Tamaño**: 1 vcpu, 4 GB de memoria
    - **Redes**:
        - **Tipo de red**: pública
        - **Etiqueta de nombre DNS**: *escriba un nombre único para el punto de conexión del contenedor*
        - **Puertos**: *cambie el puerto TCP 80 a **5000***
    - **Avanzado**:
        - **Directiva de reinicio**: en caso de error
        - **Variables de entorno**:

            | Marcar como seguro | Clave | Value |
            | -------------- | --- | ----- |
            | Sí | `ApiKey` | *Cualquier clave del recurso de Cognitive Services* |
            | Yes | `Billing` | *El URI de punto de conexión del recurso de Cognitive Services* |
            | No | `Eula` | `accept` |

        - **Invalidación de comando**: [ ]
    - **Etiquetas**:
        - *No agregue ninguna etiqueta*

2. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
3. Observe las siguientes propiedades del recurso de instancia de contenedor en su página **Información general**:
    - **Estado**: debe ser *En ejecución*.
    - **Dirección IP**: se trata de la dirección IP pública que puede usar para acceder a las instancias de contenedor.
    - **FQDN**: se trata del *nombre de dominio completo* del recurso de instancias de contenedor; puede usarlo para acceder a las instancias de contenedor en lugar de la dirección IP.

    > **Nota**: En este ejercicio, ha implementado la imagen de contenedor de Cognitive Services para la traducción de texto a un recurso de Azure Container Instances (ACI). Puede usar un enfoque similar para implementarlo en un host de *[Docker](https://www.docker.com/products/docker-desktop)* en su propio equipo o red mediante la ejecución del siguiente comando (en una sola línea) para implementar el contenedor de detección de idioma en la instancia local de Docker, reemplazando *&lt;yourEndpoint&gt;* y *&lt;yourKey&gt;* por el URI del punto de conexión y cualquiera de las claves del recurso de Cognitive Services.
    >
    > ```
    > docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 mcr.microsoft.com/azure-cognitive-services/textanalytics/language Eula=accept Billing=<yourEndpoint> ApiKey=<yourKey>
    > ```
    >
    > El comando buscará la imagen en el equipo local y, si no la encuentra, la extraerá del registro de imágenes *mcr.microsoft.com* y la implementará en la instancia de Docker. Una vez completada la implementación, el contenedor se iniciará y escuchará las solicitudes entrantes en el puerto 5000.

## <a name="use-the-container"></a>Uso del contenedor

1. En Visual Studio Code, en la carpeta **04-containers**, abra **rest-test.cmd** y edite el comando **curl** que contiene (que se muestra a continuación), reemplazando *&lt;your_ACI_IP_address_or_FQDN&gt;* por la dirección IP o FQDN del contenedor.

    ```
    curl -X POST "http://<your_ACI_IP_address_or_FQDN>:5000/text/analytics/v3.0/languages?" -H "Content-Type: application/json" --data-ascii "{'documents':[{'id':1,'text':'Hello world.'},{'id':2,'text':'Salut tout le monde.'}]}"
    ```

2. Guarde los cambios en el script. Tenga en cuenta que no es necesario especificar el punto de conexión o la clave de Cognitive Services, el servicio en contenedores procesa la solicitud. A su vez, el contenedor se comunica periódicamente con el servicio en Azure para notificar el uso para la facturación, pero no envía datos de solicitud.
3. Haga clic con el botón derecho en la carpeta **04-containers** y abra un terminal integrado. A continuación, escriba el siguiente comando para ejecutar el script:

    ```
    rest-test
    ```

4. Compruebe que el comando devuelve un documento JSON que contiene información sobre el idioma detectado en los dos documentos de entrada (que debe ser inglés y francés).

## <a name="clean-up"></a>Limpieza

Si ha terminado de experimentar con la instancia de contenedor, debe eliminarla.

1. En Azure Portal, abra el grupo de recursos donde creó los recursos para este ejercicio.
2. Seleccione el recurso de instancia de contenedor y elimínelo.

## <a name="more-information"></a>Más información

Para más información sobre Cognitive Services en contenedores, consulte la [documentación sobre contenedores de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/containers/).
