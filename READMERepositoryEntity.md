#Canducci EntityFramework Repository

[![Canducci.EntityFramework.Repository](http://i1194.photobucket.com/albums/aa377/netdragoon1/1452823324_icon-89-document-file-sql_zpslt5kmpu9.png)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)

[![NuGet](https://img.shields.io/nuget/dt/CanducciEntityFrameworkRepository.svg?style=plastic&label=downloads)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)
[![NuGet](https://img.shields.io/nuget/v/CanducciEntityFrameworkRepository.svg?style=plastic&label=version)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)

## NUGET

```Csharp

PM> Install-Package CanducciEntityFrameworkRepository

```

##HOW TO?

Use _namespace_ `using Canducci.EntityFramework.Repository.Contracts;`

The `BaseEFEntities`: `DbContext` and `Notice`: `DbSet`.

```Csharp

using Canducci.EntityFramework.Repository.Contracts;
namespace WebApplication1.Models
{
    
    //abstract class
    public abstract class RepositoryNoticeContract: Repository<Notice, BaseEFEntities>,
                                                    IRepository<Notice, BaseEFEntities>
    {
        public RepositoryNoticeContract(BaseEFEntities ctx)
            : base(ctx)
        {
        }
    }

    //class concrete
    public sealed class RepositoryNotice : RepositoryNoticeContract
    {
        public RepositoryNotice(BaseEFEntities ctx) 
            : base(ctx)
        {
        }
    }
}

```

____

__Configure in the Unity:__

```Csharp
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
        
        container.RegisterType<BaseEFEntities>();            
        container.RegisterType<RepositoryNoticeContract, RepositoryNotice>();

    }
}

```

____

__Controller__

```Csharp
public class NoticesController : Controller
{
    //Dependency Injection
    public readonly RepositoryNoticeContract repository;    

    public NoticesController(RepositoryNoticeContract repository)
    {

        this.repository = repository;        

    }    

```    

__Completed Controller__

```Csharp
public class NoticesController : Controller
{
    
    public readonly RepositoryNoticeContract repository;

    public readonly RepositoryTagsContract repositoryTags;

    public NoticesController(RepositoryNoticeContract repository, 
                             RepositoryTagsContract repositoryTags)
    {
        this.repository = repository;
        this.repositoryTags = repositoryTags;
    }        
    
    [AcceptVerbs("GET","POST")]
    public async Task<ActionResult> Index(int? page, string filter)
    {

        int rows = 2;
        ViewBag.filter = filter;

        if (!string.IsNullOrEmpty(filter))
        {
            return View(await repository
                .PaginationAsync(x => x.Title.Contains(filter), 
                x => x.Id, (page ?? 1), rows));
        }

        return View(await repository
            .PaginationAsync(x => x.Id, (page ?? 1), rows));
    }

    [HttpGet()]
    public ActionResult Create()
    {

        ViewBag.TagId = new SelectList(repositoryTags.List(x => x.Description), 
                                "Id", "Description");
        return View();

    }

    [HttpPost()]
    public async Task<ActionResult> Create(Notice notice)
    {        

        await repository.AddAsync(notice);

        return RedirectToAction("Index");

    }

    [HttpGet()]
    public async Task<ActionResult> Edit(int id)
    {

        Notice notice = await repository.FindAsync(id);

        ViewBag.TagId = new SelectList(repositoryTags.List(x => x.Description), 
                                "Id", "Description", notice.TagId);

        return View(notice);

    }

    [HttpPost()]
    public async Task<ActionResult> Edit(int id, Notice notice)
    {

        await repository.EditAsync(notice);            
        return RedirectToAction("Index");

    }

    [HttpGet()]
    public async Task<ActionResult> Delete(int id)
    {

        return View(await repository.FindAsync(id));

    }

    [HttpPost()]
    public async Task<ActionResult> Delete(int id, Notice notice)
    {

        await repository.DeleteAsync(id);

        return RedirectToAction("Index");

    }

}

```   

____

####Methods

| [Add](#add)  | [Edit](#edit)  | [Delete](#delete)  | [Find](#find)  | [All](#all) | [List](#list)  | [Pagination](#pagination) | [Count](#count)  | [Create](#create)  | [GroupBy](#groupby) | [Sum](#sum)  | [Query](#query)  | [QueryCommand](#querycommand) | [Save](#save) | [GroupOrderBy](#grouporderby-version--102) | [CreateAndAttach](#createandattach-version--101) | [IConfiguration](#iconfiguration) | [Action<IConfiguration>](#action) | [IConfiguration Implementation](#resume)


_Example class_

```Csharp
public class Tags
{        
    public int Id { get; set; }
    public string Description { get; set; }        
}
```
_Repository of class Tags_

```Csharp
BaseEFContext Context = new BaseEFContext();
RepositoryTagsContract repTags = new RepositoryTags(Context);
``` 
____

####Add
_Implementation_
```Csharp
T Add(T Model);        
IEnumerable<T> Add(IEnumerable<T> models);

//NET > 4 (Async Method).
Task<T> AddAsync(T Model);
Task<IEnumerable<T>> AddAsync(IEnumerable<T> models);

```
_Usage:_

```Csharp
Tags tag = new Tags();
tag.Description = "Example";

repTags.Add(tag);
```

```Csharp
Tags tag0 = new Tags();
tag0.Description = "Example 0";

Tags tag1 = new Tags();
tag1.Description = "Example 1";

repTags.Add(new List<Tags>() { tag0, tag1 });
```
[back](#methods)
____

####Edit
_Implementation_
```Csharp
bool Edit(T model);

//NET > 4 (Async Method).
Task<bool> EditAsync(T model);
```

_Usage_

```Csharp
Tags tag = repTags.Find(1);

if (tag != null)
{
    tag.Description = "Example Edit 1";
}

repTags.Edit(tag);
```

Or

```Csharp
Tags tag = repTags.Find(1);

if (tag != null)
{
    tag.Description = "Example Edit 1";
    repTags.Save();
}

```
[back](#methods)
____

####Delete
_Implementation_
```Csharp
bool Delete(T model);
bool Delete(IEnumerable<T> model);
bool Delete(params object[] key);
bool Delete(Expression<Func<T, bool>> where);

//NET > 4 (Async Method).
Task<bool> DeleteAsync(IEnumerable<T> model);
Task<bool> DeleteAsync(T model);
Task<bool> DeleteAsync(params object[] key);
Task<bool> DeleteAsync(Expression<Func<T, bool>> where);
````
_Usage_

```Csharp
Tags tag = repTags.Find(1);

if (tag != null)
{
    repTags.Delete(tag);
}
```
or
```Csharp
repTags.Delete(1);
```
or
```Csharp
IEnumerable<Tags> tagsList = repTags.List(x => x.Id == 1, o => o.Id, false);
repTags.Delete(tagsList);
```
or
```Csharp
repTags.Delete(x => x.Id == 1);
```
[back](#methods)
____

####Find
_Implementation_
```Csharp
T Find(params object[] key);
T Find(Expression<Func<T, bool>> where);

//NET > 4 (Async Method).
Task<T> FindAsync(params object[] key);
Task<T> FindAsync(Expression<Func<T, bool>> where);
```
_Usage_
```Csharp
Tags tag = repTags.Find(1);
```
or
```Csharp
Tags tag1 = repTags.Find(x => x.Id == 1);
```
[back](#methods)
____

####All
_Implementation_
```Csharp
IEnumerable<T> All(bool AsNoTracking = true);

IEnumerable<T> All<Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

IList<TResult> All<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, 
            bool AsNoTracking = true);

//NET > 4 (Async Method).
Task<IList<T>> AllAsync(bool AsNoTracking = true);

Task<IList<T>> AllAsync<Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

Task<IList<TResult>> AllAsync<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, 
            bool AsNoTracking = true);
```
_Usage_
```Csharp
IEnumerable<Tags> tagList = repTags.All(); //or repTags.All(false)
IEnumerable<Tags> tagList = repTags.All(x => x.Id == 1, o => o.Description);
//With viewModel
IEnumerable<ViewModel> viewModelList = repTags.All(
    x => x.Id == 1, 
    o => o.Description, 
    x => new ViewModel() { Id = x.Id, Title = x.Description }
);
```
[back](#methods)
____

###List
_Implementation_
```
IList<T> List(bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, Tkey>> orderBy, int page, int total = 10, bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, 
            bool AsNoTracking = true);

IList<T> List<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, int page, int total = 10, 
            bool AsNoTracking = true);

IList<TResult> List<TResult, Tkey>(Expression<Func<T, Tkey>> orderBy, 
            Expression<Func<T, TResult>> select, bool AsNoTracking = true);

IList<TResult> List<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, bool AsNoTracking = true);

IList<TResult> List<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10, 
            bool AsNoTracking = true);

//NET > 4 (Async Method).
Task<IList<T>> ListAsync(bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, Tkey>> orderBy, bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, Tkey>> orderBy, int page, int total = 10,
            bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, 
            bool AsNoTracking = true);

Task<IList<T>> ListAsync<Tkey>(Expression<Func<T, bool>> where, Expression<Func<T, Tkey>> orderBy, 
            int page, int total = 10, bool AsNoTracking = true);

Task<IList<TResult>> ListAsync<TResult, Tkey>(Expression<Func<T, Tkey>> orderBy, 
            Expression<Func<T, TResult>> select, bool AsNoTracking = true);

Task<IList<TResult>> ListAsync<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, bool AsNoTracking = true);

Task<IList<TResult>> ListAsync<TResult, Tkey>(Expression<Func<T, bool>> where, 
            Expression<Func<T, Tkey>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10, 
            bool AsNoTracking = true);
```
_Usage_
```Csharp
IList<Tags> tagList = repTags.List(); //or repTags.List(false);

IList<Tags> tagList = repTags.List(o => o.Description);            

IList<Tags> tagList = repTags.List(x => x.Id == 1, o => o.Description);

IList<Tags> tagList = repTags.List(o => o.Description, 1, 10);

IList<Tags> tagList = repTags.List(x => x.Id == 1, o => o.Description, 1, 10);

IList<ViewModel> tagListViewModel = repTags.List(o => o.Description, 
            x => new ViewModel() { Id = x.Id, Title = x.Description });

IList<ViewModel> tagListViewModel = repTags.List(x => x.Id == 1, o => o.Description, 
            x => new ViewModel() { Id = x.Id, Title = x.Description });

IList<ViewModel> tagListViewModel = repTags.List(x => x.Id == 1, o => o.Description, 
            x => new ViewModel() { Id = x.Id, Title = x.Description }, 1, 10);

```
[back](#methods)
____

####Pagination
_Implementation_
```Csharp
IPagedList<T> Pagination<TOrderBy>(Expression<Func<T, TOrderBy>> orderBy, int page, int total = 10);

IPagedList<TResult> Pagination<TResult, TOrderBy>(Expression<Func<T, TOrderBy>> orderBy, 
            Expression<Func<T, TResult>> select, int page, int total = 10);

IPagedList<T> Pagination<TOrderBy>(Expression<Func<T, bool>> where, Expression<Func<T, TOrderBy>> orderBy, 
            int page, int total = 10);

IPagedList<TResult> Pagination<TResult, TOrderBy>(Expression<Func<T, bool>> where, 
            Expression<Func<T, TOrderBy>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10);

//NET > 4 (Async Method).
Task<IPagedList<T>> PaginationAsync<TOrderBy>(Expression<Func<T, TOrderBy>> orderBy, int page, int total = 10);

Task<IPagedList<TResult>> PaginationAsync<TResult, TOrderBy>(Expression<Func<T, TOrderBy>> orderBy,
            Expression<Func<T, TResult>> select, int page, int total = 10);

Task<IPagedList<T>> PaginationAsync<TOrderBy>(Expression<Func<T, bool>> where, 
            Expression<Func<T, TOrderBy>> orderBy, int page, int total = 10);

Task<IPagedList<TResult>> PaginationAsync<TResult, TOrderBy>(Expression<Func<T, bool>> where, 
            Expression<Func<T, TOrderBy>> orderBy, Expression<Func<T, TResult>> select, int page, int total = 10);
#endif
```
_Usage_
```Csharp
IPagedList<Tags> Pagination = repTags.Pagination(o => o.Id, 1, 10);

IPagedList<Tags> Pagination = repTags.Pagination(x => x.Id == 1, o => o.Id, 1, 10);

IPagedList<ViewModel> PaginationViewModel = repTags.Pagination(o => o.Id, x => 
                        new ViewModel() { Id = x.Id, Title = x.Description }, 1, 10);

IPagedList<ViewModel> PaginationViewModel = repTags.Pagination(x => x.Id == 1, o => o.Id, x => 
                        new ViewModel() { Id = x.Id, Title = x.Description }, 1, 10);
```
[back](#methods)
____

####Count
_Implementation_
```Csharp
long Count();
long Count(Expression<Func<T, bool>> where);

//NET > 4 (Async Method).
Task<long> CountAsync();
Task<long> CountAsync(Expression<Func<T, bool>> where);        
#endif
```
_Usage_
```Csharp
long count = repTags.Count();
long count = repTags.Count(x => x.Description.Contains("A"));
```
[back](#methods)
____
####Create
_Implementation_
```Csharp
DbSet<T1> Create<T1>()
        where T1 : class, new();
T Create();
```
_Usage_
```Csharp
Tags tag = repTags.Create();
DbSet<Notice> notice = repTags.Create<Notice>();
```
[back](#methods)
____

####GroupBy
_Implementation_
```Csharp
IList<TSelect> GroupBy<TKey, TSelect>(Expression<Func<T, TKey>> keySelector, 
            Expression<Func<IGrouping<TKey, T>, TSelect>> select);
IList<TSelect> GroupBy<TKey, TSelect>(Expression<Func<T, bool>> where, Expression<Func<T, TKey>> keySelector, 
            Expression<Func<IGrouping<TKey, T>, TSelect>> select);

//NET > 4 (Async Method).
Task<IList<TSelect>> GroupByAsync<TKey, TSelect>(Expression<Func<T, TKey>> keySelector, 
            Expression<Func<IGrouping<TKey, T>, TSelect>> select);
Task<IList<TSelect>> GroupByAsync<TKey, TSelect>(Expression<Func<T, bool>> where, 
            Expression<Func<T, TKey>> keySelector, Expression<Func<IGrouping<TKey, T>, TSelect>> select);
```
_Usage_
```Csharp
var group = repTags.GroupBy(x => x.Id, x => new
{
    Key = x.Key,
    Count = x.Count()
});

var group = repTags.GroupBy(x => x.Id > 0, x => x.Id, x => new
{
    Key = x.Key,
    Count = x.Count()
});
```
[back](#methods)
____

####Sum
_Implementation_
```Csharp
decimal Sum(Expression<Func<T, decimal>> selector);
decimal? Sum(Expression<Func<T, decimal?>> selector);
double Sum(Expression<Func<T, double>> selector);
double? Sum(Expression<Func<T, double?>> selector);
float Sum(Expression<Func<T, float>> selector);
float? Sum(Expression<Func<T, float?>> selector);
int Sum(Expression<Func<T, int>> selector);
int? Sum(Expression<Func<T, int?>> selector);
long Sum(Expression<Func<T, long>> selector);
long? Sum(Expression<Func<T, long?>> selector);

decimal Sum(Expression<Func<T, bool>> where, Expression<Func<T, decimal>> selector);
decimal? Sum(Expression<Func<T, bool>> where, Expression<Func<T, decimal?>> selector);
double Sum(Expression<Func<T, bool>> where, Expression<Func<T, double>> selector);
double? Sum(Expression<Func<T, bool>> where, Expression<Func<T, double?>> selector);
float Sum(Expression<Func<T, bool>> where, Expression<Func<T, float>> selector);
float? Sum(Expression<Func<T, bool>> where, Expression<Func<T, float?>> selector);
int Sum(Expression<Func<T, bool>> where, Expression<Func<T, int>> selector);
int? Sum(Expression<Func<T, bool>> where, Expression<Func<T, int?>> selector);
long Sum(Expression<Func<T, bool>> where, Expression<Func<T, long>> selector);
long? Sum(Expression<Func<T, bool>> where, Expression<Func<T, long?>> selector);

//NET > 4 (Async Method).
Task<decimal> SumAsync(Expression<Func<T, decimal>> selector);
Task<decimal?> SumAsync(Expression<Func<T, decimal?>> selector);
Task<double> SumAsync(Expression<Func<T, double>> selector);
Task<double?> SumAsync(Expression<Func<T, double?>> selector);
Task<float> SumAsync(Expression<Func<T, float>> selector);
Task<float?> SumAsync(Expression<Func<T, float?>> selector);
Task<int> SumAsync(Expression<Func<T, int>> selector);
Task<int?> SumAsync(Expression<Func<T, int?>> selector);
Task<long> SumAsync(Expression<Func<T, long>> selector);
Task<long?> SumAsync(Expression<Func<T, long?>> selector);

Task<decimal> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, decimal>> selector);
Task<decimal?> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, decimal?>> selector);
Task<double> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, double>> selector);
Task<double?> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, double?>> selector);
Task<float> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, float>> selector);
Task<float?> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, float?>> selector);
Task<int> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, int>> selector);
Task<int?> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, int?>> selector);
Task<long> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, long>> selector);
Task<long?> SumAsync(Expression<Func<T, bool>> where, Expression<Func<T, long?>> selector);
```
_Usage_
```Csharp
int soma = repTags.Sum(x => x.Id);
int soma = repTags.Sum(x => x.Id > 0, x => x.Id);
```

[back](#methods)
____

####Query
_Implementation_
```Csharp
IQueryable<T> Query();
IQueryable<T> Query<T1>(params Expression<Func<T, T1>>[] includes);
DbSqlQuery<T> Query(string sql, params object[] parameters);
DbRawSqlQuery<T1> Query<T1>(string sql, params object[] parameters);
```
_Usage_
```Csharp
IQueryable<Tags> query0 = repTags.Query();
IQueryable<Tags> query1 = repTags.Query(x => x.Notice);
DbSqlQuery<Tags> query2 = repTags.Query("SELECT * FROM tags WHERE Id=@p0", 1);
DbRawSqlQuery<Tags> query3 = repTags.Query<Tags>("SELECT * FROM tags WHERE Id=@p0", 1);
```
[back](#methods)
____

####QueryCommand
_Implementation_
```Csharp
int QueryCommand(string sql, params object[] parameters);

int QueryCommand(TransactionalBehavior transactionalBehavior, string sql, 
            params object[] parameters);

//NET > 4 (Async Method).
Task<int> QueryCommandAsync(string sql, params object[] parameters);

Task<int> QueryCommandAsync(string sql, CancellationToken cancellationToken, 
            params object[] parameters);

Task<int> QueryCommandAsync(TransactionalBehavior transactionalBehavior, string sql, 
            params object[] parameters);

Task<int> QueryCommandAsync(TransactionalBehavior transactionalBehavior, string sql, 
            CancellationToken cancellationToken, params object[] parameters);
```
_Usage_
```Csharp
int count = repTags.QueryCommand("INSERT INTO Tags(Description) VALUES(@p0)", "Example");
int count = repTags.QueryCommand(TransactionalBehavior.DoNotEnsureTransaction,  
            "INSERT INTO Tags(Description) VALUES(@p0)", "Example");
```
[back](#methods)
____


####Save
_Implementation_
```Csharp
int Save();
//NET > 4 (Async Method).
Task<int> SaveAsync();

```
_Usage_
```Csharp
Tags tag = repTags.Find(1);

if (tag != null)
{
    tag.Description = "Example Edit 1";
    repTags.Save();
}
```
[back](#methods)

____

####CreateAndAttach (version >= 1.0.1)
_Implementation_
```Csharp
T CreateAndAttach();

```
_Usage_
```Csharp
Tags tag = repTags.CreateAndAttach();
tag.Description = "New Save";
repTags.Save();

```
[back](#methods)

____

####GroupOrderBy (version >= 1.0.2)
_Implementation_
    
    using Canducci.EntityFramework.Repository.Util;

```Csharp
IPagedList<T> Pagination(GroupOrderBy<T> groupOrderBy, Expression<Func<T, bool>> where, 
            int page, int total = 10);

//NET > 4 (Async Method).
Task<IPagedList<T>> PaginationAsync(GroupOrderBy<T> groupOrderBy, Expression<Func<T, bool>> where, 
            int page, int total = 10);

```
_Usage_
```Csharp
GroupOrderBy<Tags> groupOrderBy = GroupOrderBy<Tags>.Create()
                .Add(x => x.Description, Order.Asc)
                .Add(x => x.Id, Order.Desc);

var result = repTags.Pagination(groupOrderBy, x.Title.Contains(filter), (page ?? 1), rows)

```
[back](#methods)

####IConfiguration

_(Version 1.0.3)_

_Implementation_
    
    using Canducci.EntityFramework.Repository.Util;

```Csharp
//ALL
IEnumerable<T> All(IConfiguration<T> configuration);
IList<TResult> All<TResult>(IConfiguration<T, TResult> configuration);

//LIST
IList<T> List(IConfiguration<T> configuration);
IList<TResult> List<TResult>(IConfiguration<T, TResult> configuration);
``` 
_Usage_   
```Csharp
//ALL
ConfigurationOrderBy<Notice> configNotice = new ConfigurationOrderBy<Notice>();
configNotice.OrderBy.Add(x => x.Id, Order.Asc).Add(x => x.TagId, Order.Desc);

IEnumerable<Notice> n9 = rep.All(configNotice);

ConfigurationOrderBySelect<Notice, ViewModel> configNoticeViewModel = 
                                new ConfigurationOrderBySelect<Notice, ViewModel>();
configNoticeViewModel.OrderBy.Add(x => x.Id, Order.Asc).Add(x => x.TagId, Order.Desc);
configNoticeViewModel.Select.Set(s => new ViewModel() { Id = s.Id, Title = s.Title });

IList<ViewModel> n10 = rep.All(configNoticeViewModel);
//LIST
ConfigurationOrderBy<Notice> configNotice = new ConfigurationOrderBy<Notice>();
configNotice.OrderBy.Add(x => x.Id, Order.Asc).Add(x => x.TagId, Order.Desc);

IList<Notice> n18 = rep.List(configNotice);
```
[back](#methods)
____

####Action

_(Version 1.0.3)_

_Implementation_
    
    using Canducci.EntityFramework.Repository.Util;

```Chsarp
//ALL
IEnumerable<T> All<TConfiguration>(Action<TConfiguration> configuration)
            where TConfiguration : IConfiguration<T>;
IList<TResult> All<TConfiguration, TResult>(Action<TConfiguration> configuration)
    where TConfiguration : IConfiguration<T, TResult>;

//LIST
IList<T> List<TConfiguration>(Action<TConfiguration> configuration)
            where TConfiguration : IConfiguration<T>;
IList<TResult> List<TConfiguration, TResult>(Action<TConfiguration> configuration)
    where TConfiguration : IConfiguration<T, TResult>;    
```                
_Usage_   
```Csharp
//ALL
IEnumerable<Notice> n11 = rep.All<ConfigurationOrderByLimit<Notice>>(a =>
{
    a.OrderBy.Add(x => x.Id);
    a.Page = 1;
    a.Total = 2;
});
IList<ViewModel> n12 = rep.All<ConfigurationOrderBySelect<Notice, ViewModel>, ViewModel>(a =>
{
    a.OrderBy.Add(x => x.Id);
    a.Select.Set(s => new ViewModel { Id = s.Id, Title = s.Title });
});

//LIST
IList<Notice> n20 = rep.List<ConfigurationOrderByLimit<Notice>>(a =>
{
    a.OrderBy.Add(x => x.Id);
    a.Page = 1;
    a.Total = 2;
});
IList<ViewModel> n21 = rep.List<ConfigurationOrderBySelect<Notice, ViewModel>, ViewModel>(a =>
{
    a.OrderBy.Add(x => x.Id);
    a.Select.Set(s => new ViewModel { Id = s.Id, Title = s.Title });
});
```
[back](#methods)

___

####Resume:

_(Version 1.0.3)_

_Implementation_ `IConfiguration<T>` _and_ `IConfiguration<T, TResult>`

```Csharp
ConfigurationOrderBy<T>
ConfigurationOrderByLimit<T>
ConfigurationOrderByPagination<T>
ConfigurationOrderByPaginationSelect<T, TResult>
ConfigurationOrderBySelect<T, TResult>
ConfigurationOrderBySelectLimit<T, TResult>
ConfigurationOrderByWhere<T>
ConfigurationOrderByWherePagination<T>
ConfigurationOrderByWherePaginationSelect<T, TResult>
ConfigurationOrderByWhereSelect<T, TResult>
ConfigurationOrderByWhereSelectLimit<T, TResult>
ConfigurationWhere<T>
ConfigurationWhereLimit<T>
ConfigurationWhereSelect<T, TResult>
ConfigurationWhereSelectLimit<T, TResult>

```    