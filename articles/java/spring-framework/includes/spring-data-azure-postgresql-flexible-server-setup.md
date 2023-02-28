---
ms.date: 02/22/2023
author: KarlErickson
ms.author: v-yonghuiye
---

## Configure a firewall rule for your PostgreSQL server

Azure Database for PostgreSQL instances are secured by default. They have a firewall that doesn't allow any incoming connection.

To be able to use your database, open the server's firewall to allow the local IP address to access the database server. For more information, see [Firewall rules in Azure Database for PostgreSQL - Flexible Server](/azure/postgresql/flexible-server/concepts-firewall-rules).

If you're connecting to your PostgreSQL server from Windows Subsystem for Linux (WSL) on a Windows computer, you need to add the WSL host ID to your firewall.

## Create a PostgreSQL non-admin user and grant permission

Next, create a non-admin user and grant all permissions to the database.

### [Passwordless (Recommended)](#tab/passwordless)

> [!IMPORTANT]
> To use passwordless connections, configure the Azure AD admin user for your Azure Database for PostgreSQL Flexible Server instance. For more information, see [Manage Azure Active Directory roles in Azure Database for PostgreSQL - Flexible Server](/azure/postgresql/flexible-server/how-to-manage-azure-ad-users).

Create a SQL script called *create_ad_user.sql* for creating a non-admin user. Add the following contents and save it locally:

```bash
cat << EOF > create_ad_user.sql
select * from pgaadauth_create_principal('<your_postgresql_ad_non_admin_username>', false, false);
EOF
```

Then, use the following command to run the SQL script to create the Azure AD non-admin user:

```bash
psql "host=postgresqlflexibletest.postgres.database.azure.com user=<your_postgresql_ad_admin_username> dbname=postgres port=5432 password=$(az account get-access-token --resource-type oss-rdbms --output tsv --query accessToken) sslmode=require" < create_ad_user.sql
```

> [!TIP]
> to se Azure AD authentication to connect to Azure Database for PostgreSQL, you need to sign in with the Azure AD admin user you set up, and then get the access token as the password. For more information, see [Use Azure AD for authentication with Azure Database for PostgreSQL - Flexible Server](/azure/postgresql/flexible-server/how-to-configure-sign-in-azure-ad-authentication).

### [Password](#tab/password)

Create a SQL script called *create_user.sql* for creating a non-admin user. Add the following contents and save it locally:

```bash
cat << EOF > create_user.sql
CREATE ROLE "<your_postgresql_non_admin_username>" WITH LOGIN PASSWORD '<your_postgresql_non_admin_password>';
GRANT ALL PRIVILEGES ON DATABASE demo TO "<your_postgresql_non_admin_username>";
EOF
```

Then, use the following command to run the SQL script to create the Azure AD non-admin user:

```bash
psql "host=postgresqlflexibletest.postgres.database.azure.com user=<your_postgresql_admin_username> dbname=demo port=5432 password=<your_postgresql_admin_password> sslmode=require" < create_user.sql
```

> [!NOTE]
> For more information, see [Create users in Azure Database for PostgreSQL - Flexible Server](/azure/PostgreSQL/flexible-server/how-to-create-users).

---

## Store data from Azure Database for PostgreSQL

Now that you have an Azure Database for PostgreSQL Flexible Server instance, you can store data by using Spring Cloud Azure.

To install the Spring Cloud Azure Starter JDBC PostgreSQL module, add the following dependencies to your *pom.xml* file:

- The Spring Cloud Azure Bill of Materials (BOM):

  ```xml
   <dependencyManagement>
     <dependencies>
       <dependency>
         <groupId>com.azure.spring</groupId>
         <artifactId>spring-cloud-azure-dependencies</artifactId>
         <version>4.5.0</version>
         <type>pom</type>
         <scope>import</scope>
         </dependency>
     </dependencies>
   </dependencyManagement>
  ```

- The Spring Cloud Azure Starter JDBC PostgreSQL artifact:

  ```xml
  <dependency>
    <groupId>com.azure.spring</groupId>
    <artifactId>spring-cloud-azure-starter-jdbc-postgresql</artifactId>
  </dependency>
  ```

> [!NOTE]
> Passwordless connections have been supported since version `4.5.0`.
