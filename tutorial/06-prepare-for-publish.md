---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445950"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f5b1b-101">В этом упражнении вы узнаете, какие изменения необходимы примеру Azure Function для подготовки к публикации в [приложении Azure Functions.](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)</span><span class="sxs-lookup"><span data-stu-id="f5b1b-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="f5b1b-102">Обновление кода</span><span class="sxs-lookup"><span data-stu-id="f5b1b-102">Update code</span></span>

<span data-ttu-id="f5b1b-103">Конфигурация читается из секретного магазина пользователя, который применяется только к вашей машине разработки.</span><span class="sxs-lookup"><span data-stu-id="f5b1b-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="f5b1b-104">Перед публикацией в Azure необходимо изменить место хранения конфигурации и соответствующим образом обновить код **в Program.cs.**</span><span class="sxs-lookup"><span data-stu-id="f5b1b-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Program.cs** accordingly.</span></span>

<span data-ttu-id="f5b1b-105">Секреты приложений должны храниться в безопасном хранилище, например [в Хранилище ключей Azure.](https://docs.microsoft.com/azure/key-vault/general/overview)</span><span class="sxs-lookup"><span data-stu-id="f5b1b-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="f5b1b-106">Обновление параметра CORS для функции Azure</span><span class="sxs-lookup"><span data-stu-id="f5b1b-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="f5b1b-107">В этом примере мыlocal.settings.js **CORS,** чтобы разрешить тестовому приложению вызывать функцию.</span><span class="sxs-lookup"><span data-stu-id="f5b1b-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="f5b1b-108">Вам потребуется настроить опубликованную функцию, чтобы разрешить любые spa-приложения, которые будут ее вызывать.</span><span class="sxs-lookup"><span data-stu-id="f5b1b-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="f5b1b-109">Обновление регистраций приложений</span><span class="sxs-lookup"><span data-stu-id="f5b1b-109">Update app registrations</span></span>

<span data-ttu-id="f5b1b-110">Свойство в манифесте для регистрации Graph Azure Function должно быть обновлено с помощью ID приложений любых приложений, которые будут вызывать `knownClientApplications` функцию Azure. </span><span class="sxs-lookup"><span data-stu-id="f5b1b-110">The `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="f5b1b-111">Воссоздание существующих подписок</span><span class="sxs-lookup"><span data-stu-id="f5b1b-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="f5b1b-112">Любые подписки, созданные с помощью URL-адреса веб-страницы на локальном компьютере или ngrok, должны быть воссозданы с использованием производственного URL-адреса `Notify` функции Azure.</span><span class="sxs-lookup"><span data-stu-id="f5b1b-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
