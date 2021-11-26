---
lab:
  title: Configuración del entorno de laboratorio
  module: Setup
ms.openlocfilehash: 57363ce9fec94cb3a1a6ef7622a10bca48a6f055
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625957"
---
# <a name="lab-environment-setup"></a>Configuración del entorno de laboratorio

Estos ejercicios están diseñados para completarse en un entorno de laboratorio hospedado. Si desea completarlos en su propio equipo, puede hacerlo instalando el software siguiente. Puede experimentar diálogos y comportamientos inesperados al usar su propio entorno. Debido a la amplia variedad de configuraciones locales posibles, el equipo del curso no puede cubrir problemas que puedan surgir en su propio entorno.

> **Nota**: Las instrucciones siguientes son para un equipo con Windows 10. También puede usar Linux o MacOS. Es posible que tenga que adaptar las instrucciones del laboratorio para el sistema operativo que elija.

### <a name="base-operating-system-windows-10"></a>Sistema operativo base (Windows 10)

#### <a name="windows-10"></a>Windows 10

Instale Windows 10 y aplique todas las actualizaciones.

#### <a name="edge"></a>Edge

Instale [Edge (Chromium)](https://microsoft.com/edge)

### <a name="net-core-sdk"></a>SDK de .NET Core

1. Descargue e instale en https://dotnet.microsoft.com/download (descargue el SDK de .NET Core, no solo el runtime)

### <a name="c-redistributable"></a>Paquete redistribuible de C++

1. Descargue e instale Visual C++ Redistributable (x64) en https://aka.ms/vs/16/release/vc_redist.x64.exe.

### <a name="nodejs"></a>Node.JS

1. Descargue la versión LTS más reciente en https://nodejs.org/en/download/ 
2. Instale con las opciones predeterminadas

### <a name="python-and-required-packages"></a>Python (y paquetes necesarios)

1. Descargue la versión 3.8 en https://docs.conda.io/en/latest/miniconda.html 
2. Ejecute el programa de instalación para instalar. **Importante**: seleccione las opciones para agregar Miniconda a la variable PATH y para registrar Miniconda como el entorno de Python predeterminado.
3. Después de la instalación, abra el símbolo del sistema de Anaconda y escriba los siguientes comandos para instalar paquetes: 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### <a name="azure-cli"></a>Azure CLI

1. Descargue en https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest 
2. Instale con las opciones predeterminadas

### <a name="git"></a>Git

1. Descargue e instale en https://git-scm.com/download.html, con las opciones predeterminadas


### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code (y extensiones)

1. Descargue en https://code.visualstudio.com/Download 
2. Instale con las opciones predeterminadas 
3. Después de la instalación, inicie Visual Studio Code y, en la pestaña **Extensiones** (Ctrl + Mayús + X), busque e instale las siguientes extensiones de Microsoft:
    - Python
    - C#
    - Azure Functions
    - PowerShell


### <a name="bot-framework-emulator"></a>Bot Framework Emulator

Siga las instrucciones de https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md para descargar e instalar la versión estable más reciente de Bot Framework Emulator para el sistema operativo.

### <a name="bot-framework-composer"></a>Bot Framework Composer

Instalar desde https://docs.microsoft.com/en-us/composer/install-composer.
