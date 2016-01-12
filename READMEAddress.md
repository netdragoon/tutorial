#Canducci Endereço

__Web Service http://viacep.com.br/__

##[Demo](http://canduccipackages.apphb.com/#/)

[![Canducci Cep](http://i666.photobucket.com/albums/vv25/netdragoon/cep_zpsoqtae5hr.png)](https://www.nuget.org/packages/CanducciCepEndereco/)

[![NuGet](https://img.shields.io/nuget/dt/CanducciCepEndereco.svg?style=plastic&label=downloads)](https://www.nuget.org/packages/CanducciCepEndereco/)
[![NuGet](https://img.shields.io/nuget/v/CanducciCepEndereco.svg?style=plastic&label=version)](https://www.nuget.org/packages/CanducciCepEndereco/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package CanducciCepEndereco

```

##Como utilizar?

Declare o namespace `using Canducci.Address;` 

###Busca de vários CEP informando UF, Cidade e Endereço?

1) Simples

```Csharp

try
{

  //Observações
  //Cidade com no minimo 3 letras
  //Endereço com minimo 3 letras

  //Método AddressInfo só retorna os 100 primeiros registros, ou seja,
  //coloque o maior detalhamento na cidade e endereço.

AddressInfo addressInfo = null;
using (AddressLoad addressLoad = new AddressLoad())  	
{     

    AddressInfo addressInfo = addressLoad.AddressInfo(UfAddress.PR, "Colorado", "Ant")

  	ZipCode[] zips = addressInfo.AddressList;

  	if (zips.Count() > 0)
  	{


  	}

} 
catch (ZipCodeException ex)
{
    throw ex;
}

```

2) Métodos Async (_Exemplo:_ Web >= `4.5`)

```Csharp
public async Task<ActionResult> Address()
{

    AddressInfo addressInfo = null;

    using (AddressLoad addressLoad = new AddressLoad())            
    {

        addressInfo = await addressLoad.AddressInfoAsync(UfAddress.PR, "Colorado", "Ant");

        ZipCode[] zips = addressInfo.AddressList; 

        if (zips.Count() > 0)
        {


        }   

    }

    return View(addressInfo);
}
```
