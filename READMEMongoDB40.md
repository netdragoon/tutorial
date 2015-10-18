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
___Observação:___ tem um atributo `MongoCollectionName` que possui a configuração do nome da sua coleção no mongo, se por acaso não passar ele pega o nome da classe.

___Próximo passo será a criação do `Repository`.___

Crie um `class` e declare esses dois `namespace`

```Csharp
using Canducci.MongoDB.Repository.Connection;
using Canducci.MongoDB.Repository.Contracts;

```

___Em sua codificação:___

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


__Solução básica com Unity__

___Configurando Unity___

```Csharp

using Br.Mongo.Web.Models;
using Canducci.MongoDB.Repository.Connection;
using Microsoft.Practices.Unity;
using System.Web.Mvc;
using Unity.Mvc4;
namespace Br.Mongo.Web
{
  public static class Bootstrapper
  {
    public static IUnityContainer Initialise()
    {
        var container = BuildUnityContainer();
        DependencyResolver.SetResolver(new UnityDependencyResolver(container));
        return container;
    }

    private static IUnityContainer BuildUnityContainer()
    {
        var container = new UnityContainer();   
        RegisterTypes(container);
        return container;
    }
    public static void RegisterTypes(IUnityContainer container)
    {
    	//Configuração das Classes no UnityContainer

        container.RegisterType<IConnect, Connect>();

        container.RegisterType<RepositoryPeopleContract, RepositoryPeople>();

    }
  }
}

```

__Utilizando no `Controller`__

```Csharp

using Br.Mongo.Web.Models;
using Canducci.MongoDB;
using MongoDB.Bson;
using MongoDB.Driver;
using MongoDB.Driver.Builders;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text.RegularExpressions;
using System.Web.Mvc;

namespace Br.Mongo.Web.Controllers
{
    public class HomeController : Controller
    {   
        RepositoryEstudosContract Repository;
        public HomeController(RepositoryEstudosContract Repository)
        {
            this.Repository = Repository;
        }
        protected override void Dispose(bool disposing)
        {
            if (Repository != null)
            {
                Repository.Dispose();
            }
            base.Dispose(disposing);
        }

        //INDEX
        public ActionResult Index(int? Page, string Pesquisa)
        {
            ViewBag.Pesquisa = Pesquisa ?? "";
            SortByBuilder<People> SortByBuilderPeople = SortBy<People>.Ascending(x => x.Name);
            return string.IsNullOrEmpty(Pesquisa) ?
                View(Repository.Pagination(Page ?? 1, 10, SortByBuilderPeople)) :
                View(Repository.Pagination(Page ?? 1, 10, SortByBuilderPeople,
                    Query<People>.Matches(x => x.Name, new BsonRegularExpression(Pesquisa, "i"))));
        }

        //CREATE
        [HttpGet]
        public ActionResult Create()
        {
            return View();
        }
        [HttpPost]
        public ActionResult Create(Estudos estudo)
        {            
            Repository.Add(estudo);            
            return RedirectToAction("Index");
        }

        //EDIT
        [HttpGet]
        public ActionResult Edit(string Id)
        {            
            return View(Repository.Find(Repository.CreateObjectId(Id)));
        }
        [HttpPost]
        public ActionResult Edit(Estudos estudo, string Id)
        {
            estudo.Id = Repository.CreateObjectId(Id);
            Repository.Edit(estudo);
            return RedirectToAction("Index");
        }

        
        //DELETE
        [HttpGet]
        public ActionResult Delete(string Id)
        {

            return View(Repository.Delete(Repository.CreateObjectId(Id)));
        }

    }
}

```

Aqui estão todos os métodos desse `Repository`


```Csharp

using MongoDB.Bson;
using MongoDB.Driver;
using System;
using System.Collections.Generic;
using System.Linq;

namespace Canducci.MongoDB.Repository.Contracts
{    
    public interface IRepository<T>: IDisposable
        where T : class, new()
    {
        MongoCollection<T> Collection { get; }
        string CollectionName { get; }
        T Add(T Model);
        IEnumerable<T> Add(IEnumerable<T> Models);
        bool Edit(T Model);
        bool Update(IMongoQuery Query, IMongoUpdate Update);
        T Find(ObjectId Id);
        T Find(IMongoQuery Query);
        IEnumerable<T> All(IMongoQuery Query, IMongoSortBy SortBy = null);
        IEnumerable<T> All(IMongoSortBy SortBy);
        IEnumerable<T> All();
        bool Delete(IMongoQuery Query);
        bool Delete(ObjectId Id, string ObjectIdName = "Id");
        IQueryable<T> Linq();
        bool Exists();
        MongoCursor<T> Cursor(IMongoQuery Query);
        MongoCursor<T> Cursor();
        T Create();
        ObjectId CreateObjectId(string Value);
        PagedList.StaticPagedList<T> Pagination(int Page, int Total, IMongoSortBy SortBy, IMongoQuery Query = null);
    }
}

```