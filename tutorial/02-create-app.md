---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655235"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a69ee-101">В этом руководстве вы создадим простую функцию Azure, которая реализует функции триггера HTTP, которые вызывают Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="a69ee-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="a69ee-102">Эти функции охватывают следующие сценарии:</span><span class="sxs-lookup"><span data-stu-id="a69ee-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="a69ee-103">Реализует API для доступа к почтовому [](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) ящику пользователя с помощью проверки подлинности потока "от имени".</span><span class="sxs-lookup"><span data-stu-id="a69ee-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="a69ee-104">Реализует API для подписки на уведомления в почтовом ящике пользователя и [](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) отписаться от них, используя проверку подлинности потока предоставления учетных данных клиента.</span><span class="sxs-lookup"><span data-stu-id="a69ee-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="a69ee-105">Реализует веб-hook для [](https://docs.microsoft.com/graph/webhooks) получения уведомлений об изменениях из Microsoft Graph и доступа к данным с помощью потока предоставления учетных данных клиента.</span><span class="sxs-lookup"><span data-stu-id="a69ee-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="a69ee-106">Вы также создадим простое одно page application JavaScript (SPA) для вызова API, реализованных в функции Azure.</span><span class="sxs-lookup"><span data-stu-id="a69ee-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="a69ee-107">Создание проекта функций Azure</span><span class="sxs-lookup"><span data-stu-id="a69ee-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="a69ee-108">Откройте интерфейс командной строки (CLI) в каталоге, где нужно создать проект.</span><span class="sxs-lookup"><span data-stu-id="a69ee-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="a69ee-109">Выполните следующую команду.</span><span class="sxs-lookup"><span data-stu-id="a69ee-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. <span data-ttu-id="a69ee-110">Измените текущий каталог в CLI на **каталог GraphTutorial** и запустите следующие команды, чтобы создать три функции в проекте.</span><span class="sxs-lookup"><span data-stu-id="a69ee-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. <span data-ttu-id="a69ee-111">Откройте **local.settings.jsи** добавьте в файл следующее, чтобы разрешить CORS с `http://localhost:8080` URL-адреса тестового приложения.</span><span class="sxs-lookup"><span data-stu-id="a69ee-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="a69ee-112">Чтобы запустить проект локально, запустите следующую команду:</span><span class="sxs-lookup"><span data-stu-id="a69ee-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="a69ee-113">Если все работает, вы увидите следующие выходные данные:</span><span class="sxs-lookup"><span data-stu-id="a69ee-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="a69ee-114">Убедитесь, что функции работают правильно, открыв браузер и открыв URL-адреса функций, показанные в выходных данных.</span><span class="sxs-lookup"><span data-stu-id="a69ee-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="a69ee-115">В браузере должно отобраться следующее `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` сообщение:</span><span class="sxs-lookup"><span data-stu-id="a69ee-115">You should see the following message in your browser: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="a69ee-116">Создание одно page application</span><span class="sxs-lookup"><span data-stu-id="a69ee-116">Create single-page application</span></span>

1. <span data-ttu-id="a69ee-117">Откройте CLI в каталоге, в котором вы хотите создать проект.</span><span class="sxs-lookup"><span data-stu-id="a69ee-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="a69ee-118">Создайте каталог с **именем TestClient** для удержания файлов HTML и JavaScript.</span><span class="sxs-lookup"><span data-stu-id="a69ee-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="a69ee-119">Создайте файл с **именемindex.html** в **каталоге TestClient** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="a69ee-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="a69ee-120">Это определяет базовый макет приложения, включая панели навигации.</span><span class="sxs-lookup"><span data-stu-id="a69ee-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="a69ee-121">Он также добавляет следующее:</span><span class="sxs-lookup"><span data-stu-id="a69ee-121">It also adds the following:</span></span>

    - <span data-ttu-id="a69ee-122">[Bootstrap и](https://getbootstrap.com/) поддерживаемый JavaScript</span><span class="sxs-lookup"><span data-stu-id="a69ee-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="a69ee-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="a69ee-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="a69ee-124">Библиотека проверки подлинности Майкрософт для JavaScript (MSAL.js) 2.0</span><span class="sxs-lookup"><span data-stu-id="a69ee-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="a69ee-125">Страница включает в себя факс, ( `<link rel="shortcut icon" href="g-raph.png">` ).</span><span class="sxs-lookup"><span data-stu-id="a69ee-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="a69ee-126">Эту строку можно удалить или скачатьg-raph.png **с** [GitHub.](https://github.com/microsoftgraph/g-raph)</span><span class="sxs-lookup"><span data-stu-id="a69ee-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="a69ee-127">Создайте файл **style.css** в **каталоге TestClient** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="a69ee-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="a69ee-128">Создайте файл с **именемui.js** в **каталоге TestClient** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="a69ee-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="a69ee-129">Этот код использует JavaScript для отображения текущей страницы на основе выбранного представления.</span><span class="sxs-lookup"><span data-stu-id="a69ee-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="a69ee-130">Тестирование однобукв 2-странигового приложения</span><span class="sxs-lookup"><span data-stu-id="a69ee-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="a69ee-131">В этом разделе содержатся инструкции по использованию [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) для запуска простого тестового HTTP-сервера на компьютере разработки.</span><span class="sxs-lookup"><span data-stu-id="a69ee-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="a69ee-132">Использовать это средство не требуется.</span><span class="sxs-lookup"><span data-stu-id="a69ee-132">Using this specific tool is not required.</span></span> <span data-ttu-id="a69ee-133">Можно использовать любой тестовый сервер, который вы предпочитаете обслуживать **каталог TestClient.**</span><span class="sxs-lookup"><span data-stu-id="a69ee-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="a69ee-134">Чтобы установить **dotnet-serve,** запустите следующую команду в CLI.</span><span class="sxs-lookup"><span data-stu-id="a69ee-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="a69ee-135">Измените текущий каталог в CLI на **каталог TestClient** и запустите следующую команду, чтобы запустить HTTP-сервер.</span><span class="sxs-lookup"><span data-stu-id="a69ee-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. <span data-ttu-id="a69ee-136">Откройте браузер и перейдите по адресу `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="a69ee-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="a69ee-137">Страница должна отрисовки, но в настоящее время ни одна из кнопок не работает.</span><span class="sxs-lookup"><span data-stu-id="a69ee-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="a69ee-138">Добавление пакетов NuGet</span><span class="sxs-lookup"><span data-stu-id="a69ee-138">Add NuGet packages</span></span>

<span data-ttu-id="a69ee-139">Прежде чем двигаться дальше, установите некоторые дополнительные пакеты NuGet, которые вы будете использовать позже.</span><span class="sxs-lookup"><span data-stu-id="a69ee-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="a69ee-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) для вреализации зависимостей в проекте функций Azure.</span><span class="sxs-lookup"><span data-stu-id="a69ee-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="a69ee-141">[Microsoft.Extensions.Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) для чтения конфигурации приложения из секретного магазина [разработки .NET.](https://docs.microsoft.com/aspnet/core/security/app-secrets)</span><span class="sxs-lookup"><span data-stu-id="a69ee-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="a69ee-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) для звонков в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="a69ee-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="a69ee-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) для проверки подлинности и управления маркерами.</span><span class="sxs-lookup"><span data-stu-id="a69ee-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="a69ee-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) для ирисовки конфигурации OpenID для проверки маркеров.</span><span class="sxs-lookup"><span data-stu-id="a69ee-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="a69ee-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) для проверки маркеров, отправленных в веб-API.</span><span class="sxs-lookup"><span data-stu-id="a69ee-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="a69ee-146">Измените текущий каталог в CLI на **каталог GraphTutorial** и запустите следующие команды.</span><span class="sxs-lookup"><span data-stu-id="a69ee-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
