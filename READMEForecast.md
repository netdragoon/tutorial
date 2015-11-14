##Canducci FORECAST

####(version: 0.0.1)

__Web Service [CPTEC/INPE](http://servicos.cptec.inpe.br/XML/)__


##[Demo](http://canduccipackages.apphb.com/#/)

[![Canducci Forecast](http://i666.photobucket.com/albums/vv25/netdragoon/1447477148_Weather_zpsczx6fzr6.png)](https://www.nuget.org/packages/Canducci.Forecast/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package Canducci.Forecast

```

##Como utilizar?

Declare o namespace `using Canducci.Forecast;` 


```Csharp

try
{
    
    CityForecast cf = new CityForecast();
    
    //LISTA DE CIDADES
    //Busca textual das cidades para saber o código para utilizar
    // na pesquisa `cf.Forecast`
    Cities cities = cf.Cities("Sao Paulo");

    //Id = 244 é da Cidade de São Paulo Capital
    //ForecastDay pode ser de 7 resultados ou de 4 (valor padrão)
    Prevision prevision = cf.Forecast(244, ForecastDay.D4);


    cf.Dispose();
}
catch (CityForecastException ex)
{
    throw ex;
}

```

Nota: ___para versão NET 4.5 e superiores possui os métodos___ `Async`.


Exemplo MVC com Async


```Csharp
[HttpPost]
[Route("cities")]
public async Task<JsonResult> ForecastCities(string name)
{
   
    if (!string.IsNullOrEmpty(name))
    {
        ICities cities = null;

        using (ICityForecast fc = new CityForecast())
        {
            cities = await fc.CitiesAsync(name.WithoutAccents());
        }
        return Json(cities, JsonRequestBehavior.DenyGet);
    }    
    return Json(new string[] { }, JsonRequestBehavior.DenyGet);
}
[HttpPost]
[Route("prevision")]
public async Task<JsonResult> ForecastPrevision(int? Id, int? Count = 4)
{    
    IPrevision prev = null;
    if (Id.HasValue)
    {                    
        using (ICityForecast fc = new CityForecast())
        {
            prev = await fc.ForecastAsync(Id.Value, 
                ((Count.HasValue && Count == 7) ? 
                    ForecastDay.D7 : 
                    ForecastDay.D4));
        }
        return Json(prev, JsonRequestBehavior.DenyGet);
    }
    return Json(new string[] { }, JsonRequestBehavior.DenyGet);
}

```
___Observação:___ Quando for resgatar as cidades o texto informado não pode ser acentuado, então
use a `class` logo abaixo como método de extensão para remover os acentos:

```Csharp
public static class Methods
{
    public static string WithoutAccents(this string str)
    {
        if (string.IsNullOrEmpty(str))
        {
            return string.Empty;
        }
        byte[] bytes = System.Text.Encoding.GetEncoding("iso-8859-8").GetBytes(str);
        return System.Text.Encoding.UTF8.GetString(bytes);
    }
}

```
###Modelo de Resposta da classe Cities e City

___Cities___

Classe que tem as informações das cidades seguindo o modelo de cidade.

```Csharp

public class Cities : ICities
{
    public Cities()
    {
        Citys = new List<City>();
    }
    public List<City> Citys { get; set; }
    public long Count
    {
        get
        {
            return Citys.Count;
        }
    }
}
```

___City___

Modelo de cidade.

```Csharp
public class City : ICity
{    
    public int Id { get; set; } 
    public string Name { get; set; }
    public string Uf { get; set; }
}

```

###Modelo de Resposta da classe Prevision e Days

___Prevision___

Classe que tem as informações da cidade e ultima atualização
 da Previsão do tempo

```Csharp
public class Prevision : IPrevision
{    
    public string Name { get; set; }
    public string Uf { get; set; }
    public DateTime Updated { get; set; }
    public List<Days> Days { get; set; }
}
```
___Days___

Classe que possui as datas relativas a previsão do tempo.

```Csharp
public class Days : IDays
{    
    public DateTime Data { get; set; }
    public string Time { get;set; }
    public Term TimeTerm { get; private set; }
    public decimal Max { get; set; }
    public decimal Min { get; set; }
    public decimal Iuv { get; set; }
}

```