#Canducci MongoDB Repository .NET 4.5

###ASP.NET MVC 4.5

[![Canducci MongoDB Repository .NET 4.5](http://i666.photobucket.com/albums/vv25/netdragoon/nosql_zpsefi6szxd.png)](https://www.nuget.org/packages/Canducci.MongoDB.Repository4.5/)

[![NuGet](https://img.shields.io/nuget/v/Canducci.MongoDB.Repository4.5.svg?style=plastic&label=version)](https://www.nuget.org/packages/Canducci.MongoDB.Repository4.5/)

___[Version Old 1.0.0](https://github.com/netdragoon/tutorial/blob/master/READMEMongoDB45old.md)___

##Install Package (NUGET)

To install Canducci MongoDB Repository .NET 4.5 e 4.5.1, run the following command in the [Package Manager Console](http://docs.nuget.org/consume/package-manager-console)

```Csharp

PM> Install-Package Canducci.MongoDB.Repository4.5

```

##How to use?

Create in your web.config in ` appSettings` two configurations ( MongoConnectionString and MongoDatabase ) :

```Csharp

<appSettings>
    <add key="MongoConnectionString" value="mongodb://localhost" />
    <add key="MongoDatabase" value="test" />

```

these settings are responsible for the connection layer ___Repository___.

___Make a class that represents your Collection in MongoDB___

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
___Obs:___ It has a ` MongoCollectionName` attribute that has the configuration of the name of your collection in mongo , if by chance not pass he takes the class name.

___Next step will be the creation of `Repository`.___

Create a ` class` and declare these two ` namespace`

```Csharp
using Canducci.MongoDB.Repository.Contracts;
using Canducci.MongoDB.Repository.Connection;

```

___Codification:___

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

__Basic solution Unity__

___Setting Unity___

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

            container.RegisterType<IConnect, Connect>(new InjectionConstructor());

            container.RegisterType<RepositoryEstudosContract, RepositoryEstudos>();
        }
    }
}

```

__`Controller`__

```Csharp

using Br.Mongo.Web4._5.Models;
using MongoDB.Bson;
using MongoDB.Driver;
using System.Threading.Tasks;
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

            if (Repository != null) Repository.Dispose();
            base.Dispose(disposing);

        }
        public async Task<ActionResult> Index(int? Page, string Pesquisa)
        {           

            if (string.IsNullOrWhiteSpace(Pesquisa))
            {                
                return View(
                    await Repository
                    .PaginationAsync(Page ?? 1, 10, x => x.Description));
            }
            return View(
                await Repository
                    .PaginationAsync(Page ?? 1, 10, x => x.Description, 
                        x => x.Description.Contains(Pesquisa)));

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
                await Repository.AddAsync(estudo);
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
            return View(await Repository.FindAsync(Id));            
        }

        // POST: Prova/Edit/5
        [HttpPost]
        public async Task<ActionResult> Edit(string id, Estudos estudo)
        {
            try
            {                   
                estudo.Id = Repository.CreateObjectId(id);                                
                ReplaceOneResult edit = 
                    await Repository.EditAsync(a => a.Id.Equals(estudo.Id), estudo);
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
                DeleteResult delete = 
                    await Repository.DeleteAsync(x => x.Id.Equals(Id));
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

Interface `IRepository` and his methods:


```Csharp

using Canducci.MongoDB.Repository.Connection;
using MongoDB.Bson;
using MongoDB.Driver;
using MongoDB.Driver.Linq;
using PagedList;
using System;
using System.Collections.Generic;
using System.Linq.Expressions;
using System.Threading.Tasks;
namespace Canducci.MongoDB.Repository.Contracts
{    
    public interface IRepository<T>: IDisposable
        where T : class
    {
        #region Methods
        IMongoCollection<T> MongoCollection();
        IConnect MongoConnect();
        #endregion Methods

        #region CRUD
        Task<T> AddAsync(T Model);
        Task<T> AddAsync(T Model, System.Threading.CancellationToken CancellationToken);
        Task<IEnumerable<T>> AddAsync(IEnumerable<T> Models);
        Task<IEnumerable<T>> AddAsync(IEnumerable<T> Models, InsertManyOptions Options);
        Task<IEnumerable<T>> AddAsync(IEnumerable<T> Models, InsertManyOptions Options, System.Threading.CancellationToken CancellationToken);

        Task<ReplaceOneResult> EditAsync(Expression<Func<T, bool>> Where, T Model);
        Task<ReplaceOneResult> EditAsync(FilterDefinition<T> Query, T Model);
        Task<ReplaceOneResult> EditAsync(Expression<Func<T, bool>> Where, T Model, UpdateOptions Options);
        Task<ReplaceOneResult> EditAsync(Expression<Func<T, bool>> Where, T Model, UpdateOptions Options, System.Threading.CancellationToken CancellationToken);

        Task<UpdateResult> UpdateAsync(Expression<Func<T, bool>> Where, UpdateDefinition<T> Update);
        Task<UpdateResult> UpdateAsync(Expression<Func<T, bool>> Where, UpdateDefinition<T> Update, UpdateOptions Options);
        Task<UpdateResult> UpdateAsync(Expression<Func<T, bool>> Where, UpdateDefinition<T> Update, UpdateOptions Options, System.Threading.CancellationToken CancellationToken);
        Task<UpdateResult> UpdateAsync(FilterDefinition<T> Query, UpdateDefinition<T> Update);

        Task<DeleteResult> DeleteAsync(Expression<Func<T, bool>> Where);
        Task<DeleteResult> DeleteAsync(Expression<Func<T, bool>> Query, System.Threading.CancellationToken CancellationToken);
        Task<DeleteResult> DeleteAsync(FilterDefinition<T> Query);
        Task<DeleteResult> DeleteAsync(FilterDefinition<T> Query, System.Threading.CancellationToken CancellationToken);

        #endregion CRUD

        #region Find
        Task<T> FindAsync(ObjectId Id);
        Task<T> FindAsync<TKey>(TKey Id, string Name = "_id");
        Task<T> FindAsync(FilterDefinition<T> Query);
        Task<T> FindAsync(Expression<Func<T, bool>> Query);
        Task<IAsyncCursor<T>> FindAsync(FilterDefinition<T> Query, FindOptions<T, T> Options = null, System.Threading.CancellationToken CancellationToken = default(System.Threading.CancellationToken));
        Task<IAsyncCursor<T>> FindAsync(Expression<Func<T, bool>> Query, FindOptions<T, T> Options = null, System.Threading.CancellationToken CancellationToken = default(System.Threading.CancellationToken));

        T Find(Expression<Func<T, bool>> Where);

        #endregion Find

        #region All
        Task<IEnumerable<T>> AllAsync();        
        Task<IEnumerable<T>> AllAsync(SortDefinition<T> SortBy);        
        Task<IEnumerable<T>> AllAsync(FilterDefinition<T> Query, SortDefinition<T> SortBy = null);
        Task<IEnumerable<T>> AllAsync(Expression<Func<T, bool>> Query, SortDefinition<T> SortBy = null);
        Task<IEnumerable<T>> AllAsync<TKey>(Expression<Func<T, bool>> Query, Expression<Func<T, TKey>> OrderBy);
        Task<IEnumerable<T>> AllAsync<TKey>(int Page, int Total, Expression<Func<T, bool>> Query, Expression<Func<T, TKey>> OrderBy);

        IList<T> All();
        IList<T> All<Tkey>(Expression<Func<T, Tkey>> OrderBy);
        IList<T> All<TKey>(Expression<Func<T, TKey>> OrderBy, Expression<Func<T, bool>> Where);
        IList<T> All<TKey>(int Page, int Total, Expression<Func<T, TKey>> OrderBy, Expression<Func<T, bool>> Where);

        #endregion All

        #region Count
        Task<long> CountAsync();
        Task<long> CountAsync(FilterDefinition<T> Query, CountOptions Options = null, System.Threading.CancellationToken CancellationToken = default(System.Threading.CancellationToken));
        Task<long> CountAsync(Expression<Func<T, bool>> Query, CountOptions Options = null, System.Threading.CancellationToken CancellationToken = default(System.Threading.CancellationToken));

        long Count();
        long Count(Expression<Func<T, bool>> Query);

        #endregion Count

        #region Create
        T Create();
        ObjectId CreateObjectId(string Value);

        #endregion Create

        #region StaticPagedList
        Task<StaticPagedList<T>> PaginationAsync<TKey>(int Page, int Total, Expression<Func<T, TKey>> OrderBy, Expression<Func<T, bool>> Where = null);
        Task<StaticPagedList<T>> PaginationAsync(int Page, int Total, SortDefinition<T> SortBy, Expression<Func<T, bool>> Where = null);
        Task<StaticPagedList<T>> PaginationAsync(int Page, int Total, SortDefinition<T> SortBy, FilterDefinition<T> Query);

        StaticPagedList<T> Pagination<TKey>(int Page, int Total, Expression<Func<T, TKey>> OrderBy, Expression<Func<T, bool>> Where = null);

        #endregion StaticPagedList

        #region AsQueryable
        IMongoQueryable<T> Query();
        IMongoQueryable<T> Query(params Expression<Func<T, bool>>[] Where);
        IMongoQueryable<T> Query<TKey>(Expression<Func<T, TKey>> OrderBy, params Expression<Func<T, bool>>[] Where);        
        #endregion AsQueryable

    }
}

```
