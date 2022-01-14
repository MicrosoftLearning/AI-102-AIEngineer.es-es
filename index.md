---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
ms.openlocfilehash: ba3d2a5154e5e9b08e750f7d9ced8b62048dd66d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625913"
---
# <a name="content-directory"></a>Directorio de contenido

Este repositorio contiene los ejercicios de laboratorio práctico del curso de Microsoft [AI-102: Diseño e implementación de una solución de Microsoft Azure AI](https://docs.microsoft.com/learn/certifications/courses/ai-102t00) y los [módulos autodirigidos equivalentes de Microsoft Learn](https://aka.ms/AzureLearn_AIEngineer). Los ejercicios están pensados para complementar los materiales de aprendizaje y permiten practicar el uso de las tecnologías descritas en estos materiales.

Para completar estos ejercicios, necesitará una suscripción a Microsoft Azure. Si su instructor no se la ha proporcionado, puede registrarse para obtener una evaluación gratuita en[https://azure.microsoft.com](https://azure.microsoft.com).

## <a name="labs"></a>Laboratorios

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| Módulo | Laboratorio |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

