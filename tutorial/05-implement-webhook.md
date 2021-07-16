---
ms.openlocfilehash: 181afe9acfe45ff619a50cf874669228b421b475
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445978"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы завершите реализацию функций Azure и обновим тестового приложения, чтобы подписаться и отписаться от изменений в почтовом ящике `SetSubscription` `Notify` пользователя.

- Функция будет выступать в качестве API, позволяя тестовой приложению создавать или удалять подписку на изменения в почтовом `SetSubscription` ящике пользователя. [](https://docs.microsoft.com/graph/webhooks)
- Функция будет выступать в качестве веб-сайта, который получает уведомления об `Notify` изменении, созданные подпиской.

Обе функции будут использовать поток клиентских учетных данных [для](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) получения маркера только для приложений для вызова Microsoft Graph. Так как администратор предоставил администратору согласие на необходимые области разрешений, для получения маркера не потребуется взаимодействие с пользователем.

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>Добавление проверки подлинности учетных данных клиентов в проект Azure Functions

В этом разделе вы реализуете поток клиентских учетных данных в проекте Azure Functions, чтобы получить маркер доступа, совместимый с Microsoft Graph.

1. Откройте CLI в каталоге, который содержит **GraphTutorial.csproj.**

1. Добавьте свой ИД приложения webhook и секрет в секретный магазин с помощью следующих команд. Замените `YOUR_WEBHOOK_APP_ID_HERE` iD приложения для **веб-Graph Azure Function.** Замените секрет приложения, созданный на портале Azure для `YOUR_WEBHOOK_APP_SECRET_HERE` **веб-Graph Azure Function.**

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>Создание поставщика проверки подлинности учетных данных клиента

1. Создайте новый файл в **каталоге ./GraphTutorial/Authentication** с именем **ClientCredentialsAuthProvider.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

Удумайте, что делает код **в ClientCredentialsAuthProvider.cs.**

- В конструкторе инициализируется **конфиденциальнаяclientApplication** из `Microsoft.Identity.Client` пакета. Он использует `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` функции и функции, чтобы ограничить аудиторию входа только указанной Microsoft 365 `.WithTenantId(tenantId)` организации.
- В `GetAccessToken` функции она вызывает `AcquireTokenForClient` получение маркера для приложения. Поток маркеров учетных данных клиента всегда не является интерактивным.
- Он реализует интерфейс, позволяя передавать этот класс в конструкторе исходяющих запросов для проверки `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` подлинности.

## <a name="update-graphclientservice"></a>Обновление GraphClientService

1. Откройте **GraphClientService.cs** и добавьте следующее свойство в класс.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. Замените имеющуюся функцию `GetAppGraphClient` указанным ниже кодом.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>Реализация функции Notify

В этом разделе будет реализована функция, которая будет использоваться в качестве URL-адреса уведомлений об `Notify` изменениях.

1. Создание нового каталога в **каталоге GraphTutorials** с именем **Models**.

1. Создайте новый файл в **каталоге Models** с именем **ResourceData.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. Создайте новый файл в **каталоге Models** с именем **ChangeNotificationPayload.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotificationPayload.cs" id="ChangeNotificationSnippet":::

1. Создайте новый файл в **каталоге Моделей** с именем **NotificationList.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. Откройте **./GraphTutorial/Notify.cs** и замените все содержимое на следующее.

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

Удумайте, что делает код **в Notify.cs.**

- Функция `Run` проверяет наличие параметра `validationToken` запроса. Если этот параметр присутствует, он обрабатывает запрос как запрос проверки [и](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)отвечает соответствующим образом.
- Если запрос не является запросом на проверку, полезной нагрузкой JSON является deserialized в `ChangeNotificationCollection` .
- Каждое уведомление в списке проверяется на ожидаемое значение состояния клиента и обрабатывается.
- Сообщение, которое вызвало уведомление, извлекалось с помощью Microsoft Graph.

## <a name="implement-setsubscription-function"></a>Реализация функции SetSubscription

В этом разделе будет реализована функция SetSubscription. Эта функция будет выступать в качестве API, который вызван тест-приложением для создания или удаления подписки в почтовом ящике пользователя.

1. Создайте новый файл в **каталоге Models** с именем **SetSubscriptionPayload.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. Откройте **./GraphTutorial/SetSubscription.cs** и замените все содержимое следующим образом.

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

Задумайтесь о том, что делает код **в SetSubscription.cs.**

- Функция считывает полезной нагрузки JSON, отправленной в запрос POST, чтобы определить тип запроса (подписаться или отписаться), пользовательский ID для подписки и подписку на подписку, чтобы `Run` отписаться.
- Если запрос является запросом на подписку, он использует SDK microsoft Graph для создания новой подписки в почтовом ящике указанного пользователя. Подписка будет уведомлять о том, когда сообщения создаются или обновляются. Новая подписка возвращается в полезной нагрузке JSON ответа.
- Если запрос является отписаным, для удаления указанной подписки используется Graph microsoft Graph SDK.

## <a name="call-setsubscription-from-the-test-app"></a>Вызов SetSubscription из тестового приложения

В этом разделе будут реализованы функции создания и удаления подписок в тестовом приложении.

1. Откройте **./TestClient/azurefunctions.js** и добавьте следующую функцию.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    Этот код вызывает функцию Azure для подписки и добавляет новую подписку в массив `SetSubscription` подписок в сеансе.

1. Добавьте следующую функцию **вazurefunctions.js.**

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    Этот код вызывает функцию Azure для отписки и удаляет подписку из массива `SetSubscription` подписок в сеансе.

1. Если у вас нет запуска ngrok, запустите ngrok () и `ngrok http 7071` скопируйте URL-адрес переададки HTTPS.

1. Добавьте URL-адрес ngrok в хранилище секретов пользователей, выстроив следующую команду.

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > Если вы перезапустите ngrok, вам потребуется повторить эту команду, чтобы обновить URL-адрес ngrok.

1. Измените текущий каталог в CLI на **каталог ./GraphTutorial** и запустите следующую команду, чтобы запустить функцию Azure локально.

    ```Shell
    func start
    ```

1. Обновите SPA и выберите **элемент Подписки** nav. Введите пользовательский ИД для пользователя в Microsoft 365 организации с Exchange Online почтовым ящиком. Это может быть либо пользователь `id` (от Microsoft Graph), либо пользователь `userPrincipalName` . Нажмите **кнопку Подписаться**.

1. Страница обновляет отображение новой подписки в таблице.

1. Отправка электронной почты пользователю. Через некоторое время функция `Notify` должна быть вызвана. Проверить это можно в веб-интерфейсе ngrok () или в отлаговке проекта `http://localhost:4040` Azure Function.

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. В тестовом приложении нажмите **кнопку Удалить** в строке таблицы для подписки. Страница обновляется, а подписка больше не находится в таблице.
