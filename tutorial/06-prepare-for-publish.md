---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445950"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы узнаете, какие изменения необходимы примеру Azure Function для подготовки к публикации в [приложении Azure Functions.](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)

## <a name="update-code"></a>Обновление кода

Конфигурация читается из секретного магазина пользователя, который применяется только к вашей машине разработки. Перед публикацией в Azure необходимо изменить место хранения конфигурации и соответствующим образом обновить код **в Program.cs.**

Секреты приложений должны храниться в безопасном хранилище, например [в Хранилище ключей Azure.](https://docs.microsoft.com/azure/key-vault/general/overview)

## <a name="update-cors-setting-for-azure-function"></a>Обновление параметра CORS для функции Azure

В этом примере мыlocal.settings.js **CORS,** чтобы разрешить тестовому приложению вызывать функцию. Вам потребуется настроить опубликованную функцию, чтобы разрешить любые spa-приложения, которые будут ее вызывать.

## <a name="update-app-registrations"></a>Обновление регистраций приложений

Свойство в манифесте для регистрации Graph Azure Function должно быть обновлено с помощью ID приложений любых приложений, которые будут вызывать `knownClientApplications` функцию Azure. 

## <a name="recreate-existing-subscriptions"></a>Воссоздание существующих подписок

Любые подписки, созданные с помощью URL-адреса веб-страницы на локальном компьютере или ngrok, должны быть воссозданы с использованием производственного URL-адреса `Notify` функции Azure.
