---
title: Create Azure Storage with the Azure libraries for Python
description: Use the Azure SDK for Python libraries to create a blob container in an Azure Storage account and then upload a file to that container.
ms.date: 01/20/2023
ms.topic: conceptual
ms.custom: devx-track-python, devx-track-azurecli, py-fresh-zinc
---

# Example: Create Azure Storage using the Azure libraries for Python

In this article, you learn how to use the Azure management libraries in a Python script to create a resource group that contains and Azure Storage account and a Blob storage container. ([Equivalent Azure CLI commands](#for-reference-equivalent-azure-cli-commands) are given later in this article. If you prefer to use the Azure portal, see [Create an Azure storage account](/azure/storage/common/storage-account-create?tabs=azure-portal) and [Create a blob container](/azure/storage/blobs/storage-quickstart-blobs-portal).)

After creating the resources, see [Example: Use Azure Storage](azure-sdk-example-storage-use.md) to use the Azure client libraries in Python application code to upload a file to the Blob storage container.

All the commands in this article work the same in Linux/macOS bash and Windows command shells unless noted.

## 1: Set up your local development environment

If you haven't already, set up an environment where you can run this code. Here are some options:

[!INCLUDE [create_environment_options](../../includes/create-environment-options.md)]

## 2: Install the needed Azure library packages

1. Create a *requirements.txt* file that lists the management libraries used in this example:

    :::code language="txt" source="~/../python-sdk-docs-examples/storage/requirements_provision.txt":::

1. In your terminal with the virtual environment activated, install the requirements:

    ```cmd
    pip install -r requirements.txt
    ```

## 3: Write code to create storage resources

This section describes how to create storage resources from Python code. If you prefer, you can also create resources through the Azure portal or through the [equivalent Azure CLI commands](#for-reference-equivalent-azure-cli-commands).

Create a Python file named *provision_blob.py* with the following code. The comments explain the details. In particular, you must define your subscription ID as an environment variable `AZURE_SUBSCRIPTION_ID`. The resource group name, location, storage account name, and container name are all defined as constants in the code.

:::code language="python" source="~/../python-sdk-docs-examples/storage/provision_blob.py":::

[!INCLUDE [cli-auth-note](../../includes/cli-auth-note.md)]

### Reference links for classes used in the code

- [AzureCliCredential (azure.identity)](/python/api/azure-identity/azure.identity.azureclicredential)
- [ResourceManagementClient (azure.mgmt.resource)](/python/api/azure-mgmt-resource/azure.mgmt.resource.resourcemanagementclient)
- [StorageManagementClient (azure.mgmt.storage)](/python/api/azure-mgmt-storage/azure.mgmt.storage.storagemanagementclient)

## 4. Run the script

```cmd
python provision_blob.py
```

The script will take a minute or two to complete.

## 5: Verify the resources

1. Open the [Azure portal](https://portal.azure.com) to verify that the resource group and storage account were created as expected. You may need to wait a minute and also select **Show hidden types** in the resource group.

    ![Azure portal page for the new resource group, showing the storage account](../../media/azure-sdk-example-storage/portal-show-hidden-types.png)

1. Select the storage account, then select **Data storage** > **Containers** in the left-hand menu to verify that the "blob-container-01" appears:

    ![Azure portal page for the storage account showing the blob container](../../media/azure-sdk-example-storage/portal-show-blob-containers.png)

1. If you want to try using these resources from application code, continue with [Example: Use Azure Storage](azure-sdk-example-storage-use.md).

For an additional example of using the Azure Storage management library, see the [Manage Python Storage sample](/samples/azure-samples/azure-samples-python-management/storage/).

### For reference: equivalent Azure CLI commands

The following Azure CLI commands complete the same creation steps as the Python script:

# [cmd](#tab/cmd)

:::code language="azurecli" source="~/../python-sdk-docs-examples/storage/provision.cmd":::

# [bash](#tab/bash)

:::code language="azurecli" source="~/../python-sdk-docs-examples/storage/provision.sh":::

---

## 6: Clean up resources

Leave the resources in place if you want to follow the article [Example: Use Azure Storage](azure-sdk-example-storage-use.md) to use these resources in app code. Otherwise, run the [az group delete](/cli/azure/group#az-group-delete) command if you don't need to keep the resource group created in this example.

Resource groups don't incur any ongoing charges in your subscription, but it's a good practice to clean up any group that you aren't actively using. The `--no-wait` argument allows the command to return immediately instead of waiting for the operation to finish.

```azurecli
az group delete -n PythonAzureExample-Storage-rg  --no-wait
```

[!INCLUDE [resource_group_begin_delete](../../includes/resource-group-begin-delete.md)]

## See also

- [Example: Use Azure Storage](azure-sdk-example-storage-use.md)
- [Example: Create a resource group](azure-sdk-example-resource-group.md)
- [Example: List resource groups in a subscription](azure-sdk-example-list-resource-groups.md)
- [Example: Create a web app and deploy code](azure-sdk-example-web-app.md)
- [Example: Create and query a database](azure-sdk-example-database.md)
- [Example: Create a virtual machine](azure-sdk-example-virtual-machines.md)
- [Use Azure Managed Disks with virtual machines](azure-sdk-samples-managed-disks.md)
- [Complete a short survey about the Azure SDK for Python](https://microsoft.qualtrics.com/jfe/form/SV_bNFX0HECjzPWMiG?Q_CHL=docs)
