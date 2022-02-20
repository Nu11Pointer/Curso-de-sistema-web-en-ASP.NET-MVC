# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 12

## Importando Librerias

Las librerias que vamos a importar serán [jQuery LoadingOverlay](https://gasparesganga.com/labs/jquery-loading-overlay/) y [SweetAlert for Bootstrap](https://lipis.github.io/bootstrap-sweetalert/)

## SweetAlert for Bootstrap

![image](https://user-images.githubusercontent.com/59342976/154823868-a572366a-d408-4c8c-b547-5845b46c52ee.png)

Esta libreria esta disponible desde el administrador de paquetes nugget así que simplemente buscaremos ```SweetAlert.Bootstrap``` e instalaremos.

### Correción de errores

![image](https://user-images.githubusercontent.com/59342976/154823952-5b200ff4-8bc2-4a1c-a0d2-ccf69f0f774c.png)

Esta librería trabaja con una versión vieja de bootstrap así que iremos al archivo ```.\CapaPresentacionAdmin\Scripts\sweetalert.min.js``` y cambiaremos el contenido de la propiedad **cancelButtonClass** por **"btn-dark"**

## jQuery LoadingOverlay

![image](https://user-images.githubusercontent.com/59342976/154824028-de79cb00-6dcc-4661-a811-a2e384391102.png)

Esta librería la encontraremos en [este enlace](https://cdn.jsdelivr.net/npm/gasparesganga-jquery-loading-overlay@2.1.7/dist/loadingoverlay.min.js) la nombraremos ```loadingoverlay.min.js``` lo colocaremos en una carpeta de nombre ```loadingoverlay``` dentro de la carpeta ```Scripts``` de nuestro proyecto ```CapaPresentaciónAdmin``` tal como aparece en la imagen anterior.

## Añadiendo los Bundles de jQuery LoadingOverlay y SweetAlert for Bootstrap

Una de las cosas de las cuales no podemos olvidarnos es de agregar los Bundles/Paquetes a nuestro proyecto ya que esto nos permitirá utilizarlos en todas las paginas de este. Empezemos por los scripts estos serán:

```c#
"~/Scripts/loadingoverlay/loadingoverlay.min.js",
"~/Scripts/sweetalert.min.js",

```

Y ahora los estilos.

```c#
"~/Content/sweetalert.css"

```

Al final este será el resultado de nuestro **BundleConfig**:

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
                      "~/Scripts/loadingoverlay/loadingoverlay.min.js",
                      "~/Scripts/sweetalert.min.js",
                      "~/Scripts/scripts.js"
                      ));

            bundles.Add(new Bundle("~/bundles/bootstrap").Include(
                      "~/Scripts/bootstrap.bundle.js"));

            bundles.Add(new StyleBundle("~/Content/css").Include(
                "~/Content/site.css",
                "~/Content/DataTables/css/jquery.dataTables.css",
                "~/Content/DataTables/css/responsive.dataTables.css",
                "~/Content/sweetalert.css"
                ));
        }
    }
}

```

## Utilizando jQuery LoadingOverlay

![image](https://user-images.githubusercontent.com/59342976/154824208-3eb5ac76-3339-4e57-9edb-fb936992f17a.png)

Dentro de la [Documentación de jQuery LoadingOverlay](https://gasparesganga.com/labs/jquery-loading-overlay/#examples) podemos encontrar ejemplos de scripts que podemos utilizar en nuestro proyecto, es bastante sencillo pero para la ocasión simplemente utilizaremos la siguiente propiedades:

* imageResizeFactor
* text
* size

```js
$(".modal-body").LoadingOverlay("show", {
	imageResizeFactor: 2,
	text: "Cargando...",
	size: 14
});

$(".modal-body").LoadingOverlay("hide");

```

La colocaremos en la función ```Guardar``` dentro de las propiedades de **jQuery.ajax** ```success```, ```error``` y ```beforeSend``` de esta manera:

```js
jQuery.ajax({
    url: '@Url.Action("GuardarUsuario", "Home")',
    type: "POST",
    data: JSON.stringify({ objeto: Usuario }),
    dataType: "json",
    contentType: "application/json; charset=utf-8",
    success: function (data) {

        $(".modal-body").LoadingOverlay("hide");
        // Usuario Nuevo
        if (Usuario.IdUsuario == 0) {

            if (data.resultado != 0) {
                Usuario.IdUsuario = data.resultado;
                tabladata.row.add(Usuario).draw(false);
                $("#FormModal").modal("hide");
            }
            else {
                $("#mensajeError").text(data.mensaje);
                $("#mensajeError").show();
            }
        }
        // Usuario Editar
        else {
            if (data.resultado) {
                tabladata.row(filaSeleccionada).data(Usuario).draw(false);
                filaSeleccionada = null;
                $("#FormModal").modal("hide");
            }
            else {
                $("#mensajeError").text(data.mensaje);
                $("#mensajeError").show();
            }
        }
    },
    error: function (error) {

        $(".modal-body").LoadingOverlay("hide");
        $("#mensajeError").text(error.responseText);
        $("#mensajeError").show();
    },
    beforeSend: function () {

        $(".modal-body").LoadingOverlay("show", {
            imageResizeFactor: 2,
            text: "Cargando...",
            size: 14
        });
    }
});

```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte12)
