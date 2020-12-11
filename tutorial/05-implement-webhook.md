---
ms.openlocfilehash: eb227079656e2a57550511c3abfacb49935fe46a
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655242"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы завершите реализацию функций Azure и обновим тестового приложения, чтобы подписаться и отписаться от изменений в почтовом ящике `SetSubscription` `Notify` пользователя.

- Функция будет действовать в качестве API, позволяя тестовой приложению создавать или удалять подписку на изменения в почтовом ящике `SetSubscription` пользователя. [](https://docs.microsoft.com/graph/webhooks)
- Функция `Notify` будет выступать в качестве веб-hook, который получает уведомления об изменениях, созданные подпиской.

Обе функции будут использовать [поток предоставления](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) учетных данных клиента для получения маркера только приложения для вызова Microsoft Graph. Так как администратор предоставил согласие администратора на требуемую область разрешений, для получения маркера не потребуется никаких взаимодействий с пользователем.

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>Добавление проверки подлинности учетных данных клиента в проект функций Azure

В этом разделе вы реализуете поток учетных данных клиента в проекте функций Azure, чтобы получить маркер доступа, совместимый с Microsoft Graph.

1. Откройте CLI в каталоге, который содержит **GraphTutorial.csproj.**

1. Добавьте свой ИД и секрет приложения веб-ook в хранилище секретов с помощью следующих команд. Замените `YOUR_WEBHOOK_APP_ID_HERE` его на ИД приложения для веб-hook **функции Graph Azure.** Замените `YOUR_WEBHOOK_APP_SECRET_HERE` секрет приложения, созданный на портале Azure для веб-приемка функции **Graph Azure.**

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>Создание поставщика проверки подлинности учетных данных клиента

1. Создайте файл **./GraphTutorial/Authentication** с именем ClientCredentialsAuthProvider.cs и **добавьте** следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

Удумайте, что делает код **в** ClientCredentialsAuthProvider.cs.

- В конструкторе он инициализирует **ConfidentialClientApplication** из `Microsoft.Identity.Client` пакета. Он использует и функции, чтобы ограничить аудиторию входа только указанной `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` `.WithTenantId(tenantId)` организацией Microsoft 365.
- В `GetAccessToken` функции она вызывает `AcquireTokenForClient` получение маркера для приложения. Поток маркера учетных данных клиента всегда не является интерактивным.
- Он реализует интерфейс, позволяя передавать этот класс в конструкторе для проверки подлинности исходяющих `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` запросов.

## <a name="update-graphclientservice"></a>Обновление GraphClientService

1. Откройте **GraphClientService.cs** и добавьте в класс следующее свойство.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. Замените имеющуюся функцию `GetAppGraphClient` указанным ниже кодом.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>Реализация функции Notify

В этом разделе вы реализуем функцию, которая будет использоваться в качестве `Notify` URL-адреса уведомлений об изменениях.

1. Создайте каталог в **каталоге GraphTutorials** с именем **Models.**

1. Создайте новый файл в **каталоге Models** с **именем ResourceData.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. Создайте новый файл в **каталоге Models** с **именем ChangeNotification.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotification.cs" id="ChangeNotificationSnippet":::

1. Создайте новый файл в **каталоге Models** с **именем NotificationList.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. Откройте **./GraphTutorial/Notify.cs** и замените все его содержимое следующим:

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

Удумайте, что делает код **в** Notify.cs.

- Функция `Run` проверяет наличие параметра `validationToken` запроса. Если этот параметр присутствует, он обрабатывает [](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)запрос как запрос проверки и отвечает соответствующим образом.
- Если запрос не является запросом на проверку, полезной нагрузки JSON десериализации в `NotificationList` .
- Каждое уведомление в списке проверяется на наличие ожидаемого значения состояния клиента и обрабатывается.
- Сообщение, которое вызвало уведомление, извлекается с помощью Microsoft Graph.

## <a name="implement-setsubscription-function"></a>Реализация функции SetSubscription

В этом разделе вы реализуете функцию SetSubscription. Эта функция будет выступать в качестве API, который будет вызван тестовой приложением для создания или удаления подписки в почтовом ящике пользователя.

1. Создайте новый файл в **каталоге Models** с **именем SetSubscriptionPayload.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. Откройте **./GraphTutorial/SetSubscription.cs** и замените все его содержимое следующим:

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

Удумайте, что делает код **в** SetSubscription.cs.

- Функция считывает данные JSON, отправленные в запросе POST, чтобы определить тип запроса (подписку или отписаться), ИД пользователя, на который необходимо подписаться, и ИД подписки, для которой необходимо `Run` отписаться.
- Если запрос является запросом на подписку, он использует Microsoft Graph SDK для создания новой подписки в почтовом ящике указанного пользователя. Подписка будет уведомлять о том, что сообщения созданы или обновлены. Новая подписка возвращается в полезной нагрузке ответа JSON.
- Если запрос является запросом на отписку, он использует microsoft Graph SDK для удаления указанной подписки.

## <a name="call-setsubscription-from-the-test-app"></a>Вызов SetSubscription из тестового приложения

В этом разделе вы реализуйте функции для создания и удаления подписок в тестовом приложении.

1. Откройте **./TestClient/azurefunctions.js** добавьте следующую функцию.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    Этот код вызывает функцию Azure для подписки и добавляет новую подписку в массив `SetSubscription` подписок в сеансе.

1. Добавьте вazurefunctions.js **следующую функцию.**

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    Этот код вызывает функцию Azure для удаления подписки из массива подписок в `SetSubscription` сеансе.

1. Если у вас нет запущенного ngrok, запустите ngrok ( ) и `ngrok http 7071` скопируйте URL-адрес переадрит HTTPS.

1. Добавьте URL-адрес ngrok в хранилище секретов пользователя, вынеся следующую команду.

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > Если вы перезапустите ngrok, вам потребуется повторить эту команду, чтобы обновить URL-адрес ngrok.

1. Измените текущий каталог в CLI на **./GraphTutorial** и запустите следующую команду, чтобы запустить функцию Azure локально.

    ```Shell
    func start
    ```

1. Обновите SPA и выберите элемент **nav subscriptions.** Введите ИД пользователя в организации Microsoft 365 с почтовым ящиком Exchange Online. Это может быть пользователь `id` (из Microsoft Graph) или пользователь `userPrincipalName` . Нажмите **кнопку "Подписаться"**.

1. Страница обновляется с отображением новой подписки в таблице.

1. Отправьте пользователю сообщение электронной почты. Через некоторое время необходимо `Notify` будет вызвана функция. Вы можете проверить это в веб-интерфейсе ngrok ( ) или в выходных данных отлаки `http://localhost:4040` проекта функции Azure.

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. В тестовом приложении **щелкните "Удалить"** в строке таблицы для подписки. Страница обновляется, и подписка больше не находится в таблице.
