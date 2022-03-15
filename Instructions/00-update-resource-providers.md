---
lab:
  title: Habilitar proveedores de recursos
  module: Setup
ms.openlocfilehash: 2cd48d0d9f67b9bab3e64452bf6199e1f438492a
ms.sourcegitcommit: e06fbeaa3bc15e4dbe99f72216dfeeb27f58babd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/15/2022
ms.locfileid: "138765495"
---
# <a name="enable-resource-providers"></a>Habilitar proveedores de recursos

Hay algunos proveedores de recursos que se deben estar registrados en la suscripción de Azure. Siga estos pasos para asegurarse de que están registrados.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` con las credenciales de Microsoft asociadas a su suscripción.
2. En la página **Inicio**, seleccione **Suscripciones** (o amplíe el menú **&#8801;** , seleccione **Todos los servicios** y, en la categoría **General**, seleccione **Suscripciones**).
3. Seleccione su suscripción de Azure (si tiene varias suscripciones, seleccione la que creó canjeando el Pase para Azure).
4. En la hoja de la suscripción, en el panel de la izquierda, en la sección **Configuración**, seleccione **Proveedores de recursos**.
5. En la lista de proveedores de recursos, asegúrese de que están registrados los siguientes proveedores (si no es así, selecciónelos y haga clic en **registrar**):
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - microsoft.insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
