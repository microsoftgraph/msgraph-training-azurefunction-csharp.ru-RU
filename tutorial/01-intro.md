---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445971"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="113cb-101">В этом руководстве рассказывается о создании функции Azure, которая использует API microsoft Graph для получения сведений о календаре для пользователя.</span><span class="sxs-lookup"><span data-stu-id="113cb-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="113cb-102">Если вы предпочитаете просто скачать завершенный учебник, вы можете скачать или клонировать [GitHub репозиторий](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span><span class="sxs-lookup"><span data-stu-id="113cb-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="113cb-103">Инструкции по настройке  приложения с помощью ID-приложения и секрета см. в файле README в демо-папке.</span><span class="sxs-lookup"><span data-stu-id="113cb-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="113cb-104">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="113cb-104">Prerequisites</span></span>

<span data-ttu-id="113cb-105">Перед началом этого учебного пособия на компьютере разработки должны быть установлены следующие средства.</span><span class="sxs-lookup"><span data-stu-id="113cb-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="113cb-106">.NET Core SDK</span><span class="sxs-lookup"><span data-stu-id="113cb-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="113cb-107">Основные средства Azure Functions</span><span class="sxs-lookup"><span data-stu-id="113cb-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="113cb-108">Azure CLI</span><span class="sxs-lookup"><span data-stu-id="113cb-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="113cb-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="113cb-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="113cb-110">Вы также должны иметь учетную запись Microsoft work или school с доступом к глобальной учетной записи администратора в той же организации.</span><span class="sxs-lookup"><span data-stu-id="113cb-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="113cb-111">Если у вас нет учетной записи Майкрософт, вы можете зарегистрироваться в программе Office 365 разработчика, чтобы получить бесплатную Office 365 подписку. [](https://developer.microsoft.com/office/dev-program)</span><span class="sxs-lookup"><span data-stu-id="113cb-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="113cb-112">Этот учебник был написан со следующими версиями вышеуказанных инструментов.</span><span class="sxs-lookup"><span data-stu-id="113cb-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="113cb-113">Действия в этом руководстве могут работать с другими версиями, но они не были проверены.</span><span class="sxs-lookup"><span data-stu-id="113cb-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="113cb-114">.NET Core SDK 5.0.203</span><span class="sxs-lookup"><span data-stu-id="113cb-114">.NET Core SDK 5.0.203</span></span>
> - <span data-ttu-id="113cb-115">Основные средства Azure Functions 3.0.3442</span><span class="sxs-lookup"><span data-stu-id="113cb-115">Azure Functions Core Tools 3.0.3442</span></span>
> - <span data-ttu-id="113cb-116">Azure CLI 2.23.0</span><span class="sxs-lookup"><span data-stu-id="113cb-116">Azure CLI 2.23.0</span></span>
> - <span data-ttu-id="113cb-117">ngrok 2.3.40</span><span class="sxs-lookup"><span data-stu-id="113cb-117">ngrok 2.3.40</span></span>

## <a name="feedback"></a><span data-ttu-id="113cb-118">Отзывы</span><span class="sxs-lookup"><span data-stu-id="113cb-118">Feedback</span></span>

<span data-ttu-id="113cb-119">В репозитории [](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)GitHub.</span><span class="sxs-lookup"><span data-stu-id="113cb-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
