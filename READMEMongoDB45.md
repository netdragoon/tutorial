#Canducci MongoDB Repository .NET 4.5

###ASP.NET MVC 4.5

[![Canducci MongoDB Repository .NET 4.5](http://i666.photobucket.com/albums/vv25/netdragoon/nosql_zpsefi6szxd.png)](https://www.nuget.org/packages/Canducci.MongoDB.Repository4.5/)

[![NuGet](https://img.shields.io/nuget/dt/Canducci.MongoDB.Repository4.5.svg?style=plastic&label=downloads)](https://www.nuget.org/packages/Canducci.MongoDB.Repository4.5/)
[![NuGet](https://img.shields.io/nuget/v/Canducci.MongoDB.Repository4.5.svg?style=plastic&label=version)](https://www.nuget.org/packages/Canducci.MongoDB.Repository4.5/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package Canducci.MongoDB.Repository4.5

```

##Como utilizar?

Crie em seu Web.config em `appSettings` duas configurações (MongoConnectionString e MongoDatabase):

```Csharp

<appSettings>
    <add key="MongoConnectionString" value="mongodb://localhost" />
    <add key="MongoDatabase" value="test" />

```

essas configurações são responsável a conexão da camada ___Repository___.

___Faça uma classe que representa a sua Collection no MongoDB___

```Csharp

using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using MongoDB.Driver;
using MongoDB.Bson;
namespace Br.Mongo.Web4._5.Models
{
    [Canducci.MongoDB.Repository.Attributes.MongoCollectionName("estudos")]    
    public class Estudos
    {

        public Estudos()
        {

        }

        [MongoDB.Bson.Serialization.Attributes.BsonRequired()]
        [MongoDB.Bson.Serialization.Attributes.BsonId()]
        public ObjectId Id { get; set; }

        [MongoDB.Bson.Serialization.Attributes.BsonElement("description")]
        [MongoDB.Bson.Serialization.Attributes.BsonRequired()]
        public string Description { get; set; }


        [MongoDB.Bson.Serialization.Attributes.BsonRequired()]
        [MongoDB.Bson.Serialization.Attributes.BsonElement("value")]
        public decimal Value { get; set; }

        [MongoDB.Bson.Serialization.Attributes.BsonRequired()]        
        [MongoDB.Bson.Serialization.Attributes.BsonElement("datetimecreated")]
        public DateTime DateCreated { get; set; }

        [MongoDB.Bson.Serialization.Attributes.BsonRequired()]
        [MongoDB.Bson.Serialization.Attributes.BsonElement("active")]
        public bool Active { get; set; }
    }
}

```
___Observação:___ tem um atributo `MongoCollectionName` que possui a configuração do nome da sua coleção no mongo, se por acaso não passar ele pega o nome da classe.

___Próximo passo será a criação do `Repository`.___

Crie um `class` e declare esses dois `namespace`

```Csharp
using Canducci.MongoDB.Repository.Contracts;
using Canducci.MongoDB.Repository.Connection;

```

___Em sua codificação:___

```Csharp

//class abstract
public abstract class RepositoryEstudosContract: Repository<Estudos>, IRepository<Estudos>
{
    public RepositoryEstudosContract(IConnect Connect)
        :base(Connect) { }
}

//class Concret
public sealed class RepositoryEstudos : RepositoryEstudosContract
{
    public RepositoryEstudos(IConnect Connect)
        : base(Connect) { }
}


```

__Solução básica com Unity__

___Configurando Unity___

```Csharp

using System;
using Microsoft.Practices.Unity;
using Microsoft.Practices.Unity.Configuration;
using Canducci.MongoDB.Repository.Connection;
using Br.Mongo.Web4._5.Models;

namespace Br.Mongo.Web4._5.App_Start
{    
    public class UnityConfig
    {
        
        private static Lazy<IUnityContainer> container = new Lazy<IUnityContainer>(() =>
        {
            var container = new UnityContainer();
            RegisterTypes(container);
            return container;
        });
        
        public static IUnityContainer GetConfiguredContainer()
        {
            return container.Value;
        }
        
        public static void RegisterTypes(IUnityContainer container)
        {
            // NOTE: To load from web.config uncomment the line below. Make sure to add a Microsoft.Practices.Unity.Configuration to the using statements.
            // container.LoadConfiguration();

            // TODO: Register your types here
            // container.RegisterType<IProductRepository, ProductRepository>();

            // Configurações do UnityContainer

            container.RegisterType<IConnect, Connect>();

            container.RegisterType<RepositoryEstudosContract, RepositoryEstudos>();
        }
    }
}

```

__Utilizando no `Controller`__

```Csharp

using Br.Mongo.Web4._5.Models;
using Canducci.MongoDB.Repository.Connection;
using MongoDB.Bson;
using MongoDB.Driver;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Web;
using System.Web.Mvc;

namespace Br.Mongo.Web4._5.Controllers
{
    public class ProvaController : Controller
    {
        protected RepositoryEstudosContract Repository;
        public ProvaController(RepositoryEstudosContract Repository)
        {
            this.Repository = Repository;            
        }
        protected override void Dispose(bool disposing)
        {
            if (this.Repository != null) this.Repository.Dispose();
            base.Dispose(disposing);
        }
        public async Task<ActionResult> Index(int? Page, string Pesquisa)
        {
            var Sort = Builders<Estudos>.Sort.Ascending(x => x.Description);
            if (string.IsNullOrEmpty(Pesquisa)) {
                return View(await Repository.Pagination(Page ?? 1, 10, Sort));
            }
            return View(await Repository.Pagination(Page ?? 1,10, Builders<Estudos>
                .Filter
                .Regex(x => x.Description, BsonRegularExpression.Create(string.Format("/{0}/i", Pesquisa))), Sort));
        }
        
        public ActionResult Create()
        {            
            return View();
        }

        // POST: Prova/Create
        [HttpPost]
        public async Task<ActionResult> Create(Estudos estudo)
        {
            try
            {
                await Repository.Add(estudo);
                return RedirectToAction("Index");
            }
            catch
            {
                return View();
            }
        }

        // GET: Prova/Edit/5
        public async Task<ActionResult> Edit(string id)
        {
            ObjectId Id = Repository.CreateObjectId(id);
            return View(await Repository.Find(x => x.Id.Equals(Id)));
        }

        // POST: Prova/Edit/5
        [HttpPost]
        public async Task<ActionResult> Edit(string id, Estudos estudo)
        {
            try
            {                
                estudo.Id = Repository.CreateObjectId(id);
                ReplaceOneResult edit = await Repository.Edit(Builders<Estudos>.Filter.Eq<ObjectId>(x => x.Id, estudo.Id), estudo);
                return RedirectToAction("Index");
            }
            catch
            {
                return View();
            }
        }

        // GET: Prova/Delete/5
        public async Task<ActionResult> Delete(string id)
        {
            try
            {
                ObjectId Id = Repository.CreateObjectId(id);
                DeleteResult delete = await Repository.Delete(x => x.Id.Equals(Id));                
                return RedirectToAction("Index");
            }
            catch
            {
                return View();
            }
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
using System.Linq.Expressions;
using System.Threading.Tasks;

namespace Canducci.MongoDB.Repository.Contracts
{    
    public interface IRepository<T>: IDisposable
        where T : class, new()
    {       
        IMongoCollection<T> Collection { get; }
        string CollectionName { get; }
        Task<T> Add(T Model);
        Task<ReplaceOneResult> Edit(Expression<Func<T, bool>> Query, T Model);
        Task<ReplaceOneResult> Edit(FilterDefinition<T> Query, T Model);
        Task<IEnumerable<T>> Add(IEnumerable<T> Models);
        Task<UpdateResult> Update(Expression<Func<T, bool>> Query, UpdateDefinition<T> Update);
        Task<UpdateResult> Update(FilterDefinition<T> Query, UpdateDefinition<T> Update);
        Task<T> Find(FilterDefinition<T> Query);
        Task<T> Find(Expression<Func<T, bool>> Query);
        Task<IEnumerable<T>> All(Expression<Func<T, bool>> Query, SortDefinition<T> SortBy = null);
        Task<IEnumerable<T>> All(FilterDefinition<T> Query, SortDefinition<T> SortBy = null);
        Task<IEnumerable<T>> All(SortDefinition<T> SortBy);
        Task<IEnumerable<T>> All();
        Task<DeleteResult> Delete(Expression<Func<T, bool>> Query);
        Task<DeleteResult> Delete(FilterDefinition<T> Query);
        T Create();
        ObjectId CreateObjectId(string Value);
        Task<PagedList.StaticPagedList<T>> Pagination(int Page, int Total, SortDefinition<T> SortBy, Expression<Func<T, bool>> Query = null);
        Task<PagedList.StaticPagedList<T>> Pagination(int Page, int Total, FilterDefinition<T> Query, SortDefinition<T> SortBy);
    }
}

```