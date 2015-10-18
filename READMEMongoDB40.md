#Canducci MongoDB Repository .NET 4.0


[![Canducci MongoDB Repository .NET 4.0](http://i666.photobucket.com/albums/vv25/netdragoon/nosql_zpsefi6szxd.png)](https://www.nuget.org/packages/Canducci.MongoDB.Repository4.0/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package Canducci.MongoDB.Repository4.0

```

##Como utilizar?

Crie em seu Web.config em `appSettings` duas configurações (MongoConnectionString e MongoDatabase):

```Csharp

<appSettings>
    <add key="MongoConnectionString" value="mongodb://localhost" />
    <add key="MongoDatabase" value="test" />

```

essas configurações são responsável a conexão da camada ___Repository___.

Faça uma classe que representa a sua Collection no MongoDB

```Csharp

using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;
using System.ComponentModel.DataAnnotations;
using Canducci.MongoDB.Repository.Attributes;

namespace Br.Mongo.Web.Models
{
    [Serializable()]
    [MongoCollectionName("people")]
    public class People
    {
        public People()
        {
            this.Name = "";
            this.Birthday = null;
        }
        public People(string Name, DateTime? Birthday)
        {
            this.Name = Name;
            this.Birthday = Birthday;
        }
        public People(ObjectId Id, string Name, DateTime? Birthday)
        {
            this.Id = Id;
            this.Name = Name;
            this.Birthday = Birthday;
        }

        [BsonId()]
        public ObjectId Id { get; set; }

        [BsonRequired()]
        public string Name { get; set; }

        [DisplayFormat(ApplyFormatInEditMode = true, DataFormatString = "{0:dd/MM/yyyy}", ConvertEmptyStringToNull = true)]
        public DateTime? Birthday { get; set; }
    }
}

```
Observação: tem um atributo MongoCollectionName que possui a configuração do nome da sua coleção no mongo.

Próximo passo será a criação dos Repository.

Crie um `class` e declare esses dois `namespace`

```Csharp
using Canducci.MongoDB.Repository.Connection;
using Canducci.MongoDB.Repository.Contracts;

```

Em sua codificação:

```Csharp

//class abstract
public abstract class RepositoryPeopleContract : Repository<People>, IRepository<People>
{
    public RepositoryPeopleContract(IConnect Connect)
        : base(Connect) { }
}

//class Concret
public class RepositoryPeople : RepositoryPeopleContract
{
    public RepositoryPeople(IConnect Connect)
        : base(Connect) { }
}


```