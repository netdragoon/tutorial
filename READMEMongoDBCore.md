
#Canducci MongoDB Repository Core

###ASP.NET MVC Core

[![Canducci MongoDB Repository Core](http://i666.photobucket.com/albums/vv25/netdragoon/nosql_zpsefi6szxd.png)](https://www.nuget.org/packages/Canducci.MongoDB.Repository.Core/)

[![NuGet](https://img.shields.io/nuget/v/Canducci.MongoDB.Repository.Core.svg?style=plastic&label=version)](https://www.nuget.org/packages/Canducci.MongoDB.Repository.Core/)

##Install Package (NUGET)

To install Canducci MongoDB Repository Core, run the following command in the [Package Manager Console](http://docs.nuget.org/consume/package-manager-console)

```Csharp

PM> Install-Package Canducci.MongoDB.Repository.Core

```

##How to use?

Create in your `appsettings.json` a section:

```Csharp

"MongoDB": {
    "Database": "dbnew",
    "ConnectionStrings": "mongodb://localhost:27017"
}
```
In method configureServices do:
```
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<MongoConnectConfiguration>(Configuration.GetSection("MongoDB"));
    
```

these settings are responsible for the connection layer ___Repository___.

___Make a class that represents your Collection in MongoDB___

```
using System;
using Canducci.MongoDB.Repository.Core;
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

namespace WebApplication11.Models
{
    [MongoCollectionName("cars")]
    public class Carros
    {
        [BsonRequired()]
        [BsonId()]
        public ObjectId Id { get; set; }

        [BsonRequired()]
        [BsonElement("name")]
        public string Name { get; set; }
                
        [BsonElement("datecreated")]        
        public DateTime? DateCreated { get; set; }

        [BsonRequired()]
        [BsonElement("price")]
        public double Price { get; set; }
    }
}
```


___Obs:___ It has a ` MongoCollectionName` attribute that has the configuration of the name of your collection in mongo , if by chance not pass he takes the class name.

___Next step will be the creation of `Repository`.___

Create a ` class` and declare these one ` namespace`

```Csharp
using Canducci.MongoDB.Repository.Core;
```


___Codification:___

```Csharp
using Canducci.MongoDB.Repository.Core;
namespace WebApplication11.Models
{
    public abstract class RepositoryCarrosContract
        :Repository<Carros>, IRepository<Carros>
    {
        public RepositoryCarrosContract(IConnect connect)
            :base(connect)
        {
        }
    }
    public class RepositoryCarros
        : RepositoryCarrosContract
    {
        public RepositoryCarros(IConnect connect)
            : base(connect)
        {
        }
    }
}
```
