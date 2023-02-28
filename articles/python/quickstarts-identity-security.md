---
title: Overview and resources for Azure identity and security features for Python apps
description: Overview and links to resources about authentication, identity, and security for Python apps on Azure.
ms.date: 01/24/2023
ms.topic: conceptual
ms.custom: devx-track-python, py-fresh-zinc
---

# Identity and security for Python apps on Azure

Identity and security for Python apps on Azure are fundamentally about the [*authentication*](/azure/architecture/framework/security/design-identity-authentication) of the identity of a user, group, application, or service and [*authorization*](/azure/architecture/framework/security/design-identity-authorization) of that identity to perform requested actions on Azure resources. There are different identity and security options you can choose from depending on your application and security needs. This article provides links to resources to help you get started.

## Passwordless connections

Whenever possible, we recommend you use managed identities to simplify overall management and improve security. Specifically, use *passwordless connections* to avoid using embedding sensitive data such as passwords in code or environment variables.

* [Overview: Passwordless connection for Azure services](../intro/passwordless-overview.md)

* [Authenticate Python Apps to Azure services using the Azure SDK for Python](./sdk/authentication-overview.md)

* [Use DefaultAzureCredential in an application](./sdk/authentication-overview.md#use-defaultazurecredential-in-an-application)

* [Quickstart: Azure Blob Storage client library for Python with passwordless connections](/azure/storage/blobs/storage-quickstart-blobs-python)

* [Quickstart: Send messages to and receive message from Azure Service Bus queues with passwordless connections](/azure/service-bus-messaging/service-bus-python-how-to-use-queues)

* [Tutorial: Deploy a Python web app to Azure with managed identity](./tutorial-python-managed-identity-01.md)

The resources listed show how to use Azure Python SDK and passwordless connections with the [DefaultAzureCredential](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/identity/azure-identity#defaultazurecredential). The `DefaultAzureCredential` is appropriate for most applications that will run in Azure because it combines common production credentials with development credentials.

## Service Connector

Many Azure resources you're likely to use with to your Python apps enable the [Service Connector](/azure/service-connector/overview) service. Service Connector helps you configure network settings and connection information between Azure services such as App Service and Container Apps and other services such as storage or databases.

* [Quickstart: Create a service connection in App Service from the Azure portal](/azure/service-connector/quickstart-portal-app-service-connection)

* [Tutorial: Using Service Connector to build a Django app with Postgres on Azure App Service](/azure/service-connector/tutorial-django-webapp-postgres-cli)

## Key Vault

Using a key management solution like [Azure Key Vault](/azure/key-vault/general/overview) gives you more control but with an increase in management complexity.

* [Quickstart: Azure Key Vault certificate client library for Python](/azure/key-vault/certificates/quick-create-python)

* [Quickstart: Azure Key Vault keys client library for Python](/azure/key-vault/keys/quick-create-python)

* [Quickstart: Azure Key Vault secret client library for Python](/azure/key-vault/secrets/quick-create-python)

## Authentication and identity for signing in users in apps

You can build Python applications that enable your users and customers to sign in using their Microsoft identities or social accounts. Your app authorizes access to your own APIs or Microsoft APIs like Microsoft Graph.

* [Quickstart: Add sign-in with Microsoft to a web app](/azure/active-directory/develop/web-app-quickstart)

* [Quickstart: Acquire a token and call the Microsoft Graph API by using a console app's identity](/azure/active-directory/develop/console-app-quickstart?pivots=devlang-python)