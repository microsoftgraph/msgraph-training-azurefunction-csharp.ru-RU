---
ms.openlocfilehash: 914957809f268ad29f8cfc44c21dd9fb63699322
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445964"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве будет создаваться простая функция Azure, которая реализует триггерные функции HTTP, которые вызывают Microsoft Graph. Эти функции будут охватывать следующие сценарии:

- Реализует API для доступа к почтовому [](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) ящику пользователя с помощью проверки подлинности потока от имени потока.
- Реализует API для подписки и отписки уведомлений в почтовом ящике пользователя с помощью проверки подлинности потока предоставления клиентских [учетных](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) данных.
- Реализует веб-сайт для [](https://docs.microsoft.com/graph/webhooks) получения уведомлений об изменениях от Корпорации Майкрософт Graph доступа к данным с помощью потока предоставления клиентских учетных данных.

Вы также создайте простое одно страница JavaScript-приложение (SPA), чтобы вызвать API, реализованные в Функции Azure.

## <a name="create-azure-functions-project"></a>Создание проекта Azure Functions

1. Откройте интерфейс командной строки (CLI) в каталоге, в котором необходимо создать проект. Выполните следующую команду.

    ```Shell
    func init GraphTutorial --worker-runtime dotnetisolated
    ```

1. Измените текущий каталог в CLI на **каталог GraphTutorial** и запустите следующие команды, чтобы создать три функции в проекте.

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger"
    func new --name SetSubscription --template "HTTP trigger"
    func new --name Notify --template "HTTP trigger"
    ```

1. Откройте **local.settings.jsи** добавьте в файл следующее, чтобы разрешить CORS из URL-адреса `http://localhost:8080` тестового приложения.

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. Запустите следующую команду для локального запуска проекта.

    ```Shell
    func start
    ```

1. Если все работает, вы увидите следующий вывод:

    ```Shell
    Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. Убедитесь, что функции работают правильно, открыв браузер и просматривая URL-адреса функций, показанные на выходе. Вы должны увидеть следующее сообщение в браузере: `Welcome to Azure Functions!` .

## <a name="create-single-page-application"></a>Создание одно-страницного приложения

1. Откройте CLI в каталоге, в котором необходимо создать проект. Создайте каталог с именем **TestClient** для удержания файлов HTML и JavaScript.

1. Создайте новый файл **сindex.html** в **каталоге TestClient** и добавьте следующий код.

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    Это определяет основные макеты приложения, в том числе панели навигации. Кроме того, добавляется следующее:

    - [Bootstrap](https://getbootstrap.com/) и поддерживаемый JavaScript
    - [FontAwesome](https://fontawesome.com/)
    - [Библиотека проверки подлинности Майкрософт для JavaScript (MSAL.js) 2.0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > Страница включает в себя favicon, ( `<link rel="shortcut icon" href="g-raph.png">` ). Эту строку можно удалить или скачатьg-raph.png **из** [GitHub.](https://github.com/microsoftgraph/g-raph)

1. Создайте новый файл **style.css** в **каталоге TestClient** и добавьте следующий код.

    :::code language="css" source="../demo/TestClient/style.css":::

1. Создайте новый файл **сui.js** в **каталоге TestClient** и добавьте следующий код.

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    Этот код использует JavaScript для отображения текущей страницы на основе выбранного представления.

### <a name="test-the-single-page-application"></a>Тестирование одно-страницного приложения

> [!NOTE]
> В этом разделе содержатся инструкции по использованию [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) для запуска простого сервера HTTP-тестирования на компьютере разработки. Использование этого конкретного средства не требуется. Вы можете использовать любой сервер тестирования, который вы предпочитаете обслуживать **каталог TestClient.**

1. Запустите следующую команду в CLI для установки **dotnet-serve.**

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. Измените текущий каталог в CLI на **каталог TestClient** и запустите следующую команду, чтобы запустить http-сервер.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. Откройте браузер и перейдите по адресу `http://localhost:8080`. Страница должна отрисовка, но ни одна из кнопок в настоящее время не работает.

## <a name="add-nuget-packages"></a>Добавление пакетов NuGet

Прежде чем двигаться дальше, установите дополнительные NuGet пакеты, которые вы будете использовать позже.

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) для впрыскивания зависимостей в проекте Azure Functions.
- [Microsoft.Extensions.Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) для чтения конфигурации приложений из секретного магазина [разработки .NET.](https://docs.microsoft.com/aspnet/core/security/app-secrets)
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) для вызовов Microsoft Graph.
- [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) для проверки подлинности и управления маркерами.
- [Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) для восстановления конфигурации OpenID для проверки маркеров.
- [System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) для проверки маркеров, отправленных в веб-API.

1. Измените текущий каталог в CLI на **каталог GraphTutorial** и запустите следующие команды.

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.1.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 5.0.0
    dotnet add package Microsoft.Graph --version 4.0.0
    dotnet add package Microsoft.Identity.Client --version 4.34.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.11.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.11.1
    ```
