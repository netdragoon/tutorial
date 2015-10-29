##Canducci EXCEL 

####(version: 1.0.0)

[![Canducci Excel](http://i666.photobucket.com/albums/vv25/netdragoon/1446163380_excel_zps5lhqezet.png)](https://www.nuget.org/packages/CanducciQuoteDolar/)

###Classes

__1) Interfaces__

- `IHeader`
- `IHeaderCollection`
- `IListToExcel`

__2) Classes Concretas__

- `Header`
- `HeaderCollection`
- `ListToExcel`

__3) Metodos Extensivos para__ `IEnumerable`, `IEnumerable<T>`, `IQueryable` e `IQueryable<T>`

- `bool ToExcelSaveAs<T>`
- `byte[] ToExcelByte<T>`

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package CanducciQuoteDolar

```

##Como utilizar?

Declare o namespace `using Canducci.Excel;` 

##Array Simples

```Csharp
//Configurando o cabeçalho
HeaderCollection headerData = new HeaderCollection();
headerData.Add(new Header("Datas", 1));

//Array Simples de Datas
var n = new DateTime[] { DateTime.Now.Date.AddDays(-15), DateTime.Now.Date.AddDays(-16) };

//Usando o método extensivo para gerar o arquivo e gravar em disco.
n.ToExcelSaveAs("Result\\DatasSimples.xlsx", headerData);

```

##Array Bidimensional

```Csharp

//Array Bidimensional
int[,] c = new int[2,2];
c[0, 0] = 1; c[0, 1] = 2;
c[1, 0] = 3; c[1, 1] = 4;

//Configurando o cabeçalho
IHeaderCollection headersArrayBi = new HeaderCollection();
headersArrayBi.Add(2, "Col-");

//Usando o método extensivo para gerar o arquivo e gravar em disco.
c.ToExcelSaveAs<int[,]>("Result\\ArrayBi2Dimensioanl.xlsx", headersArrayBi);

```

##List

```Csharp

//Configurando o cabeçalho
HeaderCollection headers = new HeaderCollection();                  
headers.Add(new Header("Id", 1));
headers.Add(new Header("Nome", 2));
headers.Add(new Header("Valor", 3));
headers.Add(new Header("Data de Validade", 4));
headers.Add(new Header("Hora Stamp", 5));

//Criando manualmente uma Lista tipada
IList<Test> tests = new List<Test>();
tests.Add(new Test() { Id = 1, Nome = "Test 1", Valor = 10M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 2, Nome = "Test 2", Valor = 2000.69M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 3, Nome = "Test 3", Valor = 10M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 4, Nome = "Test 4", Valor = 20M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 5, Nome = "Test 5", Valor = 10M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 6, Nome = "Test 6", Valor = 2000.69M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 7, Nome = "Test 7", Valor = 10M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });
tests.Add(new Test() { Id = 8, Nome = "Test 8", Valor = 20M, 
            Data = DateTime.Now.Date, Time = DateTime.Now.TimeOfDay });

//Usando o método extensivo para gerar o arquivo e gravar em disco.
tests.ToExcelSaveAs("Result\\List.xlsx", headers);

```

##Entity Framework (IQueryable e IEnumberable)

__Simples Console ou Desktop__

```Csharp

//Classe Contexto do Entity Framework
using (AdventureWorks2014Entities db = new AdventureWorks2014Entities())
{

    //IQueryable<Person>   
    var Query = db.Person
            .AsNoTracking()
            .AsQueryable();
    
    //Configurando o cabeçalho
    IHeaderCollection headerEF = new HeaderCollection();
    headerEF.Add(new Header("Primeiro Nome", 1));

    //Usando o método extensivo para gerar o arquivo e gravar em disco.
    //Foi selecionado apenas um item da tabela o `FirstName`
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

    //Array de bytes
    byte[] fileByte = null;

    using (AdventureWorks2014Entities db = new AdventureWorks2014Entities())
    {

        //Configurando o cabeçalho
        IHeaderCollection headers = HeaderCollection.Create();
        headers.Add(Header.Create("Id", 1));
        headers.Add(Header.Create("Sexo", 2));
        headers.Add(Header.Create("Data", 3));
        headers.Add(Header.Create("Trabalho", 4));

        //Usando o método extensivo para gerar um array de bytes em memória
        //pelo método extensivo ToExcelByte
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
    
    //O navegador fará um download do arquivo 
    //e você pode abrir no seu aplicativo Excel
    return File(fileByte, ContentTypeExcel.xlsx);

}

```

__ASP.NET MVC - Maneira simples instânciando__ `ListToExcel`

```Csharp
[Route("excel1")]
public FileContentResult CreateExcel1()
{
    //Array de Bytes
    byte[] fileByte = null;

    //Entity Framework e MemoryStream
    using (AdventureWorks2014Entities db = new AdventureWorks2014Entities())
    using (System.IO.MemoryStream Stream = new System.IO.MemoryStream())            
    {
        //Configurando cabeçalho
        IHeaderCollection Headers = HeaderCollection.Create();
        Headers.Add(Header.Create("Departamento", 1));
        Headers.Add(Header.Create("Razão Social", 2));

        //Montando a Query e retornando um IQueryable
        IQueryable Query = db.Department.Select(x => new { x.DepartmentID, x.Name }).AsQueryable();

        //Instânciando a classe ListToExcel
        IListToExcel<Department> listDep = new ListToExcel<Department>(Query, Headers);

        //SaveAs passando valores para o MemoryStream (`Stream`)
        listDep.SaveAs(Stream);

        //Jogando valores em um Array de Bytes
        fileByte = Stream.ToArray();

    }

    //O navegador fará um download do arquivo 
    //e você pode abrir no seu aplicativo Excel
    return File(fileByte, ContentTypeExcel.xlsx);

}
```
Observações:

- Não é obrigatório o uso da classe `HeaderCollection`, mas, é uma forma de configuração do titulo e ordem de cada coluna. A ordem dos valores também deve seguir a mesma ordem do que foi colocado em cada titulo sempre começando do número 1 (ex. 1,2,3, sendo que menores ou igual a 0 (zero) causa um Exception (erro)).

___Exemplo:___

```csharp
IHeaderCollection Headers = HeaderCollection.Create();
Headers.Add(Header.Create("Departamento", 1));
Headers.Add(Header.Create("Razão Social", 2));
```
No exemplo acima foi criado duas colunas começando pela Departamento e a próxima Razão Social.

- O arquivo gerado em Excel tem como finalidade o transporte simples de cada informação, tendo o seu tipo de cada linha e coluna correspondente ao que foi enviado. 

- O pacote não tem formatação de layout (apesar que o titulo é centralizado por padrão e as colunas seguem a formatação de acordo com o tipo enviado), fontes, cores, etc., a preocupação dele é somente só o envio de uma informação para ser modificada em um arquivo do excel.

- O arquivo gerado é 100% compativel com __Microsoft Office Excel___