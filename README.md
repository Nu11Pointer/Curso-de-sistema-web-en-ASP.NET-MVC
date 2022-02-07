# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 07

## Empleado DataTable

![image](https://user-images.githubusercontent.com/59342976/152704705-aca3d108-edd6-48e1-a365-601180a7527b.png)

Lo primero que tendremos que hacer es instalar **jquery.datatables** usando nuestro gestor de paquetes nugget. Y deberemos crear sus respectivas referencias en la "Union/Bundle" en el archivo **BundleCongif.cs**.

```c#
using System.Web.Optimization;

namespace CapaPresentacionAdmin
{
    public class BundleConfig
    {
        public static void RegisterBundles(BundleCollection bundles)
        {
            bundles.Add(new Bundle("~/bundles/jquery").Include(
                        "~/Scripts/jquery-{version}.js"));

            bundles.Add(new Bundle("~/bundles/complementos").Include(
                      "~/Scripts/fontawesome/all.min.js",
                      "~/Scripts/DataTables/jquery.dataTables.js",
                      "~/Scripts/DataTables/dataTables.responsive.js",
                      "~/Scripts/scripts.js"
                      ));

            bundles.Add(new Bundle("~/bundles/bootstrap").Include(
                      "~/Scripts/bootstrap.bundle.js"));

            bundles.Add(new StyleBundle("~/Content/css").Include(
                "~/Content/site.css",
                "~/Content/DataTables/css/jquery.dataTables.css",
                "~/Content/DataTables/css/responsive.dataTables.css"
                ));
        }
    }
}
```

Tambien deberemos cambiar la manera en devolvemos el listado de usuarios ya que DataTable recibira un json que tenga un atributo de nombre **data** es ahí donde colocaremos nuestra lista:

```c#
[HttpGet]
public JsonResult ListarUsuarios()
{
    List<Usuario> oLista = new CN_Usuarios().Listar();
    return Json(new { data = oLista }, JsonRequestBehavior.AllowGet);
}
```

Para finalizar agregaremos el siguiente codigo al script:

```html
@section scripts{
    <script>

        var tabladata;

        jQuery.ajax({
            url: '@Url.Action("ListarUsuarios", "Home")',
            type: "GET",
            dataType: "json",
            contentType: "application/json; charset=utf-8",
            success: function (data) {
                console.log(data)
            }
        })

        tabladata = $("#tabla").DataTable()
    </script>
}
```

![image](https://user-images.githubusercontent.com/59342976/152707190-3a832d5b-5dbf-42cb-9016-fa2a9d7f671a.png)

Ahora tendremos que agregar algunos parametros por ejemplo necesitamos indicarle a DataTable lo siguiente:

* La tabla tiene que ser reponsiva: ```responsive: true```
* No queremos que se ordenen las columnas: ```ordering: false```
* La petición de los datos con **Ajax**: ```"ajax": { url: '@Url.Action("ListarUsuarios", "Home")', type: "GET", dataType: "json" }```
* Nuestras Columnas (Las columnas deben tener el mismo nombre que en el Json que recibimos): ```"columns":[{ "data": "columna1" },...]```

Al final obtendremos lo siguiente:

```html
@section scripts{
    <script>

        var tabladata;

        tabladata = $("#tabla").DataTable({
            responsive: true,
            ordering: false,
            "ajax": {
                url: '@Url.Action("ListarUsuarios", "Home")',
                type: "GET",
                dataType: "json"
            },
            "columns": [
                { "data": "Nombres" },
                { "data": "Apellidos" },
                { "data": "Correo" },
                { "data": "Activo" }
            ]
        });
    </script>
}
```

Para personlizarlo un poco en vez de retornar ```True``` o ```False``` retornaremos un (Badge Bootsrap v5.0)[https://getbootstrap.com/docs/5.0/components/badge/] haciendo uso de la propiedad render y agregaremos una columna para los botones de las operaciones **Editar** y **Eliminar**. Obtendremos lo siguiente:

```html
@section scripts{
    <script>

        var tabladata;
	    
        tabladata = $("#tabla").DataTable({
            responsive: true,
            ordering: false,
            "ajax": {
                url: '@Url.Action("ListarUsuarios", "Home")',
                type: "GET",
                dataType: "json"
            },
            "columns": [
                { "data": "Nombres" },
                { "data": "Apellidos" },
                { "data": "Correo" },
                {
                    "data": "Activo", "render": function (valor) {
                        if (valor)
                            return '<span class="badge bg-success">Si</span>';
                        else
                            return '<span class="badge bg-danger">No</span>';
                    }
                },
                {
                    "defaultContent": '<button type="button" class="btn btn-primary btn-sm"><i class="fas fa-pen"></i></button>' +
                        '<button type="button" class="btn btn-danger btn-sm ms-2"><i class="fas fa-trash"></i></button>',
                    "orderable": false,
                    "searchable": false,
                    "width":"90px"
                }
            ]
        });
    </script>
}
```

![image](https://user-images.githubusercontent.com/59342976/152721818-c9d4d281-b8b4-42f1-a11d-ac2c4cb04777.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte07)
