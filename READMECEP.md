#Canducci CEP

__Web Service http://viacep.com.br/__

[Demo](http://canduccipackages.apphb.com/#/)

[![Canducci Cep](http://i666.photobucket.com/albums/vv25/netdragoon/cep_zpsoqtae5hr.png)](https://www.nuget.org/packages/CanducciCep/)

##Instalação do Pacote (NUGET)

```Csharp
PM> Install-Package CanducciCep
```

##Como utilizar?

Declare o namespace `using Canducci.Zip;` 

###Busca das informações pelo CEP informado?

```Csharp

try
{
	
	//Formato válido para o CEP: 01414000 ou 01414-000
    ZipCodeInfo zip = ZipCodeLoad.Find("01414000");

    if (!zip.Erro)
    {

        //código postal encontrado ...

    }
}
catch (ZipCodeException ex)
{
    throw ex;
}

```

###Busca de vários CEP informando UF, Cidade e Endereço?

```Csharp

try
{

	//Observações
	//Cidade com no minimo 3 letras
	//Endereço com minimo 3 letras

	//Método Address só retorna os 100 primeiros registros.
    ZipCodeInfo[] zips = ZipCodeLoad.Address(ZipCodeUF.SP, "São Paulo", "Ave");

    if (zips.Count() > 0)
    {

    }

}
catch (ZipCodeException ex)
{
    throw ex;
}

```
