---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445957"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7196b-101">В этом упражнении вы завершите реализацию функции Azure и `GetMyNewestMessage` обновим тестовый клиент, чтобы вызвать эту функцию.</span><span class="sxs-lookup"><span data-stu-id="7196b-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="7196b-102">Функция Azure использует поток от [имени.](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)</span><span class="sxs-lookup"><span data-stu-id="7196b-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="7196b-103">Основной порядок событий в этом потоке:</span><span class="sxs-lookup"><span data-stu-id="7196b-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="7196b-104">Тестовая программа использует интерактивный поток auth, чтобы позволить пользователю войти и предоставить согласие.</span><span class="sxs-lookup"><span data-stu-id="7196b-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="7196b-105">Он возвращает маркер, который является областью для функции Azure.</span><span class="sxs-lookup"><span data-stu-id="7196b-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="7196b-106">Маркер не **содержит никаких** областей Graph Microsoft.</span><span class="sxs-lookup"><span data-stu-id="7196b-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="7196b-107">Тестовая приложение вызывает функцию Azure, отправляя маркер доступа в `Authorization` заглавном загонах.</span><span class="sxs-lookup"><span data-stu-id="7196b-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="7196b-108">Функция Azure проверяет маркер, а затем обменивает его на второй маркер доступа, содержащий области Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="7196b-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="7196b-109">Функция Azure вызывает microsoft Graph от имени пользователя с помощью второго маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="7196b-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="7196b-110">Чтобы не хранить код приложения и секрет в источнике, для хранения этих значений используется [диспетчер секрета .NET.](https://docs.microsoft.com/aspnet/core/security/app-secrets)</span><span class="sxs-lookup"><span data-stu-id="7196b-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="7196b-111">Секретный менеджер предназначен только для целей разработки, для хранения секретов в производственных приложениях должен использовать доверенный секретный менеджер.</span><span class="sxs-lookup"><span data-stu-id="7196b-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="7196b-112">Добавление проверки подлинности в приложение для одной страницы</span><span class="sxs-lookup"><span data-stu-id="7196b-112">Add authentication to the single page application</span></span>

<span data-ttu-id="7196b-113">Начните с добавления проверки подлинности в SPA.</span><span class="sxs-lookup"><span data-stu-id="7196b-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="7196b-114">Это позволит приложению получить доступ к маркеру доступа для вызова функции Azure.</span><span class="sxs-lookup"><span data-stu-id="7196b-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="7196b-115">Так как это SPA, он будет использовать поток кода [авторизации с помощью PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span><span class="sxs-lookup"><span data-stu-id="7196b-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="7196b-116">Создайте новый файл в **каталоге TestClient** с именемconfig.jsи **добавьте** следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="7196b-117">Замените с помощью ID приложения, созданного на портале `YOUR_TEST_APP_APP_ID_HERE` Azure для Graph тестирования **функций Azure.**</span><span class="sxs-lookup"><span data-stu-id="7196b-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="7196b-118">`YOUR_TENANT_ID_HERE`Замените **значение ID Directory (tenant),** скопированные с портала Azure.</span><span class="sxs-lookup"><span data-stu-id="7196b-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="7196b-119">Замените `YOUR_AZURE_FUNCTION_APP_ID_HERE` iD приложения для **функции Graph Azure.**</span><span class="sxs-lookup"><span data-stu-id="7196b-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="7196b-120">Если вы используете источник управления, например git, сейчас самое время исключить файлconfig.jsиз **источника** управления, чтобы не допустить случайной утечки кодов приложений и ID клиента.</span><span class="sxs-lookup"><span data-stu-id="7196b-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="7196b-121">Создайте новый файл в **каталоге TestClient** с именемauth.jsи **добавьте** следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="7196b-122">Рассмотрим, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="7196b-122">Consider what this code does.</span></span>

    - <span data-ttu-id="7196b-123">Инициализирует использование значений, хранимых `PublicClientApplication` **вconfig.js.**</span><span class="sxs-lookup"><span data-stu-id="7196b-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="7196b-124">Он использует для регистрации пользователя, используя область разрешений `loginPopup` для функции Azure.</span><span class="sxs-lookup"><span data-stu-id="7196b-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="7196b-125">В сеансе сохраняется имя пользователя.</span><span class="sxs-lookup"><span data-stu-id="7196b-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="7196b-126">Так как приложение использует, возможно, потребуется изменить всплывающее блокатор браузера, чтобы разрешить всплывающие `loginPopup` окна от `http://localhost:8080` .</span><span class="sxs-lookup"><span data-stu-id="7196b-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="7196b-127">Обновление страницы и вход.</span><span class="sxs-lookup"><span data-stu-id="7196b-127">Refresh the page and sign in.</span></span> <span data-ttu-id="7196b-128">Страница должна обновиться с именем пользователя, указав, что вход был успешным.</span><span class="sxs-lookup"><span data-stu-id="7196b-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="7196b-129">Добавление проверки подлинности в функцию Azure</span><span class="sxs-lookup"><span data-stu-id="7196b-129">Add authentication to the Azure Function</span></span>

<span data-ttu-id="7196b-130">В этом разделе будет реализован поток от имени в функции Azure, чтобы получить маркер доступа, совместимый с `GetMyNewestMessage` Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="7196b-130">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="7196b-131">Инициализировать секретный магазин разработки .NET, открыв CLI в каталоге, содержаном **GraphTutorial.csproj** и запуская следующую команду.</span><span class="sxs-lookup"><span data-stu-id="7196b-131">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="7196b-132">Добавьте свой ID приложения, секретный и клиентский ID в секретный магазин с помощью следующих команд.</span><span class="sxs-lookup"><span data-stu-id="7196b-132">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="7196b-133">Замените `YOUR_API_FUNCTION_APP_ID_HERE` iD приложения для **функции Graph Azure.**</span><span class="sxs-lookup"><span data-stu-id="7196b-133">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="7196b-134">Замените секрет приложения, созданный на портале Azure для `YOUR_API_FUNCTION_APP_SECRET_HERE` **Graph Azure Function.**</span><span class="sxs-lookup"><span data-stu-id="7196b-134">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="7196b-135">`YOUR_TENANT_ID_HERE`Замените **значение ID Directory (tenant),** скопированные с портала Azure.</span><span class="sxs-lookup"><span data-stu-id="7196b-135">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="7196b-136">Обработка маркера входящих носителей</span><span class="sxs-lookup"><span data-stu-id="7196b-136">Process the incoming bearer token</span></span>

<span data-ttu-id="7196b-137">В этом разделе будет реализован класс для проверки и обработки маркера-носитера, отправленного из SPA в функцию Azure.</span><span class="sxs-lookup"><span data-stu-id="7196b-137">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="7196b-138">Создайте новый каталог в **каталоге GraphTutorial** с именем **Authentication.**</span><span class="sxs-lookup"><span data-stu-id="7196b-138">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="7196b-139">Создайте новый файл с именем **TokenValidationResult.cs** в **папке ./GraphTutorial/Authentication** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-139">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="7196b-140">Создайте новый файл с именем **TokenValidation.cs** в **папке ./GraphTutorial/Authentication** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-140">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="7196b-141">Рассмотрим, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="7196b-141">Consider what this code does.</span></span>

- <span data-ttu-id="7196b-142">Оно гарантирует, что в загонах имеется маркер `Authorization` носитела.</span><span class="sxs-lookup"><span data-stu-id="7196b-142">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="7196b-143">Он проверяет подпись и эмитент из опубликованной конфигурации OpenID в Azure.</span><span class="sxs-lookup"><span data-stu-id="7196b-143">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="7196b-144">В нем проверяется, что аудитория `aud` (утверждение) соответствует ID приложения Azure Function.</span><span class="sxs-lookup"><span data-stu-id="7196b-144">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="7196b-145">Он разбирает маркер и создает ID учетной записи MSAL, который будет необходим для использования кэшинга маркеров.</span><span class="sxs-lookup"><span data-stu-id="7196b-145">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="7196b-146">Создание поставщика проверки подлинности от имени</span><span class="sxs-lookup"><span data-stu-id="7196b-146">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="7196b-147">Создайте новый файл в **каталоге** проверки подлинности с именем **OnBehalfOfAuthProvider.cs** и добавьте в этот файл следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-147">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="7196b-148">Подумайте о том, что делает код **в OnBehalfOfAuthProvider.cs.**</span><span class="sxs-lookup"><span data-stu-id="7196b-148">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="7196b-149">В функции сначала пытается получить маркер пользователя `GetAccessToken` из кэша маркера с помощью `AcquireTokenSilent` .</span><span class="sxs-lookup"><span data-stu-id="7196b-149">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="7196b-150">Если это не удается, он использует маркер-носителер, отправленный тест-приложением в функцию Azure для создания утверждения пользователя.</span><span class="sxs-lookup"><span data-stu-id="7196b-150">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="7196b-151">Затем он использует это утверждение пользователя для получения Graph совместимого маркера с помощью `AcquireTokenOnBehalfOf` .</span><span class="sxs-lookup"><span data-stu-id="7196b-151">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="7196b-152">Он реализует интерфейс, позволяя передавать этот класс в конструкторе исходяющих запросов для проверки `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` подлинности.</span><span class="sxs-lookup"><span data-stu-id="7196b-152">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="7196b-153">Реализация службы Graph клиента</span><span class="sxs-lookup"><span data-stu-id="7196b-153">Implement a Graph client service</span></span>

<span data-ttu-id="7196b-154">В этом разделе реализована служба, которую можно зарегистрировать для впрыска [зависимостей.](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)</span><span class="sxs-lookup"><span data-stu-id="7196b-154">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="7196b-155">Служба будет использоваться для получения проверки подлинности Graph клиента.</span><span class="sxs-lookup"><span data-stu-id="7196b-155">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="7196b-156">Создание нового каталога в **каталоге GraphTutorial** с именем **Services.**</span><span class="sxs-lookup"><span data-stu-id="7196b-156">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="7196b-157">Создайте новый файл в **каталоге Служб** с именем **IGraphClientService.cs** и добавьте в этот файл следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-157">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="7196b-158">Создайте новый файл в **каталоге Служб** с именем **GraphClientService.cs** и добавьте в этот файл следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-158">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

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

1. <span data-ttu-id="7196b-159">Добавьте в класс следующие `GraphClientService` свойства.</span><span class="sxs-lookup"><span data-stu-id="7196b-159">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="7196b-160">Добавьте в класс следующие `GraphClientService` функции.</span><span class="sxs-lookup"><span data-stu-id="7196b-160">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="7196b-161">Добавьте реализацию задатки для `GetAppGraphClient` функции.</span><span class="sxs-lookup"><span data-stu-id="7196b-161">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="7196b-162">Это будет реализовано в последующих разделах.</span><span class="sxs-lookup"><span data-stu-id="7196b-162">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="7196b-163">Функция принимает результаты проверки маркеров и создает проверку подлинности `GetUserGraphClient` `GraphServiceClient` для пользователя.</span><span class="sxs-lookup"><span data-stu-id="7196b-163">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="7196b-164">Откройте **./GraphTutorial/Program.cs** и замените его содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="7196b-164">Open **./GraphTutorial/Program.cs** and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    <span data-ttu-id="7196b-165">Этот код добавит секреты пользователей [](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) в конфигурацию и встроит инъекцию зависимостей в функции Azure, обнажая `GraphClientService` службу.</span><span class="sxs-lookup"><span data-stu-id="7196b-165">This code will add user secrets to the configuration, and enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="7196b-166">Реализация функции GetMyNewestMessage</span><span class="sxs-lookup"><span data-stu-id="7196b-166">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="7196b-167">Откройте **./GraphTutorial/GetMyNewestMessage.cs** и замените все содержимое на следующее.</span><span class="sxs-lookup"><span data-stu-id="7196b-167">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="7196b-168">Просмотрите код в GetMyNewestMessage.cs</span><span class="sxs-lookup"><span data-stu-id="7196b-168">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="7196b-169">Подумайте, что делает код **в GetMyNewestMessage.cs.**</span><span class="sxs-lookup"><span data-stu-id="7196b-169">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="7196b-170">В конструкторе он сохраняет объекты, переданные через впрыски `IConfiguration` `IGraphClientService` зависимостей.</span><span class="sxs-lookup"><span data-stu-id="7196b-170">In the constructor, it saves the `IConfiguration` and `IGraphClientService` objects passed in via dependency injection.</span></span>
- <span data-ttu-id="7196b-171">В `Run` функции она делает следующее:</span><span class="sxs-lookup"><span data-stu-id="7196b-171">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="7196b-172">Проверяет необходимые значения конфигурации, присутствующие в `IConfiguration` объекте.</span><span class="sxs-lookup"><span data-stu-id="7196b-172">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="7196b-173">Проверяет маркер носителей и возвращает код `401` состояния, если маркер является недействительным.</span><span class="sxs-lookup"><span data-stu-id="7196b-173">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="7196b-174">Получает Graph клиента от `GraphClientService` пользователя, который сделал этот запрос.</span><span class="sxs-lookup"><span data-stu-id="7196b-174">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="7196b-175">Использует SDK Graph Microsoft для получения самого нового сообщения из почтового ящика пользователя и возвращает его в качестве тела JSON в ответе.</span><span class="sxs-lookup"><span data-stu-id="7196b-175">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="7196b-176">Вызов функции Azure из тестового приложения</span><span class="sxs-lookup"><span data-stu-id="7196b-176">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="7196b-177">Откройте **auth.js** и добавьте следующую функцию, чтобы получить маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="7196b-177">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="7196b-178">Рассмотрим, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="7196b-178">Consider what this code does.</span></span>

    - <span data-ttu-id="7196b-179">Сначала он пытается получить маркер доступа без взаимодействия с пользователем.</span><span class="sxs-lookup"><span data-stu-id="7196b-179">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="7196b-180">Так как пользователь уже должен быть подписан, msAL должен иметь маркеры для пользователя в кэше.</span><span class="sxs-lookup"><span data-stu-id="7196b-180">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="7196b-181">Если сбой с ошибкой, которая указывает на то, что пользователю необходимо взаимодействовать, он пытается получить маркер интерактивно.</span><span class="sxs-lookup"><span data-stu-id="7196b-181">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

    > [!TIP]
    > <span data-ttu-id="7196b-182">Вы можете разопросить маркер доступа и подтвердить, что претензия — это ID приложения для функции Azure и что в утверждении содержится область разрешений Azure Function, а не Microsoft [https://jwt.ms](https://jwt.ms) `aud` `scp` Graph.</span><span class="sxs-lookup"><span data-stu-id="7196b-182">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

1. <span data-ttu-id="7196b-183">Создайте новый файл в **каталоге TestClient** с именемazurefunctions.jsи **добавьте** следующий код.</span><span class="sxs-lookup"><span data-stu-id="7196b-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="7196b-184">Измените текущий каталог в CLI на **каталог ./GraphTutorial** и запустите следующую команду, чтобы запустить функцию Azure локально.</span><span class="sxs-lookup"><span data-stu-id="7196b-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="7196b-185">Если еще не обслуживается SPA, откройте второе окно CLI и измените текущий каталог на **каталог ./TestClient.**</span><span class="sxs-lookup"><span data-stu-id="7196b-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="7196b-186">Запустите следующую команду для запуска тестового приложения.</span><span class="sxs-lookup"><span data-stu-id="7196b-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="7196b-187">Откройте браузер и перейдите по адресу `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="7196b-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="7196b-188">Впишите и выберите элемент **навигации "Последнее** сообщение".</span><span class="sxs-lookup"><span data-stu-id="7196b-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="7196b-189">Приложение отображает сведения о самом новом сообщении в почтовом ящике пользователя.</span><span class="sxs-lookup"><span data-stu-id="7196b-189">The app displays information about the newest message in the user's inbox.</span></span>
