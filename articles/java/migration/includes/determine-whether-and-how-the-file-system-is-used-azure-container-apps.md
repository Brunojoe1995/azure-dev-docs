---
author: KarlErickson
ms.author: karler
ms.date: 9/30/2024
---

#### Determine whether and how the file system is used

Find any instances where your services write to and/or read from the local file system. Identify where short-term/temporary files are written and read and where long-lived files are written and read.

Azure Container Apps offers several types of storage. Ephemeral storage can read and write temporary data and be available to a running container or replica. Azure File provides permanent storage and can be shared across multiple containers. For more information, see [Use storage mounts in Azure Container Apps](/azure/container-apps/storage-mounts).

##### Read-only static content

If your application currently serves static content, you need an alternate location for it. You might wish to consider moving static content to Azure Blob Storage and adding Azure CDN for lightning-fast downloads globally. For more information, see [Static website hosting in Azure Storage](/azure/storage/blobs/storage-blob-static-website) and [Quickstart: Integrate an Azure storage account with Azure CDN](/azure/cdn/cdn-create-a-storage-account-with-cdn).

##### Dynamically published static content

If your application supports static content, whether uploaded or generated by the application itself, that remains unchanged after its creation, you can integrate Azure Blob Storage and Azure CDN. You can also use an Azure Function to manage uploads and trigger CDN refreshes when necessary. We've provided a sample implementation for your use at [Uploading and CDN-preloading static content with Azure Functions](https://github.com/Azure-Samples/functions-java-push-static-contents-to-cdn).