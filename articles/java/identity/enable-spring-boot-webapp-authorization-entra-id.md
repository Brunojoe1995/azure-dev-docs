---
title: Enable Spring Boot app sign-in & access to Microsoft Graph
titleSuffix: Azure
description: Shows you how to develop a Java Spring Boot web app to sign in users and call Microsoft Graph with the Microsoft identity platform.
services: active-directory
ms.date: 03/11/2024
ms.service: active-directory
ms.topic: article
ms.custom: devx-track-java, devx-track-extended-java
---

# Enable Java Spring Boot apps to sign in users and access Microsoft Graph

This article demonstrates a Java Spring Boot web app that signs in users and obtains an access token for calling [Microsoft Graph](/graph/overview). It uses the [Microsoft Entra ID Spring Boot Starter client library for Java](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/spring/spring-cloud-azure-starter-active-directory) for authentication, authorization, and token acquisition. It uses [Microsoft Graph SDK for Java](https://github.com/microsoftgraph/msgraph-sdk-java) to obtain data from Graph.

The following diagram shows the topology of the app:

:::image type="content" source="media/topology.png" alt-text="Diagram that shows the topology of the app.":::

The app uses the Microsoft Entra ID Spring Boot Starter client library for Java to obtain an [access token](/entra/identity-platform/access-tokens) for [Microsoft Graph](/graph/overview) from Microsoft Entra ID. The access token proves that the user is authorized to access the Microsoft Graph API endpoint as defined in the scope.

## Prerequisites

[!INCLUDE [prerequisites-spring-boot.md](includes/prerequisites-spring-boot.md)]

[!INCLUDE [spring-boot-overview-recommendations.md](includes/spring-boot-overview-recommendations.md)]

## Set up the sample

The following sections show you how to set up the sample application.

### Clone or download the sample repository

To clone the sample, open a Bash window and use the following command:

```bash
git clone https://github.com/Azure-Samples/ms-identity-java-spring-tutorial.git
cd ms-identity-java-spring-tutorial
cd 2-Authorization-I/call-graph
```

Alternatively, navigate to the [ms-identity-java-spring-tutorial](https://github.com/Azure-Samples/ms-identity-java-spring-tutorial) repository, then download it as a *.zip* file and extract it to your hard drive.

> [!IMPORTANT]
> To avoid file path length limitations on Windows, clone or extract the repository into a directory near the root of your hard drive.

### Register the sample applications with your Azure Active Directory tenant

There's one project in this sample. To register the app on the Azure portal, you can either follow manual configuration steps or use a PowerShell script. The script does the following tasks:

- Creates the Microsoft Entra ID applications and related objects, such as passwords, permissions, and dependencies.
- Modifies the project configuration files.

### [Use PowerShell](#tab/PowerShell)

Use the following steps to run the PowerShell script:

1. On Windows, run PowerShell as administrator and navigate to the root of the cloned directory.

1. If you're new to Azure AD PowerShell, see [App Creation Scripts](https://github.com/Azure-Samples/ms-identity-java-spring-tutorial/blob/main/2-Authorization-I/call-graph/AppCreationScripts/AppCreationScripts.md) in the source repository to ensure that your environment is prepared correctly.

1. Use the following command to set the execution policy for PowerShell:

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
   ```

1. Use the following commands to run the configuration script:

   ```powershell
   cd .\AppCreationScripts\
   .\Configure.ps1
   ```

   > [!NOTE]
   > Other ways of running the scripts are described in [App Creation Scripts](https://github.com/Azure-Samples/ms-identity-java-spring-tutorial/blob/main/2-Authorization-I/call-graph/AppCreationScripts/AppCreationScripts.md). The scripts also provide a guide to automated application registration, configuration, and removal, which can help in your CI/CD scenarios.

### [Use manual steps](#tab/Manual)

The following sections show you how to register the app manually.

#### Choose the Microsoft Entra ID tenant where you want to create your applications

To choose your tenant, use the following steps:

1. Sign in to the [Azure portal](https://portal.azure.com).

1. If your account is present in more than one Microsoft Entra ID tenant, select your profile in the corner of the Azure portal, and then select **Switch directory** to change your session to the desired Microsoft Entra ID tenant.

#### Register the app (java-spring-webapp-call-graph)

To register the app, use the following steps:

1. Navigate to the [Azure portal](https://portal.azure.com) and select **Microsoft Entra ID**.

1. Select **App Registrations** on the navigation pane, then select **New registration**.

1. In the **Register an application page** that appears, enter your application's registration information:

   - In the **Name** section, enter a meaningful application name for display to users of the app - for example, `java-spring-webapp-call-graph`.
   - Under **Supported account types**, select **Accounts in this organizational directory only**.
   - In the **Redirect URI (optional)** section, select **Web** in the combo-box and enter the following redirect URI: `http://localhost:8080/login/oauth2/code/`.

1. Select **Register** to create the application.

1. In the app's registration screen, find and copy the **Application (client) ID** value to use later. You use this value in your app's configuration file or files.

1. In the app's registration screen, select **Certificates & secrets** on the navigation pane to open the page where we can generate secrets and upload certificates.

1. In the **Client secrets** section, select **New client secret**.

1. Type a key description - for example, *app secret*.

1. Select one of the available key durations: **In 1 year**, **In 2 years**, or **Never Expires**.

1. Select **Add**. The generated key value is displayed.

1. Copy and save the generated value for use in later steps. You need this key for your code's configuration files. This key value isn't displayed again, and you can't retrieve it by any other means. So, be sure to save it from the Azure portal before you navigate to any other screen or pane.

1. In the app's registration screen, select the **API permissions** pane on the navigation pane to open the page for access to the APIs that your application needs.

   - Select **Add permissions** and then,
   - Ensure that the **Microsoft APIs** tab is selected.
   - In the **Commonly used Microsoft APIs** section, select **Microsoft Graph**.
   - In the **Delegated permissions** section, select the **User.Read** in the list. Use the search box if necessary.
   - Select **Add permissions**.

---

#### Configure the app (java-spring-webapp-call-graph) to use your app registration

Use the following steps to configure the app:

> [!NOTE]
> In the following steps, `ClientID` is the same as `Application ID` or `AppId`.

1. Open the project in your IDE.

1. Open the *src\main\resources\application.yml* file.

1. Find the key `Enter_Your_Tenant_ID_Here` and replace the existing value with your Microsoft Entra tenant ID.

1. Find the key `Enter_Your_Client_ID_Here` and replace the existing value with the application ID or `clientId` of the `java-spring-webapp-call-graph` app copied from the Azure portal.

1. Find the key `Enter_Your_Client_Secret_Here` and replace the existing value with the key you saved during the creation of `java-spring-webapp-call-graph` copied from the Azure portal.

## Run the sample

### [Deploy to Azure Spring Apps](#tab/asa)

The following sections show you how to deploy the sample to Azure Spring Apps.

### Prerequisites

[!INCLUDE [deploy-spring-apps-intro.md](includes/deploy-spring-apps-intro.md)]

### Prepare the Spring project

[!INCLUDE [deploy-spring-apps-prepare.md](includes/deploy-spring-apps-prepare.md)]

### Configure the Maven plugin

[!INCLUDE [deploy-spring-apps-configure-maven.md](includes/deploy-spring-apps-configure-maven.md)]

### Prepare the web app for deployment

[!INCLUDE [deploy-spring-apps-prepare-deploy.md](includes/deploy-spring-apps-prepare-deploy.md)]

[!INCLUDE [deploy-spring-apps-secret-note.md](includes/deploy-spring-apps-secret-note.md)]

### Update your Microsoft Entra ID app registration

[!INCLUDE [deploy-spring-apps-update-registration.md](includes/deploy-spring-apps-update-registration.md)]

### Deploy the app

[!INCLUDE [deploy-spring-apps-deploy.md](includes/deploy-spring-apps-deploy.md)]

### Validate the app

[!INCLUDE [deploy-spring-apps-validate.md](includes/deploy-spring-apps-validate.md)]

### [Run locally](#tab/local)

To run the sample locally, use the following steps:

1. Open a Bash window or the integrated Visual Studio Code terminal.

1. In the root directory of the app project, use the following command:

   ```bash
   mvn clean compile spring-boot:run
   ```

1. Open your browser and navigate to `http://localhost:8080`. You should see a screen with the text `You're signed in! Click here to get your ID Token Details`.

:::image type="content" source="media/app.png" alt-text="Screenshot of the sample app.":::

---

## Explore the sample

- Notice the signed-in or signed-out status displayed at the center of the screen.
- Select the context-sensitive button in the corner. This button reads **Sign In** when you first run the app.
  - Alternatively, select **token details** or **call graph**. Because this page is protected and requires authentication, you're automatically redirected to the sign-in page.
- On the next page, follow the instructions and sign in with an account in the Microsoft Entra ID tenant.
- On the consent screen, note the scopes that are being requested.
- Upon successful completion of the sign-in flow, you should be redirected to the home page - which shows the **sign in status** - or one of the other pages, depending on which button triggered your sign-in flow.
- Notice that the context-sensitive button now says **Sign out** and displays your username.
- If you're on the home page, select **ID Token Details** to see some of the ID token's decoded claims.
- Select **Call Graph** to make a call to Microsoft Graph's [/me endpoint](/graph/api/user-get?tabs=java#example-2-signed-in-user-request) and see a selection of user details obtained.
- You can also use the button in the corner to sign out. The status page reflects the new state.

## Contents

| File/folder                                                                   | Description                                                                               |
|-------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| *AppCreationScripts/*                                                         | Scripts to automatically configure Microsoft Entra ID app registrations.                  |
| *pom.xml*                                                                     | Application dependencies.                                                                 |
| *src/main/resources/templates/*                                               | Thymeleaf Templates for UI.                                                               |
| *src/main/resources/application.yml*                                          | Application and Microsoft Entra ID Boot Starter Library Configuration.                    |
| *src/main/java/com/microsoft/azuresamples/msal4j/msidentityspringbootwebapp/* | This directory contains the main application entry point, controller, and config classes. |
| *.../MsIdentitySpringBootWebappApplication.java*                              | Main class.                                                                               |
| *.../SampleController.java*                                                   | Controller with endpoint mappings.                                                        |
| *.../SecurityConfig.java*                                                     | Security Configuration - for example, which routes require authentication.                |
| *.../Utilities.java*                                                          | Utility Class - for example, filter ID token claims.                                      |
| *CHANGELOG.md*                                                                | List of changes to the sample.                                                            |
| *CONTRIBUTING.md*                                                             | Guidelines for contributing to the sample.                                                |
| *LICENSE*                                                                     | The license for the sample.                                                               |

## About the code

This sample demonstrates how to use [Microsoft Entra ID Spring Boot Starter client library for Java](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/spring/spring-cloud-azure-starter-active-directory) to sign in users into your Microsoft Entra ID tenant and obtain an access token for calling **Microsoft Graph**. It also makes use of **Spring Oauth2 Client** and **Spring Web** boot starters.

### Project initialization

Create a new Java Maven project and copy the *pom.xml* file from this project, and the *src* folder of this repository.

If you'd like to create a project like this from scratch, you can use [Spring Initializer](https://start.spring.io):

- For **Packaging**, select **Jar**.
- For **Java**, select version **11**.
- For **Dependencies**, add the following items:
  - **Azure Active Directory**
  - **Spring Oauth2 Client**
  - **Spring Web**
- Be sure that it comes with Azure SDK version 3.3 or higher. If not, consider replacing the preconfigured *pom.xml* with the *pom.xml* from this repository.

### ID token claims

To extract token details, make use of Spring Security's `AuthenticationPrincipal` and `OidcUser` object in a request mapping. See the [Sample Controller](https://github.com/Azure-Samples/ms-identity-java-spring-tutorial/blob/main/2-Authorization-I/call-graph/src/main/java/com/microsoft/azuresamples/msal4j/msidentityspringbootwebapp/SampleController.java) for an example of this app making use of ID token claims.

```java
import org.springframework.security.oauth2.core.oidc.user.OidcUser;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
//...
@GetMapping(path = "/some_path")
public String tokenDetails(@AuthenticationPrincipal OidcUser principal) {
    Map<String, Object> claims = principal.getIdToken().getClaims();
}
```

### Sign-in and sign-out links

To sign in, you must make a request to the Azure Active Directory sign-in endpoint automatically configured by Microsoft Entra ID Spring Boot Starter client library for Java, as shown in the following example:

```html
<a class="btn btn-success" href="/oauth2/authorization/azure">Sign In</a>
```

To sign out, you must make POST request to the `logout` endpoint, as shown in the following example:

```html
<form action="#" th:action="@{/logout}" method="post">
  <input class="btn btn-warning" type="submit" value="Sign Out" />
</form>
```

### Authentication-dependent UI elements

This app has some simple logic in the UI template pages for determining content to display based on whether the user is authenticated or not. For example, you can use the following Spring Security Thymeleaf tags:

```html
<div sec:authorize="isAuthenticated()">
  this content only shows to authenticated users
</div>
<div sec:authorize="isAnonymous()">
  this content only shows to not-authenticated users
</div>
```

### Protect routes with AADWebSecurityConfigurerAdapter

By default, this app protects the **ID Token Details** and **Call Graph** pages so that only logged-in users can access them. This app uses configures these routes from the `app.protect.authenticated` property from the *application.yml* file. To configure your app's specific requirements, extend `AADWebSecurityConfigurationAdapter` in one of your classes. For an example, see this app's [SecurityConfig](https://github.com/Azure-Samples/ms-identity-java-spring-tutorial/blob/main/2-Authorization-I/call-graph/src/main/java/com/microsoft/azuresamples/msal4j/msidentityspringbootwebapp/SecurityConfig.java) class.

```java
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends AADWebSecurityConfigurerAdapter{
  @Value( "${app.protect.authenticated}" )
  private String[] protectedRoutes;

    @Override
    public void configure(HttpSecurity http) throws Exception {
    // use required configuration form AADWebSecurityAdapter.configure:
    super.configure(http);
    // add custom configuration:
    http.authorizeRequests()
      .antMatchers(protectedRoutes).authenticated()     // limit these pages to authenticated users (default: /token_details, /call_graph)
      .antMatchers("/**").permitAll();                  // allow all other routes.
    }
}
```

### Call Graph

When the user navigates to `/call_graph`, the application creates an instance of the `GraphServiceClient` - from the [Microsoft Graph SDK for Java, v3](https://github.com/microsoftgraph/msgraph-sdk-java) - using an `Oauth2AuthorizedClient` or `graphAuthorizedClient` that the Microsoft Entra ID boot starter prepared. The app asks the `GraphServiceClient` to call the `/me` endpoint and displays details for the current signed-in user.

The `Oauth2AuthorizedClient` must be prepared with the correct scopes. See the *application.yml* file and the following [Scopes](#scopes) section. The `Oauth2AuthorizedClient` is used to surface the access token and place it in the Authorization header of `GraphServiceClient` requests.

```java
//see SampleController.java
@GetMapping(path = "/call_graph")
public String callGraph(@RegisteredOAuth2AuthorizedClient("graph") OAuth2AuthorizedClient graphAuthorizedClient) {
  // See the Utilities.graphUserProperties() method for the full example of the following operation:
  GraphServiceClient graphServiceClient = Utilities.getGraphServiceClient(graphAuthorizedClient);
  User user = graphServiceClient.me().buildRequest().get();
  return user.displayName;
}
```

```yml
# see application.yml file
authorization-clients:
  graph:
    # Specifies the Microsoft Graph scopes that your app needs access to:
    scopes: https://graph.microsoft.com/User.Read
```

### Scopes

[Scopes](/entra/identity-platform/scopes-oidc) tell Microsoft Entra ID the level of access that the application is requesting. For the Microsoft Graph scopes requested by this application, see *application.yml*.

- By default, this application sets the scopes value to `https://graph.microsoft.com/User.Read`.
- The `User.Read` scope is for accessing the information of the current signed-in user from the [/me endpoint](https://graph.microsoft.com/v1.0/me).
- Valid requests to the [/me endpoint](https://graph.microsoft.com/v1.0/me) must contain the `User.Read` scope.

When a user signs in, Microsoft Entra ID presents a consent dialogue to the user based on the scopes requested by the application. If the user consents to one or more scopes and obtains a token, the scopes-consented-to are encoded into the resulting access token.

In this app, the `graphAuthorizedClient` surfaces the access token that proves which the scopes the user consented to. The app uses this token to create an instance of `GraphServiceClient` that handles Graph requests.

Using `GraphServiceClient.me().buildRequest().get()`, a request built and made to `https://graph.microsoft.com/v1.0/me`. The access token is placed in the Authorization header of the request.

## More information

- [Microsoft identity platform (Azure Active Directory for developers)](/entra/identity-platform/)
- [Overview of Microsoft Authentication Library (MSAL)](/entra/identity-platform/msal-overview)
- [Quickstart: Register an application with the Microsoft identity platform (Preview)](/entra/identity-platform/quickstart-register-app)
- [Quickstart: Configure a client application to access web APIs (Preview)](/entra/identity-platform/quickstart-configure-app-access-web-apis)
- [Understanding Microsoft Entra ID application consent experiences](/entra/identity-platform/application-consent-experience)
- [Understand user and admin consent](/entra/identity-platform/howto-convert-app-to-be-multi-tenant#understand-user-and-admin-consent-and-make-appropriate-code-changes)
- [Application and service principal objects in Azure Active Directory](/entra/identity-platform/app-objects-and-service-principals)
- [National Clouds](/entra/identity-platform/authentication-national-cloud#app-registration-endpoints)
- [MSAL code samples](/entra/identity-platform/sample-v2-code?tabs=framework#java)
- [Azure Active Directory Spring Boot Starter client library for Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-active-directory)
- [Microsoft Authentication Library for Java (MSAL4J)](https://github.com/AzureAD/microsoft-authentication-library-for-java)
- [MSAL4J Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki)
- [ID tokens](/entra/identity-platform/id-tokens)
- [Access tokens in the Microsoft identity platform](/entra/identity-platform/access-tokens)

For more information about how OAuth 2.0 protocols work in this scenario and other scenarios, see [Authentication Scenarios for Microsoft Entra ID](/entra/identity-platform/authentication-flows-app-scenarios).
