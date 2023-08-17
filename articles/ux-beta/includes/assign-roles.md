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

When developing locally, make sure that the user account that is accessing blob data has the correct permissions. If you plan to use the `DefaultAzureCredential` approach you will need `User Access Administrator` role or higher in order to assign yourself a data access role. You can assign Azure RBAC roles to a user using the Azure portal, Azure CLI, or Azure PowerShell. Roles can be assigned at the following levels:

* Specific resources, such as a storage account or a blob container
* Resource group
* Subscription
* Management group

In this scenario, you'll assign permissions to your user account at the resource level to follow the [Principle of Least Privilege](/azure/active-directory/develop/secure-least-privileged-access). This practice gives users only the minimum permissions needed and creates more secure production environments.

The following example will assign the `Storage Blob Data Contributor` role to your user account, which provides both read and write access to blob data in your storage account.

### [Azure portal](#tab/roles-azure-portal)

1. In the Azure portal, locate your storage account using the main search bar or left navigation.

1. On the storage account overview page, select **Access control (IAM)** from the left-hand menu.	

1. On the **Access control (IAM)** page, select **+ Add** from the top menu and then **Add role assignment** from the resulting drop-down menu.

    :::image type="content" source="../media/migration-add-role.png" alt-text="A screenshot showing how to assign a role.":::

1. Use the search box to filter the results to the desired role. For this example, search for *Storage Blob Data Contributor* and select the matching result and then choose **Next**.

1. Under **Assign access to**, make sure **User, group, or service principal** is selected, and then choose **+ Select members**.

1. In the dialog, search for your Azure AD username (usually your email address) and then choose **Select** at the bottom of the dialog. 

1. Select **Review + assign** to go to the final page, and then **Review + assign** again to complete the process.

### [Azure CLI](#tab/roles-azure-cli)

To assign a role at the resource level using the Azure CLI, you first must retrieve the resource ID using the `az storage account show` command. You can filter the output properties using the `--query` parameter. 

```azurecli
az storage account show --resource-group '<your-resource-group-name>' --name '<your-storage-account-name>' --query id
```

Copy the output `Id` from the preceding command. You can then assign roles using the [az role](/cli/azure/role) command of the Azure CLI.

```azurecli
az role assignment create --assignee "<your-username>" \
--role "Storage Blob Data Contributor" \
--scope "<your-resource-id>"
```

### [PowerShell](#tab/roles-powershell)

To assign a role at the resource level using Azure PowerShell, you first must retrieve the resource ID using the `Get-AzResource` command.

```azurepowershell
Get-AzResource -ResourceGroupName "<yourResourceGroupname>" -Name "<yourStorageAccountName>"
```

Copy the `Id` value from the preceding command output. You can then assign roles using the [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) command of the Azure CLI.

```azurepowershell
New-AzRoleAssignment -SignInName <yourUserName> `
-RoleDefinitionName "Storage Blob Data Contributor" `
-Scope <yourStorageAccountId>
```

--- 

> [!IMPORTANT]
> It may take a few minutes for your assigned roles to become active on the resource. If you receive authentication errors when you first run your code, wait a few moments and try again.