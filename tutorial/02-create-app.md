---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655235"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве вы создадим простую функцию Azure, которая реализует функции триггера HTTP, которые вызывают Microsoft Graph. Эти функции охватывают следующие сценарии:

- Реализует API для доступа к почтовому [](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) ящику пользователя с помощью проверки подлинности потока "от имени".
- Реализует API для подписки на уведомления в почтовом ящике пользователя и [](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) отписаться от них, используя проверку подлинности потока предоставления учетных данных клиента.
- Реализует веб-hook для [](https://docs.microsoft.com/graph/webhooks) получения уведомлений об изменениях из Microsoft Graph и доступа к данным с помощью потока предоставления учетных данных клиента.

Вы также создадим простое одно page application JavaScript (SPA) для вызова API, реализованных в функции Azure.

## <a name="create-azure-functions-project"></a>Создание проекта функций Azure

1. Откройте интерфейс командной строки (CLI) в каталоге, где нужно создать проект. Выполните следующую команду.

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. Измените текущий каталог в CLI на **каталог GraphTutorial** и запустите следующие команды, чтобы создать три функции в проекте.

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. Откройте **local.settings.jsи** добавьте в файл следующее, чтобы разрешить CORS с `http://localhost:8080` URL-адреса тестового приложения.

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. Чтобы запустить проект локально, запустите следующую команду:

    ```Shell
    func start
    ```

1. Если все работает, вы увидите следующие выходные данные:

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. Убедитесь, что функции работают правильно, открыв браузер и открыв URL-адреса функций, показанные в выходных данных. В браузере должно отобраться следующее `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` сообщение:

## <a name="create-single-page-application"></a>Создание одно page application

1. Откройте CLI в каталоге, в котором вы хотите создать проект. Создайте каталог с **именем TestClient** для удержания файлов HTML и JavaScript.

1. Создайте файл с **именемindex.html** в **каталоге TestClient** и добавьте следующий код.

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    Это определяет базовый макет приложения, включая панели навигации. Он также добавляет следующее:

    - [Bootstrap и](https://getbootstrap.com/) поддерживаемый JavaScript
    - [FontAwesome](https://fontawesome.com/)
    - [Библиотека проверки подлинности Майкрософт для JavaScript (MSAL.js) 2.0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > Страница включает в себя факс, ( `<link rel="shortcut icon" href="g-raph.png">` ). Эту строку можно удалить или скачатьg-raph.png **с** [GitHub.](https://github.com/microsoftgraph/g-raph)

1. Создайте файл **style.css** в **каталоге TestClient** и добавьте следующий код.

    :::code language="css" source="../demo/TestClient/style.css":::

1. Создайте файл с **именемui.js** в **каталоге TestClient** и добавьте следующий код.

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    Этот код использует JavaScript для отображения текущей страницы на основе выбранного представления.

### <a name="test-the-single-page-application"></a>Тестирование однобукв 2-странигового приложения

> [!NOTE]
> В этом разделе содержатся инструкции по использованию [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) для запуска простого тестового HTTP-сервера на компьютере разработки. Использовать это средство не требуется. Можно использовать любой тестовый сервер, который вы предпочитаете обслуживать **каталог TestClient.**

1. Чтобы установить **dotnet-serve,** запустите следующую команду в CLI.

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. Измените текущий каталог в CLI на **каталог TestClient** и запустите следующую команду, чтобы запустить HTTP-сервер.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. Откройте браузер и перейдите по адресу `http://localhost:8080`. Страница должна отрисовки, но в настоящее время ни одна из кнопок не работает.

## <a name="add-nuget-packages"></a>Добавление пакетов NuGet

Прежде чем двигаться дальше, установите некоторые дополнительные пакеты NuGet, которые вы будете использовать позже.

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) для вреализации зависимостей в проекте функций Azure.
- [Microsoft.Extensions.Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) для чтения конфигурации приложения из секретного магазина [разработки .NET.](https://docs.microsoft.com/aspnet/core/security/app-secrets)
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) для звонков в Microsoft Graph.
- [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) для проверки подлинности и управления маркерами.
- [Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) для ирисовки конфигурации OpenID для проверки маркеров.
- [System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) для проверки маркеров, отправленных в веб-API.

1. Измените текущий каталог в CLI на **каталог GraphTutorial** и запустите следующие команды.

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
