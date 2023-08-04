---
title: "include file"
description: "include file"
services: storage
author: alexwolfmsft
ms.service: azure-storage
ms.topic: include
ms.date: 02/25/2022
ms.author: alexwolf
ms.custom: include file
---

### [Azure CLI](#tab/sign-in-azure-cli)

Sign-in to Azure through the Azure CLI using the following command:

```azurecli
az login
```

### [Visual Studio](#tab/sign-in-visual-studio)

Select the **Sign in** button in the top right of Visual Studio.

:::image type="content" source="../media/storage-blobs-introduction/sign-in-visual-studio-small.png" alt-text="Screenshot showing the button to sign in to Azure using Visual Studio.":::

Sign-in using the Azure AD account you assigned a role to previously.

:::image type="content" source="../media/storage-blobs-introduction/sign-in-visual-studio-account-small.png" alt-text="Screenshot showing the account selection.":::

### [Visual Studio Code](#tab/sign-in-visual-studio-code)

Make sure you have the [Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account) extension installed.

:::image type="content" source="../media/storage-blobs-introduction/azure-extension.png" alt-text="A screenshot showing the Azure extension.":::

Use the **CTRL + Shift + P** shortcut to open the command palette. Search for the **Azure: Sign In** command and follow the prompts to authenticate. Make sure to use the Azure AD account you assigned a role to previously from your Blob Storage account.

:::image type="content" source="../media/storage-blobs-introduction/azure-command.png" alt-text="A screenshot showing the Azure sign-in command.":::


### [PowerShell](#tab/sign-in-powershell)

Sign-in to Azure using PowerShell via the following command:

```azurepowershell
Connect-AzAccount
```

--- 