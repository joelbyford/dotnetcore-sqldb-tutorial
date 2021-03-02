---
languages:
- csharp
- aspx-csharp
page_type: sample
description: "This is a sample application you can use to follow along w/ the Build a .NET Core and SQL Database web app in Azure Web Apps for Containers tutorial."
products:
- azure
- aspnet-core
- azure-app-service
---

# .NET Core MVC sample for Azure App Service

This is a sample application that you can use to follow along with the tutorial at 
[Build a .NET Core and SQL Database web app in Azure Web Apps for Containers](https://docs.microsoft.com/azure/app-service/containers/tutorial-dotnetcore-sqldb-app). 

## License

See [LICENSE](LICENSE.md).

## Contributing

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
  

## Additional SQL Configuration for Managed Identities
1. Modify SQL Server Firewall
    - Add Web Server IP Address (or just enable Azure Services Access)
2. Enable Managed Identity on WebApp
    - Note the name of the WebApp
3. Grant the WebApp Access to to the database in the query editor (or SSMS)
```
CREATE USER [WebAppName] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [WebAppName];
ALTER ROLE db_datawriter ADD MEMBER [WebAppName];
```
4. Add packages to the project `dotnet add package`
    - `Azure.Identity`
    - `System.Data.SqlClient`
5. Update `startup.cs` in the `ConfigureServices` method
```
var connString = Configuration.GetConnectionString("MyDbConnection"); //Just need Server and Database/Initial Catalog in the conn string now
// Create a new connection
var sqlConn = new System.Data.SqlClient.SqlConnection(connString);
// Get the AD credential context for the application
var credential = new DefaultAzureCredential();
//get a token with the current app's context
var token = credential.GetToken(new Azure.Core.TokenRequestContext(new[] {"https://database.windows.net/.default"}));
// Add the token obtained to the connection to OAuth enable it
sqlConn.AccessToken = token.Token;

services.AddDbContext<MyDatabaseContext>(options =>
    options.UseSqlServer(sqlConn)); //pass the connection now, not the connection string
```
