# Curso de sistema web en ASP.NET MVC + SQL Server - Parte 03

## ¿Qué son los bundles?

Cuando creamos el proyecto en la plantilla **_Layaout.cshtml** una de las cosas que resaltan a la vista son los **Styles.Render y Script.Render** en lugar de incluir etiquetas **link** para css o **script** para javascript. El parámetro que se pasa por parámetro es el nombre del bundle a renderizar.

![image](https://user-images.githubusercontent.com/59342976/152063080-c28e228c-3a98-4dcb-be2b-e881e50bacdb.png)

Los bundles están definidos en el archivo **App_start/BundleConfig** en el metodo **RegisterBundles**

```c#
public static void RegisterBundles(BundleCollection bundles)
{
    namespace CapaPresentacionAdmin
{
    public class BundleConfig
    {
        public static void RegisterBundles(BundleCollection bundles)
        {
            bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
                        "~/Scripts/jquery-{version}.js"));
            bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(
                        "~/Scripts/jquery.validate*"));
            bundles.Add(new ScriptBundle("~/bundles/modernizr").Include(
                        "~/Scripts/modernizr-*"));
            bundles.Add(new ScriptBundle("~/bundles/bootstrap").Include(
                      "~/Scripts/bootstrap.js"));
            bundles.Add(new StyleBundle("~/Content/css").Include(
                      "~/Content/bootstrap.css",
                      "~/Content/site.css"));
        }
    }
}
}
```

Dicho metodo es llamado desde **Application_Start**.

```c#
public class MvcApplication : System.Web.HttpApplication
{
	protected void Application_Start()
	{
		AreaRegistration.RegisterAllAreas();
		FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
		RouteConfig.RegisterRoutes(RouteTable.Routes);
		BundleConfig.RegisterBundles(BundleTable.Bundles); // <----
	}
}

```

La ventaja de usar bundles es la facil optimización a las peticiones al servidor que proporciona, para mas información puedes ir a los s

* [Bundling and Minification](https://docs.microsoft.com/en-us/aspnet/mvc/overview/performance/bundling-and-minification)
* [Bundles en ASP.NET MVC4](https://geeks.ms/etomas/2012/07/30/bundles-en-asp-net-mvc4/#:~:text=Los%20bundles%20es%20el%20mecanismo,que%20est%C3%A1n%20relacionados%20entre%20ellos.&text=En%20lugar%20de%20incluir%20directamente,Render%20y%20Scripts.)

## Plantillas Bootstrap

Utilizaremos unas plantillas de Bootstrap utilizaremos el siguiente enlance: [Startbootstrap](https://startbootstrap.com/?showPro=false)

![image](https://user-images.githubusercontent.com/59342976/152060555-d24c4552-f308-4661-a382-581ea5bc79c4.png)

Utilizaremos las siguiente tres plantillas:

* [SB Admin](https://startbootstrap.com/template/sb-admin)
* [Shop Homepage](https://startbootstrap.com/template/shop-homepage)
* [Shop Item](https://startbootstrap.com/template/shop-item)

## Configurar Bootstrap 5

Nuestro proyecto no cuenta con la versión 5 de Bootstrap sino con la número 3. Para configurarla tendremos que ir al gestor de paquetes nuget, simplemente tendremos que actualizar ambos proyectos.

![image](https://user-images.githubusercontent.com/59342976/152084354-c10626f6-bfe8-402c-9ca6-34138e1f5bd3.png)

## Configurar BundleConfig

Tendremos que hacer unos cambios menores a nuestros Bundles, de tal forma que nos quede de la siguiente manera:

```c#
using System.Web;
using System.Web.Optimization;

namespace CapaPresentacionAdmin
{
    public class BundleConfig
    {
        public static void RegisterBundles(BundleCollection bundles)
        {
            bundles.Add(new Bundle("~/bundles/jquery").Include(
                        "~/Scripts/jquery-{version}.js"));

            bundles.Add(new Bundle("~/bundles/bootstrap").Include(
                      "~/Scripts/bootstrap.bundle.js"));
		      
            bundles.Add(new StyleBundle("~/Content/css").Include("~/Content/site.css"));
        }
    }
}
```

## Integración de la nueva plantilla

Nos dirigiremos a **Views/Shared/_Layaout.cshtml** y realizaremos la siguientes modificaciones:

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - Mi aplicación ASP.NET</title>
    @Styles.Render("~/Content/css")
    @*@Scripts.Render("~/bundles/modernizr")*@
</head>
<body>
    <nav class="sb-topnav navbar navbar-expand navbar-dark bg-dark">
        <!-- Navbar Brand-->
        <a class="navbar-brand ps-3" href="index.html">Start Bootstrap</a>
        <!-- Sidebar Toggle-->
        <button class="btn btn-link btn-sm order-1 order-lg-0 me-4 me-lg-0" id="sidebarToggle" href="#!"><i class="fas fa-bars"></i></button>
        <!-- Navbar Search-->
        <form class="d-none d-md-inline-block form-inline ms-auto me-0 me-md-3 my-2 my-md-0">
            <div class="input-group">
                <input class="form-control" type="text" placeholder="Search for..." aria-label="Search for..." aria-describedby="btnNavbarSearch" />
                <button class="btn btn-primary" id="btnNavbarSearch" type="button"><i class="fas fa-search"></i></button>
            </div>
        </form>
        <!-- Navbar-->
        <ul class="navbar-nav ms-auto ms-md-0 me-3 me-lg-4">
            <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" id="navbarDropdown" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false"><i class="fas fa-user fa-fw"></i></a>
                <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="navbarDropdown">
                    <li><a class="dropdown-item" href="#!">Settings</a></li>
                    <li><a class="dropdown-item" href="#!">Activity Log</a></li>
                    <li><hr class="dropdown-divider" /></li>
                    <li><a class="dropdown-item" href="#!">Logout</a></li>
                </ul>
            </li>
        </ul>
    </nav>
    <div id="layoutSidenav">
        <div id="layoutSidenav_nav">
            <nav class="sb-sidenav accordion sb-sidenav-dark" id="sidenavAccordion">
                <div class="sb-sidenav-menu">
                    <div class="nav">
                        <div class="sb-sidenav-menu-heading">Core</div>
                        <a class="nav-link" href="index.html">
                            <div class="sb-nav-link-icon"><i class="fas fa-tachometer-alt"></i></div>
                            Dashboard
                        </a>
                        <div class="sb-sidenav-menu-heading">Interface</div>
                        <a class="nav-link collapsed" href="#" data-bs-toggle="collapse" data-bs-target="#collapseLayouts" aria-expanded="false" aria-controls="collapseLayouts">
                            <div class="sb-nav-link-icon"><i class="fas fa-columns"></i></div>
                            Layouts
                            <div class="sb-sidenav-collapse-arrow"><i class="fas fa-angle-down"></i></div>
                        </a>
                        <div class="collapse" id="collapseLayouts" aria-labelledby="headingOne" data-bs-parent="#sidenavAccordion">
                            <nav class="sb-sidenav-menu-nested nav">
                                <a class="nav-link" href="layout-static.html">Static Navigation</a>
                                <a class="nav-link" href="layout-sidenav-light.html">Light Sidenav</a>
                            </nav>
                        </div>
                        <a class="nav-link collapsed" href="#" data-bs-toggle="collapse" data-bs-target="#collapsePages" aria-expanded="false" aria-controls="collapsePages">
                            <div class="sb-nav-link-icon"><i class="fas fa-book-open"></i></div>
                            Pages
                            <div class="sb-sidenav-collapse-arrow"><i class="fas fa-angle-down"></i></div>
                        </a>
                        <div class="collapse" id="collapsePages" aria-labelledby="headingTwo" data-bs-parent="#sidenavAccordion">
                            <nav class="sb-sidenav-menu-nested nav accordion" id="sidenavAccordionPages">
                                <a class="nav-link collapsed" href="#" data-bs-toggle="collapse" data-bs-target="#pagesCollapseAuth" aria-expanded="false" aria-controls="pagesCollapseAuth">
                                    Authentication
                                    <div class="sb-sidenav-collapse-arrow"><i class="fas fa-angle-down"></i></div>
                                </a>
                                <div class="collapse" id="pagesCollapseAuth" aria-labelledby="headingOne" data-bs-parent="#sidenavAccordionPages">
                                    <nav class="sb-sidenav-menu-nested nav">
                                        <a class="nav-link" href="login.html">Login</a>
                                        <a class="nav-link" href="register.html">Register</a>
                                        <a class="nav-link" href="password.html">Forgot Password</a>
                                    </nav>
                                </div>
                                <a class="nav-link collapsed" href="#" data-bs-toggle="collapse" data-bs-target="#pagesCollapseError" aria-expanded="false" aria-controls="pagesCollapseError">
                                    Error
                                    <div class="sb-sidenav-collapse-arrow"><i class="fas fa-angle-down"></i></div>
                                </a>
                                <div class="collapse" id="pagesCollapseError" aria-labelledby="headingOne" data-bs-parent="#sidenavAccordionPages">
                                    <nav class="sb-sidenav-menu-nested nav">
                                        <a class="nav-link" href="401.html">401 Page</a>
                                        <a class="nav-link" href="404.html">404 Page</a>
                                        <a class="nav-link" href="500.html">500 Page</a>
                                    </nav>
                                </div>
                            </nav>
                        </div>
                        <div class="sb-sidenav-menu-heading">Addons</div>
                        <a class="nav-link" href="charts.html">
                            <div class="sb-nav-link-icon"><i class="fas fa-chart-area"></i></div>
                            Charts
                        </a>
                        <a class="nav-link" href="tables.html">
                            <div class="sb-nav-link-icon"><i class="fas fa-table"></i></div>
                            Tables
                        </a>
                    </div>
                </div>
                <div class="sb-sidenav-footer">
                    <div class="small">Logged in as:</div>
                    Start Bootstrap
                </div>
            </nav>
        </div>
        <div id="layoutSidenav_content">
            <main>
                <div class="container-fluid px-4">
                    @RenderBody()
                </div>
            </main>
            <footer class="py-4 bg-light mt-auto">
                <div class="container-fluid px-4">
                    <div class="d-flex align-items-center justify-content-between small">
                        <div class="text-muted">Copyright &copy; Your Website 2021</div>
                        <div>
                            <a href="#">Privacy Policy</a>
                            &middot;
                            <a href="#">Terms &amp; Conditions</a>
                        </div>
                    </div>
                </div>
            </footer>
        </div>
    </div>

    @Scripts.Render("~/bundles/jquery")
    @Scripts.Render("~/bundles/bootstrap")
    @RenderSection("scripts", required: false)
</body>
</html>
```

Copiaremos y pegaremos lo que hay dentro del fichero con ruta **startbootstrap-sb-admin-gh-pages\css\styles.css** en el fichero con ruta **Content\Site.css**

![image](https://user-images.githubusercontent.com/59342976/152089524-eefec3a4-7637-4e36-8143-9da4a7f3ebce.png)

![image](https://user-images.githubusercontent.com/59342976/152089579-df2b1879-264e-489c-8bd9-1768f90a4941.png)

También agregaremos el archivo script.js que tiene la ruta **startbootstrap-sb-admin-gh-pages\js\scripts.js** en la carpeta script de nuestro proyecto.

![image](https://user-images.githubusercontent.com/59342976/152090345-b33591e2-c107-462d-a976-20ccf22ee9f5.png)

También para llamarlo tendremos que añadirlo a la configuración de Bundles.

```c#
bundles.Add(new Bundle("~/bundles/complementos").Include(
                        "~/Scripts/script.js"));
```

También tendremos que añadir el script a nuestro **_Layaout.cshtml**

```c#
@Scripts.Render("~/bundles/complementos")
```
Para el tema de iconos tendremos que instalar el paquete Font.Awesome

![image](https://user-images.githubusercontent.com/59342976/152095152-0b0a1025-2e31-4fc1-9ba5-d890e5783309.png)

Luego modificaremos nuestro Bundles complementos de esta forma:

```c#
bundles.Add(new Bundle("~/bundles/complementos").Include(
                      "~/Scripts/fontawesome/all.min.js",
                      "~/Scripts/scripts.js"));
```

Y este es nuestro resultado

![image](https://user-images.githubusercontent.com/59342976/152095849-77209912-58a2-4e7d-9bb5-98a31bc654b7.png)

![image](https://user-images.githubusercontent.com/59342976/152095872-0bcd1707-f2b8-4de2-9777-56835e6fbce2.png)

