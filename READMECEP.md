#Canducci CEP 

__Web Service http://viacep.com.br/__

##[Demo](http://canduccipackages.apphb.com/#/)

[![Canducci CEP](http://i1194.photobucket.com/albums/aa377/netdragoon1/1451501901_send-mail-circle_zps7rugskgd.png)](https://www.nuget.org/packages/CanducciCep/)

####Informações Pacote Canducci CEP

[![Downloads](https://img.shields.io/nuget/dt/CanducciCep.svg?style=plastic&label=downloads)](https://www.nuget.org/packages/CanducciCep/)
[![Version](https://img.shields.io/nuget/v/CanducciCep.svg?style=plastic&label=version)](https://www.nuget.org/packages/CanducciCep/)
[https://img.shields.io/twitter/url/http/www.nuget.org/packages/CanducciCep.svg?style=social]

####Informações Nuget

[![NuGet Core Version](https://img.shields.io/nuget/v/Nuget.Core.svg?style=plastic&label=nuget core version)](https://www.nuget.org/)
[![NuGet Core Configuration](https://img.shields.io/nuget/v/NuGet.Configuration.svg?style=plastic&label=nuget core configuration version)](https://www.nuget.org/)
[![NuGet Core Configuration](https://img.shields.io/nuget/v/NuGet.Server.svg?style=plastic&label=nuget server version)](https://www.nuget.org/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package CanducciCep

```

###Como utilizar?

Declare o namespace `using Canducci.Zip;` 

####Busca das informações pelo CEP informado?

```Csharp

try
{
	//Observação
	//Formato válido para o CEP: 01414000 ou 01414-000

    ZipCodeLoad zipLoad = new ZipCodeLoad();
    
    ZipCode zipCode = null;
    if (ZipCode.TryParse("01414000", out zipCode))
    {
        ZipCodeInfo zipCodeInfo = zipLoad.Find(zipCode);
    }   
    
}
catch (ZipCodeException ex)
{
    throw ex;
}

```

####Busca de vários CEP informando UF, Cidade e Endereço?

```Csharp

try
{

	//Observações
	//Cidade no minimo 3 letras
	//Endereço no minimo 3 letras

    ZipCodeLoad zipLoad = new ZipCodeLoad();

    AddressCode addressCode = null;
    if (AddressCode.TryParse(ZipCodeUf.SP, "SÃO PAULO", "AVENIDA ANA", out addressCode))
    {
        ZipCodeInfo[] zipCodeInfos = zipLoad.Address(addressCode);
    }

}
catch (ZipCodeException ex)
{
    throw ex;
}

```

####Exemplo MVC ASP.NET (com async, await e Task)

```csharp
using System.Web.Mvc;
using Canducci.Zip;
using System.Threading.Tasks;

namespace Canducci.Web4._5._1.Controllers
{
    public class HomeController : Controller
    {
        public readonly ZipCodeLoad load;

        public HomeController()
        {
            load = new ZipCodeLoad();
        }

        protected override void Dispose(bool disposing)
        {
            if (load != null) load.Dispose();
            base.Dispose(disposing);
        }

        //com Async, Await e Task
        public async Task<ActionResult> BuscaComAsync()
        {           
             
            ZipCodeInfo _info = await load.FindAsync("01414000");
            ZipCodeInfo[] _infos = await load.AddressAsync(ZipCodeUf.SP, "SÃO PAULO", "AVE");

            return View();

        }

        //comum
        public ActionResult BuscaNormal()
        {

            ZipCodeInfo _info = load.Find("01414000");
            ZipCodeInfo[] _infos = load.Address(ZipCodeUf.SP, "SÃO PAULO", "AVE");

            return View();

        }

```

####Lista de Unidade Federativas do Brasil

```csharp
Dictionary<object, object> ufs = ZipCodeLoad.UfToList();

```

####Verificação de dados válidos para buscas de CEP e Endereços

_CEP_

```csharp    
    ZipCode zipCode = null;
    if (ZipCode.TryParse("01414000", out zipCode))
    {

    }

```

_Endereços_

```csharp    
    AddressCode addressCode = null;
    if (AddressCode.TryParse(ZipCodeUf.SP, "SÃO PAULO", "AVENIDA ANA", out addressCode))
    {

    }

```