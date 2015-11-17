#Canducci Endereço

###(version: 1.0.0)

__Web Service http://viacep.com.br/__

##[Demo](http://canduccipackages.apphb.com/#/)

[![Canducci Cep](http://i666.photobucket.com/albums/vv25/netdragoon/cep_zpsoqtae5hr.png)](https://www.nuget.org/packages/CanducciCepEndereco/)

[![NuGet](https://img.shields.io/nuget/dt/CanducciCepEndereco.svg?style=plastic)](https://www.nuget.org/packages/CanducciCepEndereco/)
[![NuGet](https://img.shields.io/nuget/v/CanducciCepEndereco.svg?style=plastic)](https://www.nuget.org/packages/CanducciCepEndereco/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package CanducciCepEndereco

```

##Como utilizar?

Declare o namespace `using Canducci.Address;` 

###Busca de vários CEP informando UF, Cidade e Endereço?

```Csharp

try
{

	//Observações
	//Cidade com no minimo 3 letras
	//Endereço com minimo 3 letras

	//Método AddressInfo só retorna os 100 primeiros registros.
    using (AddressLoad addressLoad = new AddressLoad())
   	using (AddressInfo addressInfo = addressLoad.AddressInfo(UfAddress.PR, "Colorado", "Ant"))
   	{               
    	ZipCode[] zips = addressInfo.AdressList; 

    	if (zips.Count() > 0)
    	{


    	}   

   	} 

}
catch (ZipCodeException ex)
{
    throw ex;
}

```
