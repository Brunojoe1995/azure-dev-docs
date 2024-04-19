---
title: Configure template sources
description: Learn how to configure the Azure Developer CLI to use different template sources
author: alexwolfmsft
ms.author: alexwolf
ms.date: 04/19/2024
ms.service: azure-dev-cli
ms.topic: how-to
ms.custom: devx-track-azdevcli
---

# Configure and consume template sources

The Azure Developer CLI is designed around a powerful template system that streamlines deploying and provisioning Azure resources. While developing with `azd`, you have the option to either build your own template, or choose from a configurable list of existing templates. In this article, you learn how to work with template lists and configure your local `azd` installation to support different template list sources.

## Understand template sources

An `azd` template source points to a JSON configuration file that describes a list of available templates and their essential metadata, such as the title, description, and location of the template source code (usually a GitHub repo). When you enable a template source, the templates it defines are made available to `azd` through other commands. For example, the template source JSON snippet below defines two templates:

```json
[
  {
    "title": "Starter - Bicep",
    "description": "A starter template with Bicep as infrastructure provider",
    "preview": "./templates/images/test.png",
    "website": "https://github.com/Azure/azure-dev",
    "author": "Azure Dev",
    "source": "https://github.com/Azure-Samples/azd-starter-bicep",
    "tags": ["bicep", "msft"]
  },
  {
    "title": "Starter - Terraform",
    "description": "A starter template with Terraform as infrastructure provider",
    "preview": "./templates/images/test.png",
    "website": "https://github.com/Azure/azure-dev",
    "author": "Azure Dev",
    "source": "https://github.com/Azure-Samples/azd-starter-terraform",
    "tags": ["terraform", "msft"]
  }
}
```

`azd` allows you to enable multiple template source at a time. The following template source options are currently available to choose from:

- **default** - A small set of preselected templates to demonstrate different tech stacks.
- **awesome-azd** - A list of the templates from the [Awesome AZD gallery] that is enabled by default.
- **file** -  A local/network path that points to a template source JSON configuration file.
- **url** - A an http/https addressable path that points to a template source JSON configuration file.
- **ade** - Points to an Azure Deployment Environment template list. [Learn more about Azure Developer CLI support for Azure Deployment Environments](/azure/developer/azure-developer-cli/ade-integration).

## Work with template sources

`azd` provides several commands to configure template sources.

Use the `azd template source list` command to list all currently configured template sources:

```azurecli
azd template source list
```

Example output with two configured template sources:

```output
Key          Name         Type         Location

awesome-azd  Awesome AZD  awesome-azd  https://aka.ms/awesome-azd/templates.json
default      Default      resource
```

Use the `azd template source add` command to add a new template source. This command accepts the following parameters:

- **key**: The technical name of the template source.
- **--type, -t**: The source type of **file** or **url**.
- **--location, -l**: The template source location, which should be a local network or HTTP web URI.
- **--displayName, -n**: The template source display name (optional, will use **key** value if omitted).

```azurecli
azd template source add <key> --type url --location <your-uri> --displayname <your-display-name>
```

Use the `azd template source remove` command to remove a template source:

```azurecli
azd template source remove <key>
```

Use the `azd config reset` command to reset the template configuration back to default settings:

```azurecli
azd config reset
```

## Work with template lists

After you configure your template sources, use the `azd template list` command to list the available templates from those sources:

```azurecli
azd template list
```

For example, a default installation of `azd` lists the following templates from the **awesome-azd** template source:

```output
Name                                                                                               Source       Repository Path

Event Driven Java Application with Azure Service Bus on Azure Spring Apps                          Awesome AZD  Azure-Samples/ASA-Samples-Event-Driven-Application
Static React Web App with Java API and PostgreSQL                                                  Awesome AZD  Azure-Samples/ASA-Samples-Web-Application
SAP CAP on Azure App Service Quickstart                                                            Awesome AZD  Azure-Samples/app-service-javascript-sap-cap-quickstart
SAP Cloud SDK on Azure App Service Quickstart (TypeScript)                                         Awesome AZD  Azure-Samples/app-service-javascript-sap-cloud-sdk-quickstart
Java Spring Apps with Azure OpenAI                                                                 Awesome AZD  Azure-Samples/app-templates-java-openai-springapps
WordPress with Azure Container Apps                                                                Awesome AZD  Azure-Samples/apptemplate-wordpress-on-ACA
Bicep template to bootstrap Azure Deployment Environments                                          Awesome AZD  Azure-Samples/azd-deployment-environments
Starter - Bicep                                                                                    Awesome AZD  Azure-Samples/azd-starter-bicep
Starter - Terraform                                                                                Awesome AZD  Azure-Samples/azd-starter-terraform

# Additional templates omitted 
```

Include the `--source` flag to only list templates from a specific source:

```azurecli
azd template list --source <source-name>
```

To initialize a template from the displayed list, run the `azd init` command and provide the path of the template:

```azurecli
azd init --template <path-value>
```

## Work with Azure Deployment Environments

The Azure Developer CLI (`azd`) also provides support for Azure Deployment Environments. An Azure Deployment Environment (ADE) is a preconfigured collection of Azure resources deployed in predefined subscriptions. Azure governance is applied to those subscriptions based on the type of environment, such as sandbox, testing, staging, or production. With Azure Deployment Environments, your can enforce enterprise security policies and provide a curated set of predefined infrastructure as code (IaC) templates.

ADE integration is beyond the scope of this article. Learn more about configuring `ade` support in the [Azure Developer CLI support for Azure Deployment Environments](/azure/developer/azure-developer-cli/ade-integration) documentation.

## Next steps

> [!div class="nextstepaction"]
> [Azure Developer CLI support for Azure Deployment Environments](/azure/developer/azure-developer-cli/ade-integration) documentation.
> [Template list command reference](/azure/developer/azure-developer-cli/reference#azd-template)
