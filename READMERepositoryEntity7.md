#Canducci EntityFramework Repository

[![Canducci.EntityFramework.Repository](http://i1194.photobucket.com/albums/aa377/netdragoon1/1452823324_icon-89-document-file-sql_zpslt5kmpu9.png)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)

[![NuGet](https://img.shields.io/nuget/dt/CanducciEntityFrameworkRepository.svg?style=plastic&label=downloads)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)
[![NuGet](https://img.shields.io/nuget/v/CanducciEntityFrameworkRepository.svg?style=plastic&label=version)](https://www.nuget.org/packages/CanducciEntityFrameworkRepository/)

## NUGET

```Csharp

PM> Install-Package Canducci.EntityFramework.Repository -Pre

```

##HOW TO?

Use _namespace_ :

    using Microsoft.Data.Entity;
    using Canducci.EntityFramework.Repository.Contracts;
    using Canducci.EntityFramework.Repository.Interfaces;

####Class:

    public class Car
    {
        public int Id { get; set; }
        public string Model { get; set; }
        public short Year { get; set; }
    }


    public class Database: DbContext
    {
        public Database()
        {

        }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer("Server=.\\SQLExpress;Database=Mvc;User Id=sa;Password=senha;");
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Car>()
                .ToTable("Cars");

            modelBuilder.Entity<Car>()
                .Property(x => x.Model)
                .IsRequired()
                .HasMaxLength(50);

            modelBuilder.Entity<Car>()
                .Property(x => x.Year)
                .IsRequired();

            modelBuilder.Entity<Car>()
                .Property(x => x.Id)
                .UseSqlServerIdentityColumn()
                .IsRequired();             


        }
    }

####Class Repository

The `Database`: `DbContext` and `Car`: `DbSet`.

```Csharp

using Canducci.EntityFramework.Repository.Contracts;
namespace WebApplication1.Models
{
    
    public abstract class RepositoryCarContract:
        Repository<Car, Database>,
        IRepository<Car, Database>
    {
        public RepositoryCarContract(Database database)
            :base(database)
        {
        }
    }

    public sealed class RepositoryCar :
        RepositoryCarContract
    {
        public RepositoryCar(Database database)
            : base(database)
        {
        }
    }
}

```