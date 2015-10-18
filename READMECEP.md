#Canducci CEP

__Web Service http://viacep.com.br/__

[![Canducci Cep](http://i666.photobucket.com/albums/vv25/netdragoon/cep_zpsoqtae5hr.png)](https://www.nuget.org/packages/CanducciCep/)

Instalação do Pacote (NUGET)

```Csharp
PM> Install-Package CanducciCep
```

Como utilizar

Declare seu namespace `using Canducci.Zip;`. Após a declaração faça a pesquisa do CEP dessa forma:

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