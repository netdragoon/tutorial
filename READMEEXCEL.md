#Canducci EXCEL 

###(version: 1.0.0)

[![Canducci Excel](http://i666.photobucket.com/albums/vv25/netdragoon/1446163380_excel_zps5lhqezet.png)](https://www.nuget.org/packages/CanducciQuoteDolar/)

##Classes
1) Interfaces
- IHeader
- IHeaderCollection
- IListToExcel

2) Classes Concretas
- Header
- HeaderCollection
- ListToExcel

3) Metodos Extensivos para IEnumerable, IEnumerable<T>, IQueryable e IQueryable<T>
- bool ToExcelSaveAs<T>
- byte[] ToExcelByte<T>

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package CanducciQuoteDolar

```

##Como utilizar?

Declare o namespace `using Canducci.Excel;` 

##Array Simples

```Csharp
HeaderCollection headerData = new HeaderCollection();
headerData.Add(new Header("Datas", 1));
var n = new DateTime[] { DateTime.Now.Date.AddDays(-15), DateTime.Now.Date.AddDays(-16) };
n.ToExcelSaveAs("Result\\DatasSimples.xlsx", headerData);
```

##Array Bidimensional

```Csharp
int[,] c = new int[2,2];
c[0, 0] = 1; c[0, 1] = 2;
c[1, 0] = 3; c[1, 1] = 4;
IHeaderCollection headersArrayBi = new HeaderCollection();
headersArrayBi.Add(2, "Col-");
c.ToExcelSaveAs<int[,]>("Result\\ArrayBi2Dimensioanl.xlsx", headersArrayBi);

```
##List

```Csharp

HeaderCollection headers = new HeaderCollection();                  
headers.Add(new Header("Id", 1));
headers.Add(new Header("Nome", 2));
headers.Add(new Header("Valor", 3));
headers.Add(new Header("Data de Validade", 4));
headers.Add(new Header("Hora Stamp", 5));

IList<Test> tests = new List<Test>();
tests.Add(new Test() { Id = 1, Nome = "Test1", Valor = 10M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 2, Nome = "Test2", Valor = 2000.69M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 3, Nome = "Test3", Valor = 10M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 4, Nome = "Test4", Valor = 20M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 1, Nome = "Test5", Valor = 10M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 2, Nome = "Test6", Valor = 2000.69M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 3, Nome = "Test7", Valor = 10M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 4, Nome = "Test8", Valor = 20M, Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });

var item = new ListToExcel<Test>(tests, headers);
tests.ToExcelSaveAs("Result\\List.xlsx", headers);

```

##Entity Framework (IQueryable e IEnumberable)

__Simples Console ou Desktop__

```Csharp
using (AdventureWorks2014Entities db = new AdventureWorks2014Entities())
{
    db.Configuration.LazyLoadingEnabled = false;
    db.Configuration.ProxyCreationEnabled = false;
    
    var Query = db.Person.
        AsNoTracking()
        .AsQueryable();
    //////////////////////////////////////////////////////////////////////////////////////////////////////////
    IHeaderCollection headerEF = new HeaderCollection();
    headerEF.Add(new Header("Primeiro Nome", 1));
    Query.Select(x => x.FirstName)                    
        .Take(10)
        .ToExcelSaveAs("Result\\Forma1.xlsx", headerEF);
}

```

__ASP.NET MVC__
```Csharp

[Route("excel")]
public FileContentResult CreateExcel()
{
    byte[] fileByte = null;

    using (AdventureWorks2014Entities db = new AdventureWorks2014Entities())
    {
        IHeaderCollection headers = HeaderCollection.Create();
        headers.Add(Header.Create("Id", 1));
        headers.Add(Header.Create("Idade", 2));
        headers.Add(Header.Create("Data de Fogo", 3));
        headers.Add(Header.Create("Trabalho Candidatura", 4));

        db.Configuration.LazyLoadingEnabled = false;
        db.Configuration.ProxyCreationEnabled = false;

        fileByte = db.Employee
            .AsNoTracking()
            .Select(x => new
            {
                x.BusinessEntityID,
                x.Gender,
                x.HireDate,
                x.JobCandidate
            })                    
            .Take(20)
            .ToExcelByte(headers);
    }           
    
    return File(fileByte, ContentTypeExcel.xlsx);
}

```