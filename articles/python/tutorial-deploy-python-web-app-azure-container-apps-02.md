---
title: Build and deploy a Python web app with Azure Container Apps
description: Describes how to create a container from a Python web app and deploy it to Azure Container Apps, a serverless platform for hosting containerized applications.
ms.topic: conceptual
ms.date: 09/14/2022
ms.custom: devx-track-python
---

# Build and deploy a Python web app with Azure Container Apps and PostgreSQL

This article is part of a tutorial about how to containerize and deploy a Python web app to [Azure Container Apps][8]. Container Apps enables you to deploy containerized apps without managing complex infrastructure.

In this part of the tutorial, you learn how to containerize and deploy a Python sample web app (Django or Flask). Specifically, you'll build the container image in the cloud and deploy it to Azure Container Apps. You'll define environment variables that enable the container app to connect to an [Azure Database for PostgreSQL - Flexible Server][10] instance, where the sample app stores data.

The service diagram shown below highlights the components covered in this article: building and deploying a container image.

:::image type="content" source="./media/tutorial-container-apps/service-diagram-overview-for-tutorial-deploy-python-azure-container-apps-deploy.png" alt-text="A screenshot of the services in the Tutorial - Deploy a Python App on Azure Container Apps. Section highlighted is what is covered in this article." lightbox="./media/tutorial-container-apps/service-diagram-overview-for-tutorial-deploy-python-azure-container-apps-deploy.png":::

> [!NOTE]
> Command lines in this tutorial are shown in the Bash shell, on multiple lines for clarity. For other shell types, change the line continuation characters as appropriate. For example, for PowerShell, use back tick ("\`"). Or, remove the continuation characters and enter the command on one line.

## Get the sample app

Fork and clone the sample code to your developer environment.

**Step 1.** Go to the GitHub repository of the sample app ([Django][1] or [Flask][2]) and select **Fork**.

Follow the steps to fork the directory to your GitHub account. You can also download the code repo directly to your local machine without forking or a GitHub account, however, you won't be able to set up CI/CD discussed later in the tutorial.

**Step 2.** Use the [git clone][21] command to clone the forked repo into the *python-container* folder:

```bash
# Django
git clone https://github.com/$USERNAME/msdocs-python-django-azure-container-apps.git python-container

# Flask
# git clone https://github.com/$USERNAME/msdocs-python-flask-azure-container-apps.git python-container
```

**Step 3.** Change directory.

```bash
cd python-container
```

## Build a container image from web app code

After following these steps, you'll have an Azure Container Registry that contains a Docker container image built from the sample code.

### [Azure portal](#tab/azure-portal)

:::row:::
    :::column span="2":::
        **Step 1.** In the [Azure portal][3], search for "container registries" and select the **Container Registries** service in the results.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-build-image-01.png" alt-text="Screenshot showing how to search for container registries services in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-build-image-01.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 2.** Select **+ Create** to start the create process.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-build-image-02.png" alt-text="Screenshot showing how to start creating a new Azure Container Registry in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-build-image-02.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** Fill out the form and specify.
        * **Resource group** &rarr; Create a new one named *pythoncontainer-rg*.
        * **Registry name** &rarr; The registry name must be unique within Azure, and contain 5-50 alphanumeric characters.
        * **Location** &rarr; Select a location. 
        * **SKU** &rarr; Select **Standard**.

        When finished, select **Review + create**. After  validation is complete, select **Create**.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-build-image-03.png" alt-text="Screenshot showing how to start specify a new Azure Container Registry in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-build-image-03.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 4.** Enable the administrator user account.

        * In the Container registry you just created, go to the **Access Keys** resource.
        * Select **Enabled** for the **Admin user**.

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-build-image-05.png" alt-text="Screenshot showing how to set enable administrator user account for Azure Container Registry in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-build-image-05.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 5.** Select the Azure Cloud Shell icon in the top menu bar to finish configuration and building an image.

        You can also go directly to [Azure Cloud Shell][4].
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-build-image-04.png" alt-text="Screenshot showing how to access Azure Cloud Shell in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-build-image-04.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 6.** Use the [az acr build][5] command to build the image from the repo.

        ```bash
        az acr build --registry <registry-name> \
           --resource-group pythoncontainer-rg \ 
           --image pythoncontainer:latest <repo-path>
        ```
        Specify *\<registry-name>* as the name of the registry you created. For *\<repo-path>*, choose either the [Django][1] or [Flask][2] repo path.

        After the command completes, go to the registry's **Repositories** resource and confirm the image shows up.
    :::column-end:::
:::row-end:::

### [VS Code](#tab/vscode-aztools)

These steps require the [Docker extension][6] for VS Code.

:::row:::
    :::column span="2":::
        **Step 1.** Start the build image task.
        
        * Select **F1** or **CTRL+SHIFT+P** to open the command palette.
        * Type "Azure Container Registry".
        * Select the task **Azure Container Registry: Build Image in Azure**.      

        Alternatively, right-click the *Dockerfile* and select **Build Image in Azure**. This UI action starts the same create registry task. If you don't see the **Build Image Azure** task, check if you are signed into Azure.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-build-image-01.png" alt-text="Screenshot showing how to start creating a new Azure Container Registry in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-build-image-01.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 2.** Follow the prompts to create a registry, a resource group, and build the image.
        
        * **Tag image as** &rarr; Enter *pythoncontainer:latest*.
        * **Create new registry** &rarr; Select this option to create new registry.
        * **Registry name** &rarr; The registry name must be unique within Azure, and contain 5-50 alphanumeric characters. 
        * **Select a SKU** &rarr; Select **Basic**.
        * **Create a new resource group** &rarr; Select this option to create resource group.
        * **Resource group** &rarr; Create a new resource group named *pythoncontainer-rg*.
        * **Location** &rarr; Select a location and wait a few seconds for the final prompt for the base image OS.
        * **Select OS** &rarr; Select **Linux**.

        If you see an error in the **Output** window, see the [Troubleshooting section](#troubleshoot-deployment).
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-build-image-02.gif" alt-text="Screenshot showing how to specify configuration of a new Azure Container Registry in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-build-image-02.gif":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** Confirm the registry was created.

        Select the Docker extension and to the **Registries** section. Expand the Azure node to find the new Azure Container Registry.  
      
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-build-image-03.png" alt-text="Screenshot showing how to confirm the Azure Container Registry was created in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-build-image-03.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 4.** Use the [az acr update][29] command to enable the administrator user account for the registry. You can run the command in Visual Studio Code terminal window or the Azure [Cloud Shell][4].

        ```bash
        az acr update --name <registry-name> \
           --resource-group pythoncontainer-rg \
           --admin-enabled true
        ```

        Alternatively, select the registry in the Docker extension, right-click and select **Open in Portal** to enable the administrator user account. See the instructions int the portal tab of this article.

        You can view the credentials created for admin with:

        ```bash
        az acr credential show \
           --name  <registry-name> \
           --resource-group pythoncontainer-rg
        ```
    :::column-end:::
:::row-end:::

### [Azure CLI](#tab/azure-cli)

Azure CLI commands can be run in the [Azure Cloud Shell][4] or on a workstation with the [Azure CLI][7] installed.

:::row:::
    :::column span="1":::
        **Step 1.** Create a resource group with the [az group create][17] command.

        ```bash        
        az group create \
           --name pythoncontainer-rg \
           --location <location>
        ```

        *\<location>* is one of the Azure location *Name* values from the output of the command `az account list-locations -o table`.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 2.** Create a container registry with the [az acr create][18] command.

        ```bash
        az acr create \
           --resource-group pythoncontainer-rg \
           --name <registry-name> \
           --sku Basic \
           --admin-enabled
        ```

        *\<registry-name>* must be unique within Azure, and contain 5-50 alphanumeric characters.

        You can view the credentials created for admin with:

        ```bash
        az acr credential show \
           --name <registry-name> \
           --resource-group pythoncontainer-rg
        ```
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 3.** Sign in to the registry using the [az acr sign in][19] command.

        ```bash
        az acr login --name <registry-name>
        ```
        
        The above command adds "azurecr.io" to the name to create the fully qualified registry name. If successful, you'll see the message "Login Succeeded". If you're accessing the registry from a subscription different from the one in which the registry was created, use the `--suffix` switch.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 4.** Build the image with the [az acr build][5] command.

        ```bash
        az acr build \
           --registry <registry-name> \
           --resource-group pythoncontainer-rg \
           --image pythoncontainer:latest .
        ```
        
        Note that:

        * The dot (".") at the end of the command indicates the location of the source code to build. If you aren't running this command in the sample app root directory, specify the path to the code.

        * If you are running the command in Azure Cloud Shell, use `git clone` to first pull the repo into the Cloud Shell environment first and change directory into the root of the project so that dot (".") is interpreted correctly.

        * If you leave out the `-t` (same as `--image`) option, the command queues a local context build without pushing it to the registry. Building without pushing can be useful to check that the image builds.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 5.** Confirm the container image was created with the [az acr repository list][20] command.

        ```bash
        az acr repository list --name <registry-name>
        ```
        :::column-end:::
:::row-end:::

---

## Create a PostgreSQL Flexible Server instance

The sample app ([Django][1] or [Flask][2]) stores restaurant review data in a PostgreSQL database. In these steps, you'll create the server that will contain the database.

### [Azure portal](#tab/azure-portal)

:::row:::
    :::column span="2":::
        **Step 1.** In [Azure portal][3], search for "postgres flexible" and select the **Azure Database for PostgreSQL flexible servers** service in the results.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-postgres-server-01.png" alt-text="Screenshot showing how to search for Azure PostgreSQL Flexible Server resources in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-postgres-server-01.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 2.** Select **+ Create** to start the create process.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-postgres-server-02.png" alt-text="Screenshot showing how to create an Azure PostgreSQL Flexible Server instance in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-postgres-server-02.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** Fill out the **Basics** settings as follows:

        * **Resource group** &rarr; The resource group used in this tutorial "pythoncontainer-rg".
        * **Server name** &rarr; Enter a name for the database server that's unique across all Azure. The database server's URL becomes `https://<server-name>.postgres.database.azure.com`. Allowed characters are `A`-`Z`, `0`-`9`, and `-`. For example: *postgres-db-\<unique-id>*.
        * **Region** &rarr; The same region you used for the resource group.
        * **Admin username** &rarr; Use *demoadmin*.
        * **Password** and **Confirm password** &rarr; A password that you'll use later when connecting the container app to this database.

        For all other settings, leave the defaults. When done, select **Networking** to go to the networking page.

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-postgres-server-03.png" alt-text="Screenshot showing how to specify basic settings of an Azure PostgreSQL Flexible Server instance in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-postgres-server-03.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** Fill out the **Networking** settings as follows:

        * **Connectivity method** &rarr; Select **Public access**.
        * **Allow public access from any Azure service** &rarr; Select the checkbox, that is, allow access. 
        * **Add current client IP address** &rarr; Select (add) if you plan on accessing the database from your local server.

        For all other settings, leave the defaults. Select **Review + Create** to continue.

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-postgres-server-04.png" alt-text="Screenshot showing how to specify networking settings of an Azure PostgreSQL Flexible Server instance in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-postgres-server-04.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** Review the information and when satisfied, select **Create**.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-postgres-server-05.png" alt-text="Screenshot showing how to finish creation of an Azure PostgreSQL Flexible Server instance in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-postgres-server-05.png":::
    :::column-end:::
:::row-end:::

### [VS Code](#tab/vscode-aztools)

These steps require the [Azure Databases extension][26] for VS Code.

:::row:::
    :::column span="2":::
        **Step 1.** Start the PostgreSQL create task.

        * Select **F1** or **CTRL+SHIFT+P** to open the command palette.
        * Type "Azure Databases".
        * Select the task **Azure Databases: Create Server**.      
                        
        Alternatively, select the **Azure** extension, **RESOURCES**, and expand your subscription. (Make sure you viewing resources by **Group by Resource Type**.). Then,
        right-click **PostgreSQL servers** and select  **Create server**. This UI action starts the same create server task. 
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-postgres-server-01.png" alt-text="Screenshot showing how to search for the task to create an Azure PostgreSQL Flexible Server instance in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-postgres-server-01.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 2.** A series of prompts will guide you through the process of creating the database. Fill in the information as follows.

        * **Select an Azure Database Server** &rarr; Select **PostgreSQL Flexible Server**.

        * **Server name** &rarr; Specify a **name** for the server.
           Enter a name for the database server that's unique across all Azure (the database server's URL becomes `https://<server-name>.postgres.database.azure.com`). Allowed characters are `A`-`Z`, `0`-`9`, and `-`. For example: *postgres-db-\<unique-id>*.

        * **Select the Postgres SKU and options** &rarr; Select the **B1 Basic** SKU (1 vCore, 2 GiB Memory, 5-GB storage).

        * **Administrator Username** &rarr; Create an administrator user name. This name for an administrator account on the database server. Record this name and password as you'll need them later in this tutorial.

        * **Administrator Password** &rarr; Create a password for the administrator and confirm it.

        * **Select a resource group for new resources** &rarr; Select a user group to put the database in. Use the same resource group that you created the App Service in.
        
        * **Select a location for new resources** &rarr; Select the same location as the resource group and App Service.

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-postgres-server-02.gif" alt-text="Screenshot showing how to complete the task to create an Azure PostgreSQL Flexible Server instance in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-postgres-server-02.gif":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** Once the database is created, configure access from your local environment to the Azure Database for PostgreSQL server.

        First, confirm that the database was created by checking the **Azure: Activity Log** window. When you are sure the database exists then:

        * Open the Command Palette (**F1** or **Ctrl** + **Shift** + **P**).
        
        * Search for and select **PostgreSQL: Configure Firewall**.  (Select a subscription if prompted.)
        
        * Select the database you created above. If the database name doesn't appear in the list, it's likely it hasn't finished being created.
        
        * Select **Yes** in the dialog box to add your IP address to the firewall rules of the PostgreSQL server.

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-postgres-server-03.gif" alt-text="Screenshot showing how to add local workstation IP as firewall rule for Azure PostgreSQL Flexible Server instance in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-postgres-server-03.gif":::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-postgres-server-04.png" alt-text="Screenshot showing how to Confirm adding local workstation IP as firewall rule for Azure PostgreSQL Flexible Server instance in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-postgres-server-04.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 4.** Add a rule to allow your web app to access the PostgreSQL Flexible server.

        You also need to configure the database server's firewall to accept connections from all Azure resources. To complete this step in VS Code, open an [Azure Cloud Shell][25] in VS Code, or go to [Azure Cloud Shell][4] and follow the Azure CLI instructions. Or, use the Azure portal instructions.        
    :::column-end:::
:::row-end:::

### [Azure CLI](#tab/azure-cli)

Azure CLI commands can be run in the [Azure Cloud Shell][4] or on a workstation with the [Azure CLI][7] installed.

**Step 1.** Use the [az postgres flexible-server create][22] command to create the PostgreSQL server in Azure. It isn't uncommon for this command to run for a few minutes to complete.

```bash
az postgres flexible-server create \
   --resource-group pythoncontainer-rg \
   --name <postgres-server-name>  \
   --location <location> \
   --admin-user <admin-username> \
   --admin-password <admin-password> \
   --sku-name Standard_D2s_v3 \
   --public-access 0.0.0.0 
```

* "pythoncontainer-rg" &rarr; The resource group name used in this tutorial. If you used a different name, change this value.
* *\<postgres-server-name>* &rarr; The PostgreSQL database server name. This name must be **unique across all Azure**. The server endpoint is "https://\<postgres-server-name>.postgres.database.azure.com"). Allowed characters are "A"-"Z", "0"-"9", and "-".
* *\<location>* &rarr; Use the same location used for the web app. *\<location>* is one of the Azure location *Name* values from the output of the command `az account list-locations -o table`.
* *\<admin-username>* &rarr; Username for the administrator account. It can't be "azure_superuser", "admin", "administrator", "root", "guest", or "public". Use "demoadmin" for this tutorial.
* *\<admin-password>* Password of the administrator user. It must contain 8 to 128 characters from three of the following categories: English uppercase letters, English lowercase letters, numbers, and non-alphanumeric characters.

    > [!IMPORTANT]
    > When creating usernames or passwords **do not** use the "$" character. Later you create environment variables with these values where the "$" character has special meaning within the Linux container used to run Python apps.

* `<sku-name>` &rarr; The name of the pricing tier and compute configuration, for example "Standard_D2s_v3". For more information, see [Azure Database for PostgreSQL pricing][24]. To list available SKUs, use `az postgres flexible-server list-skus --location <location>`.

* `<public-access>` &rarr; Use "0.0.0.0", which allows public access to the server from any Azure service, such as Container Apps.

> [!NOTE]
> If you plan on working the PostgreSQL server from your local workstation with tools other than Azure CLI, you'll need to add a firewall rule with the [az postgres flexible-server firewall-rule create][28] command.

---

## Create a database on the server

At this point, you have a PostgreSQL server and now you'll create a database on the server.

### [psql](#tab/create-database-psql)

You can use the PostgreSQL interactive terminal [psql][15] in your local environment, or in the [Azure Cloud Shell][4], which is also accessible in the [Azure portal][3]. When working with psql, it's often easier to use the [Cloud Shell][4] because all the dependencies are included for you in the shell.

**Step 1.** Connect to the database with psql.

```bash
psql --host=<postgres-server-name>.postgres.database.azure.com \
     --port=5432 \
     --username=demoadmin@<postgres-server-name> \
     --dbname=postgres
```

Where *\<postgres-server-name>* is the name of the PostgreSQL server. The command above will prompt you for the admin password.

If you have trouble connecting, restart the database and try again. If you're connecting from your local environment, your IP address must be added to the firewall rule list for the database service. 

**Step 2.** Create the database.

At the `postgres=>` prompt type:

```sql
CREATE DATABASE restaurants_reviews;
```

The semicolon (";") at the end of the command is necessary. To verify that the database was successfully created, use the command `\c restaurants_reviews`. Type `\?` to show help or `\q` to quit.

### [VS Code](#tab/create-database-vscode-aztools)

These steps require the [Azure Databases extension][26] for VS Code.

**Step 1.** In the **Azure** extension, find the PostgreSQL Server you created, right-click it, and select **Create Database**.

**Step 2.** At the prompt, enter *restaurants_reviews* as the **Database Name**.

If you have trouble creating the database, the server may still be processing the firewall rule from the previous step. Wait a moment and try again. If you're prompted to enter credentials to access the database, use the "demoadmin" username, and password you used to create the database.

### [Azure CLI](#tab/create-database-azure-cli)

You can use the Azure CLI anywhere it's installed, including the Azure [Cloud Shell][4].

**Step 1** Use the [az postgres flexible-server db create][27] command to create a "restaurants_reviews" database.

```bash
az postgres flexible-server db create \
   --resource-group pythoncontainer-rg \
   --server-name <postgres-server-name> \
   --database-name restaurants_reviews
```

Where:

* "pythoncontainer-rg" &rarr; The resource group name used in this tutorial. If you used a different name, change this value.
* `<postgres-server-name>` &rarr; The name of the PostgreSQL server.

You could also use the [az postgres flexible-server connect][16] command to connect to the database and then work with [psql][15] commands. When working with psql, it's often easier to use the Azure [Cloud Shell][4] because all the dependencies are included for you in the shell.

---

You can also connect to Azure PostgreSQL Flexible server and create a database using [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) or any other IDE that supports PostgreSQL.

## Deploy the web app to Container Apps

Container apps are deployed to Container Apps [*environments*][30], which act as a secure boundary. In the following steps, you'll create the environment, a container inside the environment, and configure the container so that the website is visible externally.

### [Azure portal](#tab/azure-portal)

:::row:::
    :::column span="2":::
        **Step 1.** In the portal search at the top of the screen, search for "container apps" and select the **Container Apps** service in the results.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-01.png" alt-text="Screenshot showing how to search for the Azure Container Apps service in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-01.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 2.** Select **+ Create** to start the create process.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-02.png" alt-text="Screenshot showing how to start the create process for an Azure Container Apps service in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-02.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** On the **Basics** page, specify the basic configuration of the container app.
        
        * **Resource group** &rarr; Use the group created earlier and contains the Azure Container Registry.        
        * **Container app name** &rarr; *python-container-app*.        
        * **Region** &rarr; Use the same region/location as the resource group.
        * **Container Apps Environment** &rarr; Select **Create new** to create a new environment named *python-container-env*.
        
        Select **Next: App settings** to continue configuration.        

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-03.png" alt-text="Screenshot showing how to start the configure basic settings for an Azure Container Apps service in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-03.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 4.** On the **App settings** page, continue configuring the container app.
        
        * **Use quickstart image** &rarr; Unselect checkbox.        
        * **Name** &rarr; *python-container-app*.
        * **Image Source** &rarr; Select *Azure Container Registry*.        
        * **Registry** &rarr; Select the name of registry you created earlier.        
        * **Image name** &rarr; Select *pythoncontainer* (the name of the image you built).
        * **Image tag** &rarr; Select *latest*.        
        * **HTTP Ingress** &rarr;  Select checkbox (enabled).        
        * **Ingress traffic** &rarr; Select **Accepting traffic from anywhere**.        
        * **Target port**&rarr; Set to 8000 for Django or 5000 for Flask.
        
        Select **Review and create** to go to review page. After reviewing the settings, select **Create** to kick off deployment.

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-04.png" alt-text="Screenshot showing how to the configure app settings for an Azure Container Apps service in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-04.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 5.** After the deployment finishes, select **Go to resource**.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-05.png" alt-text="Screenshot showing the resource deployment complete page for an Azure Container Apps service in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-05.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 6.** Create a [*revision*][31] of the container that contains environment variables.

        * Select the **Containers** resource of the newly created container.
        * Then, select **Edit and deploy**.
        * On the **Create and deploy new revision** page, select the name of the container image, in this case *python-container-app*.
        * On the **Edit container** page, create environment variables as shown below and then select **Save**.
        * Back on the **Create and deploy new revision** page, select **Create**.

        Here are the following environment variables to create:

        * AZURE_POSTGRESQL_HOST=\<postgres-server-name>.postgres.database.azure.com
        * AZURE_POSTGRESQL_DATABASE=restaurants_reviews
        * AZURE_POSTGRESQL_USERNAME=demoadmin
        * AZURE_POSTGRESQL_PASSWORD=\<admin-password>
        * RUNNING_IN_PRODUCTION=1
        
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-06.png" alt-text="Screenshot showing how to edit a Azure Containers Apps container in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-06.png":::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-07.png" alt-text="Screenshot showing how to create a new Azure Container Apps container revision in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-07.png":::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-08.png" alt-text="Screenshot showing how to add environment variables to an Azure Container Apps revision in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-08.png"::: 
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-09.png" alt-text="Screenshot showing how to finish creating revision of an Azure Container Apps container in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-09.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        > [!TIP]
        > Instead of defining environment variables as shown above, you can use [Service Connector][9]. Service Connector helps you connect to Azure compute services to other backing services by configuring connection information and generating and storing environment variables for you. If you use a service connector, make sure you synchronize the environment variables in the sample code to the environment variables created with Service Connector.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 7.** Django only, migrate and create database schema. (In the Flask sample app, it's done automatically, and you can skip this step.)
        * Go to the **Monitoring** - **Console** resource of the container app.
        * Choose a startup command and select **Connect**.
        * At the shell prompt, type `python manage.py migrate`.

        You don't need to migrate for revisions of the container.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-11.png" alt-text="Screenshot showing how to connect to an Azure Container Apps container in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-11.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 8.** Test the website.

        * Go to the container app's **Overview** resource.
        * Under **Essentials**, select **Application Url** to open the website in a browser.
        
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-10.png" alt-text="Screenshot showing how to find the website Url of an Azure Container Apps container in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-10.png":::
    :::column-end:::
:::row-end:::

### [VS Code](#tab/vscode-aztools)

These steps require the [Azure Container Apps extension][11] for VS Code.

:::row:::
    :::column span="1":::
        **Step 1.** Create an *.env* file that you'll reference during the creation of the container app.

        In the sample repo, there is an *.env.example* file you can start from. Create the *.env* file with the following values:
        
        ```
        AZURE_POSTGRESQL_HOST=<postgres-server-name>.postgres.database.azure.com
        AZURE_POSTGRESQL_DATABASE=restaurants_reviews
        AZURE_POSTGRESQL_USERNAME=demoadmin
        AZURE_POSTGRESQL_PASSWORD=<db-password>
        RUNNING_IN_PRODUCTION=1
        ```
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 2.** Start the container apps create task.

        * Select **F1** or **CTRL+SHIFT+P** to open the command palette.
        * Type "containers apps".
        * Select the task **Azure Container Apps: Create Container Apps Environment**

        Alternatively, you can open the Azure extension and select **+** icon in the **Resources** section. 

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-container-app-01.png" alt-text="Screenshot showing how to search for the Azure Container Apps create task in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-container-app-01.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 3.** Follow the steps to create the container environment.

        * **Select Container Apps environment** &rarr; Select **Create new Container Apps environment** and specify a name.
        * **Select a location for new resources** &rarr; Choose the same location that resource group you created previously.        

        It will take several moments to create the environment. A notification shows the progress of the operation. Look for "Successfully created new Container Apps environment" before going to the next step. The environment will be created in a resource group of the same name "python-container-env".
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-container-app-02.gif" alt-text="Screenshot showing how to create an environment for Azure Container Apps in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-container-app-02.gif":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 4.** After the environment is created, create a container app in it by finding the **Azure Container Apps: Create Container App** task in the command palette.

        * **Select a resource** &rarr; Select the environment created above.
        * **Enter a name for the new container app** &rarr; Enter *python-container-app*.
        * **Select a container registry** &rarr; Select **Azure Container Registries**.
        * **Select an Azure Container Registry** &rarr; Select the name of the registry you create previously.
        * **Select a repository** &rarr; Select **pythoncontainer**.
        * **Select a tag** &rarr; Select **latest**.
        * **Set with environment variables file** &rarr; Select the *.env* file you created above.
        * **Enable ingress for applications** &rarr; Select **Enable**.
        * **Select the HTTP traffic that the endpoint will accept** &rarr; Select **External**.
        * **Port the container is listening on** &rarr; Set to 8000 (Django) or 5000 (Flask).

        To start the container task, you can also go to the Azure extension, Container Apps section, select the environment, right-click and select **Create Container App**.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-container-app-03.gif" alt-text="Screenshot showing how to create an Azure Container app in an environment in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-container-app-03.gif":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 5.** For Django only, migrate and create database schema. (In the Flask sample app, it's done automatically, and you can skip this step.)

        * Go to the **Azure** extension, expand the **Container Apps** section, find and expand your container environment, and right-click the container your created and select **Open Console in Portal**.
        * Choose a startup command and select **Connect**.
        * At the shell prompt, type `python manage.py migrate`.

        You don't need to migrate for revisions of the container.
    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/azure-portal-create-container-app-11.png" alt-text="Screenshot showing how to connect to an Azure Container Apps container in Azure portal." lightbox="media/tutorial-container-apps/azure-portal-create-container-app-11.png":::
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="2":::
        **Step 6.** Test the website.

        * After the create container task completes, you'll see a notification with a **Browse** button to go to the website.

        If you miss the notification, go to the **Azure** extension, expand the **Container Apps** section, find and expand your container environment, and right-click the container app and select **Browse**.

    :::column-end:::
    :::column:::
        :::image type="content" source="media/tutorial-container-apps/visual-studio-code-create-container-app-04.png" alt-text="Screenshot showing how to browse to an Azure Container app after it is created in Visual Studio Code." lightbox="media/tutorial-container-apps/visual-studio-code-create-container-app-04.png":::
    :::column-end:::
:::row-end:::

### [Azure CLI](#tab/azure-cli)

:::row:::
    :::column:::
        **Step 1.** Sign in to Azure and authenticate, if needed.

        ```bash
        az login
        ```
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 2.** Install or upgrade the extension for Azure Container Apps withe [az extension add][14] command.
        
        ```bash
        az extension add --name containerapp --upgrade
        ```
        
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 3.** Create a Container Apps environment with the [az containerapp env create][13] command.

        ```bash
        az containerapp env create \
        --name python-container-env \
        --resource-group pythoncontainer-rg \
        --location <location>
        ```
        *\<location>* is one of the Azure location *Name* values from the output of the command `az account list-locations -o table`.

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 4.** Get the sign-in credentials for the Azure Container Registry.

        ```bash
        az acr credential show -n <registry-name>
        ```

        Use the username and one of the passwords returned from the output of the above command.

    :::column-end:::
:::row-end:::

:::row:::
    :::column span="1":::
        **Step 5.** Create a container app in the environment with the [az containerapp create][12] command.

        ```bash
        az containerapp create \
        --name python-container-app \
        --resource-group pythoncontainer-rg \
        --image <registry-name>.azurecr.io/pythoncontainer:latest \
        --environment python-container-env \
        --ingress external \
        --target-port 8000 \
        --registry-server <registry-name>.azurecr.io \
        --registry-username <registry-username> \
        --registry-password <registry-password> \
        --env-vars <env-variable-string>
        --query properties.configuration.ingress.fqdn
        ```

        *\<env-variable-string>* is a string composed of space-separated values in the key="value" format with the following values.

        * AZURE_POSTGRESQL_HOST=\<postgres-server-name>.postgres.database.azure.com
        * AZURE_POSTGRESQL_DATABASE=restaurants_reviews
        * AZURE_POSTGRESQL_USERNAME=demoadmin
        * AZURE_POSTGRESQL_PASSWORD=\<db-password>
        * RUNNING_IN_PRODUCTION=1

        Here's an example: `--env-vars AZURE_POSTGRESQL_HOST="my-postgres-server.postgres.database.azure.com" AZURE_POSTGRESQL_DATABASE="restaurants_reviews" AZURE_POSTGRESQL_USERNAME="demoadmin" AZURE_POSTGRESQL_PASSWORD="somepassword" RUNNING_IN_PRODUCTION="1"`.

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 7.** For Django only, migrate and create database schema. (In the Flask sample app, it's done automatically, and you can skip this step.)

        Connect with the [az containerapp exec][31] command:

        ```bash
            az containerapp exec \
                --name python-container-app \
                --resource-group pythoncontainer-rg
        ```

        Then, at the shell command prompt type `python manage.py migrate`.

        You don't need to migrate for revisions of the container.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        **Step 8.** Test the website.

        The create command above outputs an application Url you can use to browse to. The Url ends in "azurecontainerapps.io". Navigate to the Url.

    :::column-end:::
:::row-end:::

---

Here's an example of the sample website after adding a restaurant and two reviews.

:::image type="content" source="media/tutorial-container-apps/final-website-example-400px.png" alt-text="Screenshot showing an example of the sample website built in this tutorial." lightbox="media/tutorial-container-apps/final-website-example.png":::

## Troubleshoot deployment

* You forgot the Application Url to access the website.
  * In the Azure portal, go to the **Overview** page of the Container App and look for the **Application Url**.
  * In VS Code, go to the Azure extension and select the **Container Apps** section. Expand the subscription, expand the container environment, and when you find the container app, right-click **python-container-app** and select **Browse**.
  * With Azure CLI, use the command `az containerapp show -g pythoncontainer-rg -n python-container-app --query properties.configuration.ingress.fqdn`.

* In VS Code, the **Build Image in Azure** task returns an error.
  * If you see the message "Error: failed to download context. Please check if the URL is incorrect." in the VS Code **Output** window, then refresh the registry in the Docker extension. To refresh, select the Docker extension, go to the Registries section, find the registry and select it.
  * If you run the **Build Image in Azure** task again, check to see if your registry from a previous run exists and if so, use it.

* In the Azure portal during the creation of a Container App, you see an access error that contains "Cannot access ACR '\<name>.azurecr.io'".
  * This error occurs when admin credentials on the ACR are disabled. To check admin status in the portal, go to your Azure Container Registry, select the **Access keys** resource, and ensure that **Admin user** is enabled.

* Your container image doesn't appear in the Azure Container Registry.
  * Check the output of the Azure CLI command or VS Code Output and look for messages to confirm success.
  * Check that the name of the registry was specified correctly in your build command with the Azure CLI or in the VS Code task prompts.
  * Make sure your credentials haven't expired. For example, in VS Code, find the target registry in the Docker extension and refresh. In Azure CLI, run `az login`.

* Website returns "Bad Request (400)".
  * Check the PostgreSQL environment variables passed in to the container. The 400 error often indicates that the Python code can't connect to the PostgreSQL instance.
  * The sample code used in this tutorial checks for the existence of  the container environment variable `RUNNING_IN_PRODUCTION`, which can be set to any value like "1".

* Website returns "Not Found (404)".
  * Check the **Application Url** on the **Overview** page for the container. If the Application Url contains the word "internal", then ingress isn't set correctly.
  * Check the ingress of the container. For example, in Azure portal, go to the **Ingress** resource of the container and make sure **HTTP Ingress** is enabled and **Accepting traffic from anywhere** is selected.

* Website doesn't start, you see "stream timeout", or nothing is returned.
  * Check the logs.
    * In the Azure portal, go to the Container App's Revision management resource and check the **Provision Status** of the container.
      * If "Provisioning", then wait until provisioning has completed.
      * If "Failed", then select the revision and view the console logs. Choose the order of the columns to show "Time Generated", "Stream_s", and "Log_s". Sort the logs by most-recent first and look for Python *stderr* and *stdout* messages in the "Stream_s" column. Python 'print' output will be *stdout* messages.
    * With the Azure CLI, use the [az containerapp logs show][32] command.
  * If using the Django framework, check to see if the *restaurants_reviews* tables exist in the database. If not, use a console to access the container and run `python manage.py migrate`.

## Next step

> [!div class="nextstepaction"]
> [Configure continuous deployment](tutorial-deploy-python-web-app-azure-container-apps-03.md)

[1]: https://github.com/Azure-Samples/msdocs-python-django-azure-container-apps
[2]: https://github.com/Azure-Samples/msdocs-python-flask-azure-container-apps
[3]: https://portal.azure.com/
[4]: https://shell.azure.com/
[5]: /cli/azure/acr#az-acr-build
[6]: https://code.visualstudio.com/docs/containers/overview
[7]: /cli/azure/install-azure-cli
[8]: /azure/container-apps/overview
[9]: /azure/service-connector/overview
[10]: /azure/postgresql/flexible-server/overview
[11]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurecontainerapps
[12]: /cli/azure/containerapp#az-containerapp-create
[13]: /cli/azure/containerapp/env#az-containerapp-env-create
[14]: /cli/azure/extension#az-extension-add
[15]: https://www.postgresql.org/docs/13/app-psql.html
[16]: /cli/azure/postgres/flexible-server#az-postgres-flexible-server-connect
[17]: /cli/azure/group#az-group-create
[18]: /cli/azure/acr#az-acr-create
[19]: /cli/azure/acr#az-acr-login
[20]: /cli/azure/acr/repository#az-acr-repository-list
[21]: https://git-scm.com/docs/git-clone
[22]: /cli/azure/postgres/flexible-server#az-postgres-flexible-server-create
[23]: https://www.whatsmyip.org/
[24]: https://azure.microsoft.com/pricing/details/postgresql/flexible-server/
[25]: https://techcommunity.microsoft.com/t5/itops-talk-blog/how-to-use-cloud-shell-in-visual-studio-code/ba-p/663431
[26]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb
[27]: /cli/azure/postgres/flexible-server/db#az-postgres-flexible-server-db-create
[28]: /cli/azure/postgres/flexible-server/firewall-rule#az-postgres-flexible-server-firewall-rule-create
[29]: /cli/azure/acr#az-acr-update
[30]: /azure/container-apps/environment
[31]: /azure/container-apps/revisions
[32]: /cli/azure/containerapp/logs#az-containerapp-logs-show
[33]: /cli/azure/containerapp#az-containerapp-exec
