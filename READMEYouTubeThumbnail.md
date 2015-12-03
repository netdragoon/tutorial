#Canducci YoutubeThumbnail

[![Canducci YoutubeThumbnail](http://i1194.photobucket.com/albums/aa377/netdragoon1/1449170629_Neon_Line_Social_Circles_50Icon_10px_grid-45_zpsjazbo2dc.png)](https://github.com/netdragoon/tutorial/blob/master/READMEYouTubeThumbnail.md)

##Install Package (NUGET)

To install Canducci YoutubeThumbnail, run the following command in the [Package Manager Console](http://docs.nuget.org/consume/package-manager-console)

```Csharp

PM> Install-Package CanducciYoutubeThumbnail

```

##How to use?

Declare o namespace:

```Csharp
using Canducci.YoutubeThumbnail;
```

Create a folder in your MVC Web Application, for example `Images/`, to serve as image cache, have adequate performance on single request for a particular address.

####Controller

```Csharp
using Canducci.YoutubeThumbnail;
using System;
using System.Threading.Tasks;
using System.Web.Mvc;
namespace WebCanducci45.Controllers
{
    public class YouTubeController : Controller
    {
        [HttpGet]
        public ActionResult Index()
        {
            return View();
        }
        [HttpPost]
        public async Task<ActionResult> Index(string Url)
        {            
            Thumbnail thumb = null;
            Uri _url;
            try
            {
                if (Thumbnail.UrlTryParse(Url, out _url))
                {
                    thumb = new Thumbnail(_url);                    
                    await thumb.SaveAllAsync("Imagens/", Server.MapPath("~"));
                    ViewBag.Exits = false;
                    if (await thumb.ThumbnailPicture0.ExistsAsync())
                    {
                        ViewBag.Exits = true;
                    }
                    ViewBag.Url = Url;
                    return View(thumb);
                }
                return View();
            }
            catch (Exception ex)
            {
                throw ex;
            }           
        }
    }
}
```

####View

```HTML
@model Canducci.YoutubeThumbnail.Thumbnail
@{Layout = null;}
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Pagina do You Tube</title>
    <script src="~/Scripts/jquery-1.10.2.min.js"></script>
    <link href="~/Content/bootstrap.min.css" rel="stylesheet" />
    <script src="~/Scripts/bootstrap.min.js"></script>
</head>
<body>
    <div class="container">
        @using (Html.BeginForm())
        {
            <div class="">
                <input type="url" name="Url" id="Url" class="form-control" value="@ViewBag.Url" required />
                <button type="submit" class="btn btn-primary">Enviar</button>
            </div>
        }
        @if (Model != null)
        {
            <p>@ViewBag.Exits</p>
            <div class="row">
                <div class="col-md-6">
                    <h4>Foto 0</h4>
                    <p><img src="@Model.ThumbnailPicture0.PathWeb" style="border:0" /> </p>
                </div>
            </div>
            <div class="row">
                <div class="col-md-2">
                    <h4>Foto 1</h4>
                    <p><img src="@Model.ThumbnailPicture1.PathWeb" style="border:0" /> </p>
                </div>
                <div class="col-md-2">
                    <h4>Foto 2</h4>
                    <p><img src="@Model.ThumbnailPicture2.PathWeb" style="border:0" /> </p>
                </div>
                <div class="col-md-2">
                    <h4>Foto 3</h4>
                    <p><img src="@Model.ThumbnailPicture3.PathWeb" style="border:0" /> </p>
                </div>
                <div  class="col-md-2">
                    <h4>Foto Default</h4>
                    <p><img src="@Model.ThumbnailPictureDefault.PathWeb" style="border:0" /> </p>
                </div>
            </div>
            <div>
                <h2>Video Compartilhar</h2>
                <p>@Model.VideoShare</p>
            </div>
            <div>
                <h2>Video Embed</h2>
                <p>@Html.Raw(Model.VideoEmbed())</p>
            </div>
        }
    </div>
</body>
</html>
```

