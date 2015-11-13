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