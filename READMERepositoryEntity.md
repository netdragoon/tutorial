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
    public abstract class RepositoryNoticeContract: Repository<Notice, BaseEFEntities>,
                                                    IRepository<Notice, BaseEFEntities>
    {
        public RepositoryNoticeContract(BaseEFEntities ctx)
            : base(ctx)
        {
        }
    }
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
    public readonly RepositoryNoticeContract repository;    
    public NoticesController(RepositoryNoticeContract repository)
    {
        this.repository = repository;        
    }

```    

_Completed Controller_

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