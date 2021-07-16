---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445971"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве рассказывается о создании функции Azure, которая использует API microsoft Graph для получения сведений о календаре для пользователя.

> [!TIP]
> Если вы предпочитаете просто скачать завершенный учебник, вы можете скачать или клонировать [GitHub репозиторий](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp). Инструкции по настройке  приложения с помощью ID-приложения и секрета см. в файле README в демо-папке.

## <a name="prerequisites"></a>Предварительные требования

Перед началом этого учебного пособия на компьютере разработки должны быть установлены следующие средства.

- [.NET Core SDK](https://dotnet.microsoft.com/download)
- [Основные средства Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

Вы также должны иметь учетную запись Microsoft work или school с доступом к глобальной учетной записи администратора в той же организации. Если у вас нет учетной записи Майкрософт, вы можете зарегистрироваться в программе Office 365 разработчика, чтобы получить бесплатную Office 365 подписку. [](https://developer.microsoft.com/office/dev-program)

> [!NOTE]
> Этот учебник был написан со следующими версиями вышеуказанных инструментов. Действия в этом руководстве могут работать с другими версиями, но они не были проверены.
>
> - .NET Core SDK 5.0.203
> - Основные средства Azure Functions 3.0.3442
> - Azure CLI 2.23.0
> - ngrok 2.3.40

## <a name="feedback"></a>Отзывы

В репозитории [](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)GitHub.
