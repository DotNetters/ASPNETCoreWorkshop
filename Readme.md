# Entorno de desarrollo

## GIT

**Comprobar:**

    git --version       
> `git version 2.7.1.windows.2`

**Instalar en Windows:**

> [Descargar e instalar](https://git-scm.com/download/win)

**Instalar en Debian/Ubuntu:**

    sudo apt-get install git

## NodeJs
**Comprobar:**

    node --version       
> `v5.6.0`

**Instalar en Windows:**

> [Descargar e instalar](https://nodejs.org/dist/v5.8.0/node-v5.8.0-x64.msi) 

**Instalar en Debian/Ubuntu:**

    curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -
    sudo apt-get install -y nodejs

## Gulp, Bower y Yeoman

    npm install -g gulp bower yo generator-aspnet

## Visual Studio Code
- [Descargar e instalar](https://code.visualstudio.com/Download)
- [Instalar DotNet CLI](https://dotnet.github.io/getting-started/)
- Instalar extensiones
  - yo
  - C#

## Mono (solo en Debian/Ubuntu)

**Comprobar:**

    mono --version

>Mono JIT compiler version 4.2.2 (Stable 4.2.2.30/996df3c Mon Feb 15 17:30:30 UTC 2016) 

>Copyright (C) 2002-2014 Novell, Inc, Xamarin Inc and Contributors. www.mono-project.com
>```
TLS:           __thread
SIGSEGV:       altstack
Notifications: epoll
Architecture:  amd64
Disabled:      none
Misc:          softdebug 
LLVM:          supported, not enabled.
GC:            sgen
```

**Instalar:**

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
    echo "deb http://download.mono-project.com/repo/debian wheezy main" | sudo tee /etc/apt/sources.list.d/mono-xamarin.list
    sudo apt-get update
    sudo apt-get install mono-complete 

## .NET Core
**Comprobar:**

    dnvm list

>```
Active Version           Runtime Architecture OperatingSystem Alias
------ -------           ------- ------------ --------------- -----
  *    1.0.0-rc1-update1 clr     x86          win             default
```
```
Active Version              Runtime Architecture OperatingSystem Alias
------ -------              ------- ------------ --------------- -----
       1.0.0-rc1-update1    coreclr x64          linux           
  *    1.0.0-rc1-update1    mono                 linux/osx       default
```

**Instalar en Windows:**
```
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "&{$Branch='dev';iex ((new-object net.webclient).DownloadString('https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.ps1'))}"
```
```
    dnvm upgrade -r clr
```
    
**Instalar en Debian/Ubuntu:**
```
sudo apt-get install unzip curl
curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
```
```
dnvm upgrade -r mono
```

# MusicStoreWebApp
## Proyecto
```
yo aspnet web MusicStoreWebApp
cd "MusicStoreWebApp"
dnu restore
dnx web
```
```
bower install
dnu commands install Microsoft.Dnx.Watcher
SET ASPNET_ENV=Development
dnx-watch web
```

## Base de datos (ORM)
- Descargar base de datos de ejemplo (SQLite) de [chinookdatabase.codeplex.com](https://chinookdatabase.codeplex.com/downloads/get/557773)
 y extraer archivo `Chinook_Sqlite.sqlite` al directorio db de la aplicación  
- Agregar dependencia `"EntityFramework.Sqlite.Design": "7.0.0-rc1-final",` y ejecutar `dnu restore` 
- Ejecutar comando para generar el modelo de entidades
```
dnx ef dbcontext scaffold "Data source=C:\Work\MusicStoreWebApp\db\Chinook_Sqlite.sqlite" EntityFramework.Sqlite -c ChinookDbContext -o Entities
```
- Registrar ChinookDbContext (Startup.cs)
```
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddEntityFramework()
        .AddSqlite()
        .AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlite(Configuration["Data:DefaultConnection:ConnectionString"]))
        // Registrar ChinookDbContext  
        .AddDbContext<ChinookDbContext>();
    ...
}
```

## Listado de canciones (MVC y ViewBag)
- ViewModel
```
public class MusicTrack
{       
    public long TrackId { get; set; }
    public string TrackName { get; set; }
    
    public string Artist { get; set; }
    
    public string Album { get; set; }        
}
```

- Nuevo controlador MVC
```  
yo aspnet:MvcController MusicStoreController
```
```
public class MusicStoreController : Controller
{
    public MusicStoreController(ChinookDbContext dbContext)
    {
        DbContext = dbContext;
    }
    
    protected ChinookDbContext DbContext { get; private set; }
    
    // GET: /<controller>/
    public IActionResult Index()
    {
        ViewBag.Tracks = DbContext.Track.Select(t => new MusicTrack 
        {
            Artist = t.Album.Artist.Name,  
            Album = t.Album.Title,
            TrackId = t.TrackId, 
            TrackName = t.Name
        });             
        return View();
    }
}
```
- Vista Index.cshtml
```  
yo aspnet:MvcView MusicStore\Index
```
```
@{
     ViewBag.Title = "Listado de canciones";
}
<h1>Listado de canciones</h1>
<table class="table table-striped table-bordered">
    <thead>
        <tr>
            <th>Artista</th>
            <th>Album</th>
            <th>Canción</th>
        </tr>
    </thead>
     
    @foreach(var track in ViewBag.Tracks){                
        <tr>
            <td>@track.Artist</td>
            <td>@track.Album</td>
            <td>@track.TrackName</td>
        </tr>
    }       
</table>
```

## Selección de canciones (carrito de la compra)

  - Vista (nueva columna con las acciones)
```
    <td>
        @if(!@track.InCart) {
            <a class="btn btn-primary" asp-controller="MusicStore" asp-action="AddToCart" asp-route-id="@track.TrackId">La quiero</a>
        }
        else {
            <a class="btn btn-default" asp-controller="MusicStore" asp-action="RemoveFromCart" asp-route-id="@track.TrackId">No la quiero</a>                    
        }
    </td>
```

- ViewModel (nueva propiedad indicando si la canción ha sido seleccionada)
```
 public class MusicTrack
{
    ...       
    public bool InCart { get; set; }
}
``` 
- Contrato del servicio que gestionará el carrito de la compra (IShoppingCartService)
```
public interface IShoppingCartService
{
    string NewCart();   
    IQueryable<long> GetItems(string cartId);
    bool HasItem(string cartId, long itemId);
    void AddItem(string cartId, long itemId);
    void RemoveItem(string cartId, long itemId);
}
```  
- Modificaciones en el Controlador MVC:
  - Inyectar servicio de carrito de la compra por el constructor
  - Modificar acción Index para incluir la información de si la canción está
  en el carrito de la compra o no 
  - Método para gestionar el identificador del carrito de la compra con Cookie
  - Acciones `AddToCart`, `RemoveFromCart` para añadir y eliminar una canción al carrito
```
    public class MusicStoreController : Controller
    {
        // Nuevo constructor
        public MusicStoreController(ChinookDbContext dbContext, IShoppingCartService shoppingCartService)
        {
            DbContext = dbContext;
            ShoppingCartService = shoppingCartService;            
        }
        
        // Propiedad con el servicio de carrito de compra
        protected IShoppingCartService ShoppingCartService { get; private set; }
            
        // Gestión del identificador del carrito con Cookie     
        string _shopingCartId;
        string ShopingCartId
        {
            get 
            {
                if (_shopingCartId == null)
                {
                    var cookie = Request.Cookies["ShopingCartId"];
                    if (cookie.Count > 0)
                    {
                        _shopingCartId = cookie.FirstOrDefault();
                    }
                    else 
                    {
                        _shopingCartId = ShoppingCartService.NewCart();                        
                        Response.Cookies.Append("ShopingCartId", _shopingCartId);
                    }
                }
                return _shopingCartId;
            }
        }
        
        // Nueva implementación de la acción Index
        public IActionResult Index()
        {
            ViewBag.Tracks = DbContext.Track.Select(t => new MusicTrack 
            {
                Artist = t.Album.Artist.Name,  
                Album = t.Album.Title,
                TrackId = t.TrackId, 
                TrackName = t.Name,
                InCart = ShoppingCartService.HasItem(ShopingCartId, t.TrackId)        
            });             
            return View();
        }       

        // Acciones que añaden/eliminan una canción del carrito
        public IActionResult AddToCart(long id)
        {           
            ShoppingCartService.AddItem(ShopingCartId, id);
            
            return RedirectToAction("Index");
        }       
        public IActionResult RemoveFromCart(long id)
        {
            ShoppingCartService.RemoveItem(ShopingCartId, id);
            
            return RedirectToAction("Index");
        }              
    }  
```    
- Implementación del servicio de carrito de la compra almacenado en memoria (MemoryStoredShopingCartService)
```
    public class MemoryStoredShoppingCartService: IShoppingCartService
    {
        //TODO: Usar ConcurentDiccionary 
        Dictionary<string, IList<long>> storage = new Dictionary<string, IList<long>>();       
              
        public string NewCart()
        {
            string cartId = Guid.NewGuid().ToString("N");
            storage.Add(cartId, new List<long>());
            return cartId;
        }
        public IQueryable<long> GetItems(string cartId)
        {
            IList<long> items;
            if (storage.TryGetValue(cartId, out items))
            {
                return items.AsQueryable();
            }
            else 
            {
                throw new InvalidOperationException($"No se ha encontrado el carrito con id={cartId}");
            }
        }
        public bool HasItem(string cartId, long itemId)
        {
            IList<long> items;
            if (storage.TryGetValue(cartId, out items))
            {
                return items.Contains(itemId);
            }
            else 
            {
                throw new InvalidOperationException($"No se ha encontrado el carrito con id={cartId}");
            }
        }
        public void AddItem(string cartId, long itemId)
        {
            IList<long> items;
            if (storage.TryGetValue(cartId, out items))
            {
                items.Add(itemId);
            }
            else 
            {
                throw new InvalidOperationException($"No se ha encontrado el carrito con id={cartId}");
            }
        }
        public void RemoveItem(string cartId, long itemId)
        {
            IList<long> items;
            if (storage.TryGetValue(cartId, out items))
            {
                items.Remove(itemId);
            }
            else 
            {
                throw new InvalidOperationException($"No se ha encontrado el carrito con id={cartId}");
            }
        }
    }
```
  
## Carrito de la compra y la Sesión en ASP.NET Core
http://benjii.me/2015/07/using-sessions-and-httpcontext-in-aspnet5-and-mvc6/

- Añadir la referencia `Microsoft.AspNet.Session`
- Configurar el uso de sesiones en el pipeline de OWIN añadiendo las llamadas a `AddSession()` y `AddCaching()` 
en el método `ConfigureServices(IServiceCollection services)` del archivo `startup.cs` 
debajo de la llamada a `AddMvc()` 
```
// Add MVC services to the services container.
services.AddMvc();
services.AddCaching(); // Adds a default in-memory implementation of     IDistributedCache
services.AddSession();
```
- En ese mismo archivo, el método `Configure`, añadir una llamada a `UseSession()` justo antes la llamada a `UseMvc()`
```
// IMPORTANT: This session call MUST go before UseMvc()
app.UseSession();
```
- Por último, modificar la propiedad `ShopingCartId` de `MusicStoreController` para guardar 
el identificador del carrito de la compra en la sesión en vez de en la cookie 
```
string ShopingCartId
{
    get 
    {
        if (_shopingCartId == null)
        {
            _shopingCartId = HttpContext.Session.GetString("ShopingCartId");
            if (_shopingCartId == null)
            {
                _shopingCartId = ShoppingCartService.NewCart();                        
                HttpContext.Session.SetString("ShopingCartId", _shopingCartId);
            }
        }
        return _shopingCartId;
    }
}
```


## API REST

### Canciones
- Controlador de Web API
```
    [Route("api/[controller]")]
    public class MusicTracksController : Controller
    {
        public MusicTracksController(ChinookDbContext dbContext)
        {
            DbContext = dbContext;
        }      
        
        protected ChinookDbContext DbContext { get; private set; }
        
    }
```
- GET /api/tracks --> Lista de canciones
```
    [HttpGet]
    public IEnumerable<MusicTrack> GetAll()
    {
        return DbContext.Track.Select(t => new MusicTrack 
        {
            Artist = t.Album.Artist.Name,  
            Album = t.Album.Title,
            TrackId = t.TrackId, 
            TrackName = t.Name
        });             
    }
```

- GET /api/tracks/:id --> Detalle de una canción
```
//TODO: 
```

### Carrito 
- POST /api/carts --> Iniciar un carrito de la compra (o registarlo completo)
```
//TODO: 
```

- PUT /api/carts/:id/items --> Añadir una canción al carrito de la compra 
```
//TODO: 
```

- DELETE /api/carts/:id/items --> Eliminar una canción del carrito de la compra
```
//TODO: 
```

## Vista con AngularJs y API REST 
- IndexAngular.cshtml
```
@{
    ViewBag.Title = "Listado de canciones";
}
<h1>Listado de canciones</h1>
<table class="table table-striped table-bordered" ng-app="app" ng-controller="musicStoreCtrl as vm">
    <thead>
        <tr>
            <th>Artista</th>
            <th>Album</th>
            <th>Canción</th>
            <th></th>
        </tr>
    </thead>
    
    <tr ng-repeat="track in vm.tracks">
        <td>{{track.Artist}}</td>
        <td>{{track.Album}}</td>
        <td>{{track.TrackName}}</td>
        <td>
            <a ng-if="!vm.IsInCart(track.TrackId)" class="btn btn-primary" ng-click="vm.AddToCart(track.TrackId)">La quiero</a>
            <a ng-if="vm.IsInCart(track.TrackId)" class="btn btn-default" ng-click="vm.RemoveFromCart(track.TrackId)">No la quiero</a>                    
        </td>
    </tr>
</table>
```
- site.js
```
angular
    .module('app', [])
        .controller('musicStoreCtrl', ['$http', '$rootScope', '$timeout', function($http, $rootScope, $timeout){
            var vm = this;
            vm.shoppingCart = [];
            
            $http.get('/api/musictracks').then(function(response){
                vm.tracks = response.data;
            });
            
            vm.IsInCart = function(trackId) {
                return vm.shoppingCart.indexOf(trackId) >= 0;
            }
            
            vm.AddToCart = function(trackId) {
                vm.shoppingCart.push(trackId);
            }
            
            vm.RemoveFromCart = function(trackId) {
                vm.shoppingCart.splice(vm.shoppingCart.indexOf(trackId));                
            }
            
        }]);
```
- _Layout.cshtml
```
<environment names="Development">
    ...
    <script src="~/lib/angular/angular.js"></script>
    <script src="~/js/site.js"></script>
</environment>
<environment names="Staging,Production">
    ...
    <script src="~/lib/angular/angular.min.js" asp-append-version="true"></script>
    <script src="~/js/site.min.js" asp-append-version="true"></script>
</environment>
```
## Cosas que se han quedado en el tintero 
- Seguridad (Autenticación/Autorización)
- Configuración
- Gestión de la sesión y caché distribuido
- TagHelpers
- ..., refactoring, refactoring, refactoring 




