---
title: Additional methods to authenticate to Azure resources from Python apps
description: This article describes additional, less common methods you can use to authenticate your Python app to Azure resources. 
ms.date: 03/31/2022
ms.topic: how-to
ms.custom: devx-track-python
---

# Additional methods to authenticate to Azure resources from Python apps

This article lists additional methods apps may use to authenticate to Azure resources.  The methods on this page are less commonly used and when possible, it is encouraged to use one of the methods outlined in the [authenticating Python apps to Azure using the Azure SDK overview](./authentication-overview.md) article.

## Interactive browser authentication

This method interactively authenticates an application through [`InteractiveBrowserCredential`](/python/api/azure-identity/azure.identity.interactivebrowsercredential) by collecting user credentials in the default system.

Interactive browser authentication enables the application for all operations allowed by the interactive login credentials. As a result, if you are the owner or administrator of your subscription, your code has inherent access to most resources in that subscription without having to assign any specific permissions.  For this reason, the use of interactive browser authentication is discouraged for anything but experimentation.

### Enable applications for interactive browser authentication

Perform the following steps to enable the application to authenticate through the interactive browser flow. These steps also work for [device code authentication](#device-code-authentication) described later. Following this process is necessary only if using `InteractiveBrowserCredential` in your code.

1. On the [Azure portal](https://portal.azure.com), navigate to Microsoft Entra ID and select **App registrations** on the left-hand menu.
1. Select the registration for your app, then select **Authentication**.
1. Under **Advanced settings**, select **Yes** for **Allow public client flows**.
1. Select **Save** to apply the changes.
1. To authorize the application for specific resources, navigate to the resource in question, select **API Permissions**, and enable **Microsoft Graph** and other resources you want to access. Microsoft Graph is usually enabled by default.
    1. You must also be the admin of your tenant to grant consent to your application when you log in for the first time.

If you can't configure the device code flow option on your Active Directory, your application may need to be multi-tenant. To make this change, navigate to the **Authentication** panel, select **Accounts in any organizational directory** (under **Supported account types**), and then select **Yes** for **Allow public client flows**.

### Example using InteractiveBrowserCredential

The following example demonstrates using an [`InteractiveBrowserCredential`](/python/api/azure-identity/azure.identity.interactivebrowsercredential) to authenticate with the [`SubscriptionClient`](/python/api/azure-mgmt-resource/azure.mgmt.resource.subscriptions.v2019_06_01.subscriptionclient):

:::code language="python" source="~/../python-sdk-docs-examples/show_subscription/use_interactive_browser.py":::

For more exact control, such as setting redirect URIs, you can supply specific arguments to `InteractiveBrowserCredential` such as `redirect_uri`.

## Device code authentication

This method interactively authenticates a user on devices with limited UI (typically devices without a keyboard):

1. When the application attempts to authenticate, the credential prompts the user with a URL and an authentication code.
1. The user visits the URL on a separate browser-enabled device (a computer, smartphone, etc.) and enters the code.
1. The user follows a normal authentication process in the browser.
1. Upon successful authentication, the application is authenticated on the device.

For more information, see [Microsoft identity platform and the OAuth 2.0 device authorization grant flow](/azure/active-directory/develop/v2-oauth2-device-code).

Device code authentication in a development environment enables the application for all operations allowed by the interactive login credentials. As a result, if you are the owner or administrator of your subscription, your code has inherent access to most resources in that subscription without having to assign any specific permissions. However, you can use this method with a specific client ID, rather than the default, for which you can assign specific permissions.

## Authentication with a username and password

This method authenticates an application using previous-collected credentials and the [`UsernamePasswordCredential`](/python/api/azure-identity/azure.identity.usernamepasswordcredential) object.

This method of authentication is discouraged because it's less secure than other flows. Also, this method is not interactive and is therefore **not compatible with any form of multi-factor authentication or consent prompting.** The application must already have consent from the user or a directory administrator.

Furthermore, this method authenticates only work and school accounts; Microsoft accounts are not supported. For more information, see [Sign up your organization to use Microsoft Entra ID](/azure/active-directory/fundamentals/sign-up-organization).

:::code language="python" source="~/../python-sdk-docs-examples/show_subscription/use_username_password.py":::
