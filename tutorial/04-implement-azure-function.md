---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445957"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы завершите реализацию функции Azure и `GetMyNewestMessage` обновим тестовый клиент, чтобы вызвать эту функцию.

Функция Azure использует поток от [имени.](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) Основной порядок событий в этом потоке:

- Тестовая программа использует интерактивный поток auth, чтобы позволить пользователю войти и предоставить согласие. Он возвращает маркер, который является областью для функции Azure. Маркер не **содержит никаких** областей Graph Microsoft.
- Тестовая приложение вызывает функцию Azure, отправляя маркер доступа в `Authorization` заглавном загонах.
- Функция Azure проверяет маркер, а затем обменивает его на второй маркер доступа, содержащий области Microsoft Graph.
- Функция Azure вызывает microsoft Graph от имени пользователя с помощью второго маркера доступа.

> [!IMPORTANT]
> Чтобы не хранить код приложения и секрет в источнике, для хранения этих значений используется [диспетчер секрета .NET.](https://docs.microsoft.com/aspnet/core/security/app-secrets) Секретный менеджер предназначен только для целей разработки, для хранения секретов в производственных приложениях должен использовать доверенный секретный менеджер.

## <a name="add-authentication-to-the-single-page-application"></a>Добавление проверки подлинности в приложение для одной страницы

Начните с добавления проверки подлинности в SPA. Это позволит приложению получить доступ к маркеру доступа для вызова функции Azure. Так как это SPA, он будет использовать поток кода [авторизации с помощью PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).

1. Создайте новый файл в **каталоге TestClient** с именемconfig.jsи **добавьте** следующий код.

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    Замените с помощью ID приложения, созданного на портале `YOUR_TEST_APP_APP_ID_HERE` Azure для Graph тестирования **функций Azure.** `YOUR_TENANT_ID_HERE`Замените **значение ID Directory (tenant),** скопированные с портала Azure. Замените `YOUR_AZURE_FUNCTION_APP_ID_HERE` iD приложения для **функции Graph Azure.**

    > [!IMPORTANT]
    > Если вы используете источник управления, например git, сейчас самое время исключить файлconfig.jsиз **источника** управления, чтобы не допустить случайной утечки кодов приложений и ID клиента.

1. Создайте новый файл в **каталоге TestClient** с именемauth.jsи **добавьте** следующий код.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    Рассмотрим, что делает этот код.

    - Инициализирует использование значений, хранимых `PublicClientApplication` **вconfig.js.**
    - Он использует для регистрации пользователя, используя область разрешений `loginPopup` для функции Azure.
    - В сеансе сохраняется имя пользователя.

    > [!IMPORTANT]
    > Так как приложение использует, возможно, потребуется изменить всплывающее блокатор браузера, чтобы разрешить всплывающие `loginPopup` окна от `http://localhost:8080` .

1. Обновление страницы и вход. Страница должна обновиться с именем пользователя, указав, что вход был успешным.

## <a name="add-authentication-to-the-azure-function"></a>Добавление проверки подлинности в функцию Azure

В этом разделе будет реализован поток от имени в функции Azure, чтобы получить маркер доступа, совместимый с `GetMyNewestMessage` Microsoft Graph.

1. Инициализировать секретный магазин разработки .NET, открыв CLI в каталоге, содержаном **GraphTutorial.csproj** и запуская следующую команду.

    ```Shell
    dotnet user-secrets init
    ```

1. Добавьте свой ID приложения, секретный и клиентский ID в секретный магазин с помощью следующих команд. Замените `YOUR_API_FUNCTION_APP_ID_HERE` iD приложения для **функции Graph Azure.** Замените секрет приложения, созданный на портале Azure для `YOUR_API_FUNCTION_APP_SECRET_HERE` **Graph Azure Function.** `YOUR_TENANT_ID_HERE`Замените **значение ID Directory (tenant),** скопированные с портала Azure.

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>Обработка маркера входящих носителей

В этом разделе будет реализован класс для проверки и обработки маркера-носитера, отправленного из SPA в функцию Azure.

1. Создайте новый каталог в **каталоге GraphTutorial** с именем **Authentication.**

1. Создайте новый файл с именем **TokenValidationResult.cs** в **папке ./GraphTutorial/Authentication** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. Создайте новый файл с именем **TokenValidation.cs** в **папке ./GraphTutorial/Authentication** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

Рассмотрим, что делает этот код.

- Оно гарантирует, что в загонах имеется маркер `Authorization` носитела.
- Он проверяет подпись и эмитент из опубликованной конфигурации OpenID в Azure.
- В нем проверяется, что аудитория `aud` (утверждение) соответствует ID приложения Azure Function.
- Он разбирает маркер и создает ID учетной записи MSAL, который будет необходим для использования кэшинга маркеров.

### <a name="create-an-on-behalf-of-authentication-provider"></a>Создание поставщика проверки подлинности от имени

1. Создайте новый файл в **каталоге** проверки подлинности с именем **OnBehalfOfAuthProvider.cs** и добавьте в этот файл следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

Подумайте о том, что делает код **в OnBehalfOfAuthProvider.cs.**

- В функции сначала пытается получить маркер пользователя `GetAccessToken` из кэша маркера с помощью `AcquireTokenSilent` . Если это не удается, он использует маркер-носителер, отправленный тест-приложением в функцию Azure для создания утверждения пользователя. Затем он использует это утверждение пользователя для получения Graph совместимого маркера с помощью `AcquireTokenOnBehalfOf` .
- Он реализует интерфейс, позволяя передавать этот класс в конструкторе исходяющих запросов для проверки `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` подлинности.

### <a name="implement-a-graph-client-service"></a>Реализация службы Graph клиента

В этом разделе реализована служба, которую можно зарегистрировать для впрыска [зависимостей.](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) Служба будет использоваться для получения проверки подлинности Graph клиента.

1. Создание нового каталога в **каталоге GraphTutorial** с именем **Services.**

1. Создайте новый файл в **каталоге Служб** с именем **IGraphClientService.cs** и добавьте в этот файл следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. Создайте новый файл в **каталоге Служб** с именем **GraphClientService.cs** и добавьте в этот файл следующий код.

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. Добавьте в класс следующие `GraphClientService` свойства.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. Добавьте в класс следующие `GraphClientService` функции.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. Добавьте реализацию задатки для `GetAppGraphClient` функции. Это будет реализовано в последующих разделах.

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    Функция принимает результаты проверки маркеров и создает проверку подлинности `GetUserGraphClient` `GraphServiceClient` для пользователя.

1. Откройте **./GraphTutorial/Program.cs** и замените его содержимое следующим.

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    Этот код добавит секреты пользователей [](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) в конфигурацию и встроит инъекцию зависимостей в функции Azure, обнажая `GraphClientService` службу.

### <a name="implement-getmynewestmessage-function"></a>Реализация функции GetMyNewestMessage

1. Откройте **./GraphTutorial/GetMyNewestMessage.cs** и замените все содержимое на следующее.

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>Просмотрите код в GetMyNewestMessage.cs

Подумайте, что делает код **в GetMyNewestMessage.cs.**

- В конструкторе он сохраняет объекты, переданные через впрыски `IConfiguration` `IGraphClientService` зависимостей.
- В `Run` функции она делает следующее:
  - Проверяет необходимые значения конфигурации, присутствующие в `IConfiguration` объекте.
  - Проверяет маркер носителей и возвращает код `401` состояния, если маркер является недействительным.
  - Получает Graph клиента от `GraphClientService` пользователя, который сделал этот запрос.
  - Использует SDK Graph Microsoft для получения самого нового сообщения из почтового ящика пользователя и возвращает его в качестве тела JSON в ответе.

## <a name="call-the-azure-function-from-the-test-app"></a>Вызов функции Azure из тестового приложения

1. Откройте **auth.js** и добавьте следующую функцию, чтобы получить маркер доступа.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    Рассмотрим, что делает этот код.

    - Сначала он пытается получить маркер доступа без взаимодействия с пользователем. Так как пользователь уже должен быть подписан, msAL должен иметь маркеры для пользователя в кэше.
    - Если сбой с ошибкой, которая указывает на то, что пользователю необходимо взаимодействовать, он пытается получить маркер интерактивно.

    > [!TIP]
    > Вы можете разопросить маркер доступа и подтвердить, что претензия — это ID приложения для функции Azure и что в утверждении содержится область разрешений Azure Function, а не Microsoft [https://jwt.ms](https://jwt.ms) `aud` `scp` Graph.

1. Создайте новый файл в **каталоге TestClient** с именемazurefunctions.jsи **добавьте** следующий код.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. Измените текущий каталог в CLI на **каталог ./GraphTutorial** и запустите следующую команду, чтобы запустить функцию Azure локально.

    ```Shell
    func start
    ```

1. Если еще не обслуживается SPA, откройте второе окно CLI и измените текущий каталог на **каталог ./TestClient.** Запустите следующую команду для запуска тестового приложения.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. Откройте браузер и перейдите по адресу `http://localhost:8080`. Впишите и выберите элемент **навигации "Последнее** сообщение". Приложение отображает сведения о самом новом сообщении в почтовом ящике пользователя.
