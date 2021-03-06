# RepositoryHelpers

Extensions for HttpClient and Custom Repository based on dapper

###### This is the component, works on .NET Core and.NET Framework

**Info**

|Code Quality|Build|Nuget|
| ------------------- | ------------------- | :------------------: |
|[![Codacy Badge](https://api.codacy.com/project/badge/Grade/ea9b954b18e942d4800825dccd6ef77c)](https://app.codacy.com/app/TBertuzzi/RepositoryHelpers?utm_source=github.com&utm_medium=referral&utm_content=TBertuzzi/RepositoryHelpers&utm_campaign=Badge_Grade_Dashboard)|[![Build status](https://ci.appveyor.com/api/projects/status/github/TBertuzzi/RepositoryHelpers?branch=master&svg=true)](https://ci.appveyor.com/project/ThiagoBertuzzi/repositoryhelpers)|[![NuGet](https://buildstats.info/nuget/RepositoryHelpers)](https://www.nuget.org/packages/RepositoryHelpers/)|

**Build History**

[![Build history](https://buildstats.info/appveyor/chart/ThiagoBertuzzi/repositoryhelpers?buildCount=7)](https://ci.appveyor.com/project/ThiagoBertuzzi/repositoryhelpers/history)

**Platform Support**

RepositoryHelpers is a .NET Standard 2.0 library.

**Database/Dapper Extensions**

Use the connection class to define the type of database and connection string

```csharp
 var connection = new Connection()
 {
     Database = RepositoryHelpers.Utils.DataBaseType.SqlServer, //RepositoryHelpers.Utils.DataBaseType.Oracle
     ConnectionString = "Your string"
 };
```

Create a CustomRepository of the type of object you want to return

```csharp
  var Repository = new CustomRepository<User>(conecction);
``````

Attributes:

```csharp
[DapperIgnore]
public string InternalControl { get; set; }
[PrimaryKey]
public int MyCustomId { get; set; }
[PrimaryKey]
[IdentityIgnore]
public int MyBdIdIndentity { get; set; }

``````
Get Data:

To get results just use the Get method. can be syncronous or asynchronous

```csharp
//Get All Users
var usersAsync = await Repository.GetAsync();
var users = Repository.Get();

//Get User by Id
var userAsync = await Repository.GetByIdAsync(1);
var user = Repository.GetById(1);

//Get by CustomQuery
var customQuery = "Select name from user where login = @userLogin";
var parameters = new Dictionary<string, object> { { "userLogin", "bertuzzi" } };
var resultASync = await Repository.GetAsync(customQuery, parameters);
var result = Repository.Get(customQuery, parameters);
```

Insert Data :

user identity parameter to return the id if your insert needs

```csharp
Repository.Insert(NewUser, true);
```

Update data

```csharp
Repository.Update(updateUser);
Repository.UpdateAsync(updateUser);
```

Delete data

```csharp
Repository.Delete(1);
Repository.DeleteAsync(1);
```

You can use ADO if you need

```csharp
//Return DataSet
var customQuery = "Select name from user where login = @userLogin";
var parameters = new Dictionary<string, object> { { "userLogin", "bertuzzi" } };
Repository.GetDataSet(customQuery, parameters);

//ExecuteQuery
Repository.ExecuteQueryAsync();
Repository.ExecuteQuery();

//ExecuteScalar
Repository.ExecuteScalarAsync();
Repository.ExecuteScalar();

//ExecuteProcedure
Repository.ExecuteProcedureAsync();
Repository.ExecuteProcedure();
```

CustomTransaction is possible to use transaction

```csharp

CustomTransaction customTransaction = new CustomTransaction(YourConnection);

customTransaction.BeginTransaction();
customTransaction.CommitTransaction();
customTransaction.RollbackTransaction();

//Sample
Repository.ExecuteQuery("yourquery", parameters, customTransaction);


```


DapperIgnore : if you want some property of your object to be ignored by Dapper, when inserting or updating, just use the attribute.
PrimaryKey : Define your primary key. It is used for queries, updates, and deletes.
IdentityIgnore: Determines that the field has identity, autoincrement ... Warns the repository to ignore it that the database will manage the field

*TIP Create a ConnectionHelper for BaseRepository and BaseTransaction to declare the connection only once :

```csharp

 public sealed class ConnectionHelper
    {
        static ConnectionHelper _instance;
        public static ConnectionHelper Instance
        {
            get { return _instance ?? (_instance = new ConnectionHelper()); }
        }
        private ConnectionHelper() 
        {
            Connection = new Connection()
            {
                Database = RepositoryHelpers.Utils.DataBaseType.SqlServer,
                ConnectionString = "YourString"
            };
        }
        public Connection Connection { get; }
    }
    
 public class BaseRepository<T>
    {
        protected readonly CustomRepository<T> Repository;

        protected BaseRepository()
        {
            Repository = new CustomRepository<T>(ConnectionHelper.Instance.Connection);
        }
    }
    
     public class BaseTransaction : CustomTransaction
    {
        public BaseTransaction() :
             base(ConnectionHelper.Instance.Connection)
        {
           
        }
    }
    
```

**LiteDB Extensions**

coming soon ..

**HttpClient Extensions**

Extensions to make using HttpClient easy.

To enable :

```csharp
using HttpExtension;
```

* GetAsync<T> : Gets the return of a Get Rest and converts to the object or collection of pre-defined objects.
You can use only the path of the rest method, or pass a parameter dictionary. In case the url has parameters.

```csharp
 public static async Task<ServiceResponse<T>> GetAsync<T>(this HttpClient httpClient, string address);
 public static async Task<ServiceResponse<T>> GetAsync<T>(this HttpClient httpClient, string address,
        Dictionary<string, string> values);
```


* PostAsync<T>,PutAsync<T> and DeleteAsync<T> : Use post, put and delete service methods rest asynchronously and return objects if necessary. 
 
```csharp
 public static async Task<HttpResponseMessage> PostAsync(this HttpClient httpClient,string address, object dto);
 public static async Task<ServiceResponse<T>> PostAsync<T>(this HttpClient httpClient, string address, object dto);
 
 public static async Task<HttpResponseMessage> PutAsync(this HttpClient httpClient,string address, object dto);
 public static async Task<ServiceResponse<T>> PutAsync<T>(this HttpClient httpClient, string address, object dto);
 
 public static async Task<HttpResponseMessage> DeleteAsync(this HttpClient httpClient,string address, object dto);
 public static async Task<ServiceResponse<T>> DeleteAsync<T>(this HttpClient httpClient, string address, object dto);
```

* ServiceResponse<T> : Object that facilitates the return of requests Rest. It returns the Http code of the request, already converted object and the contents in case of errors.

```csharp
public class ServiceResponse<T>
{
  public HttpStatusCode StatusCode { get; private set; }

  public T Value { get; set; }

  public string Content { get; set; }

  public Exception Error { get; set; }
}
```

Example of use :

```csharp
public async Task<List<Model.Todo>> GetTodos()
 {
    try
    {

        //GetAsync Return with Object
        var response = await _httpClient.GetAsync<List<Model.Todo>>("todos");
           
        if (response.StatusCode == HttpStatusCode.OK)
        {
              return response.Value;
        }
        else
        {
            throw new Exception(
                   $"HttpStatusCode: {response.StatusCode.ToString()} Message: {response.Content}");
        }
    }
    catch (Exception ex)
    {
        throw new Exception(ex.Message);
    }
 }
```

Samples coming soon ..

Thanks users who reported bugs and helped improve the package :

* Thiago Vieira
* Luis Paulo Souza
* Alexandre Harich
