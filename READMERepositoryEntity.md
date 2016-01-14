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

    public abstract class RepositoryTagsContract : Repository<Tags, BaseEFEntities>,
                                                   IRepository<Tags, BaseEFEntities>
    {
        public RepositoryTagsContract(BaseEFEntities ctx)
            : base(ctx)
        {

        }
    }
    public sealed class RepositoryTags : RepositoryTagsContract
    {
        public RepositoryTags(BaseEFEntities ctx)
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
        container.RegisterType<RepositoryTagsContract, RepositoryTags>();

    }
}

```