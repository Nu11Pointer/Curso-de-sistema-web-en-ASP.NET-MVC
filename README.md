# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 20

# Funcion Para Convertir a Base64

Esta función tiene como proposito convertir los bytes de nuestra imagen en una cadena en base 64 como su nombre indica, esto nos permitirá mostrar nuestra imagen.

```c#
public static string ConvertirBase64(string ruta, out bool conversion)
{
    string textoBase64 = string.Empty;
    conversion = true;

    try
    {
        byte[] bytes = File.ReadAllBytes(ruta);
        textoBase64 = Convert.ToBase64String(bytes);
    }
    catch (Exception)
    {
        conversion = false;
    }

    return textoBase64;
}

```

# Accion Imagen Producto

Esto nos permitira llamar a la función de CN_Recursos y obtener nuestra imagen en el formato requerido.

```c#
[HttpPost]
public JsonResult ImagenProducto(int id)
{
    bool conversion;
    Producto oproducto = new CN_Producto().Listar().Where(p => p.IdProducto == id).FirstOrDefault();

    string textoBase64 = CN_Recursos.ConvertirBase64(Path.Combine(oproducto.RutaImagen, oproducto.NombreImagen), out conversion);

    return Json(new
    {
        conversion = conversion,
        textoBase64 = textoBase64,
        extension = Path.GetExtension(oproducto.NombreImagen)
    },
    JsonRequestBehavior.AllowGet);
}

```

# Vista Producto

Se modificó la vista segun lo requerido para hacer uso de la sección de productos respecto al apartado grafico, todavía faltará modificar los scripts correspondientes.

```cshtml

@{
    ViewBag.Title = "Producto";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h1 class="mt-4">Producto</h1>

<ol class="breadcrumb mb-4">
    <li class="breadcrumb-item"><a href="@Url.Action("Producto", "Mantenedor")">Mantenimiento</a></li>
    <li class="breadcrumb-item active">Producto</li>
</ol>

@*Form Tabla*@
<div class="card">

    @*Titulo*@
    <div class="card-header">
        <i class="fas fa-boxes me-1"></i> Lista de Marcas
    </div>

    @*Cuerpo*@
    <div class="card-body">

        @*Botones*@
        <div class="row">
            <div class="col-12">
                <button type="button" class="btn btn-success" onclick="abrirModal()">Crear Nuevo</button>
            </div>
        </div>

        <hr />

        @*Tabla*@
        <table id="tabla" class="display cell-border" style="width: 100%">

            <thead>
                <tr>
                    <th>Nombre</th>
                    <th>Descripcion</th>
                    <th>Marca</th>
                    <th>Categoria</th>
                    <th>Precio</th>
                    <th>Stock</th>
                    <th>Activo</th>
                    <th></th>
                </tr>
            </thead>

            <tbody>
            </tbody>

        </table>

    </div>

</div>

<!-- Modal -->
<div class="modal fade" id="FormModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true"
     data-bs-backdrop="static">

    <div class="modal-dialog modal-xl">

        @*Contenido*@
        <div class="modal-content">

            @*Cabecera*@
            <div class="modal-header bg-dark text-white">

                <h5 class="modal-title" id="exampleModalLabel">Producto</h5>

                <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"
                        aria-label="Close"></button>

            </div>

            @*Cuerpo*@
            <div class="modal-body">

                <input id="txtid" type="hidden" value="0" />

                @*Inputs*@

                <form id="contenedor" class="row">
                    @* Imagen y Selector *@
                    <div class="col-sm-3">
                        <div class="mb-2">
                            <img id="img_producto" height="197" width="200" class="border rounded mx-auto d-block img-fluid" />
                        </div>

                        <div class="mb-2">
                            <input class="form-control" type="file" id="fileProducto" accept="image/png, image/jpg, image/jpeg" onchange="" />
                        </div>
                    </div>

                    <div class="col-sm-3">
                        @* Nombre *@
                        <div class="mb-3">
                            <label class="form-label">Nombre</label>
                            <input type="text" class="form-control" id="txtnombre" name="nombre" autocomplete="off"/>
                        </div>
                        @* Descripcion *@
                        <div class="mb-3">
                            <label class="form-label">Descripcion</label>
                            <textarea type="text" class="form-control" id="txtdescripcion" name="descripcion" style="height:125px;resize:none" autocomplete="off"></textarea>
                        </div>
                    </div>

                    <div class="col-sm-3">
                        @* Marca *@
                        <div class="mb-3">
                            <label class="form-label">Marca</label>
                            <select id="cbomarca" class="form-select">
                            </select>
                        </div>
                        @* Categoria *@
                        <div class="mb-3">
                            <label class="form-label">Categoria</label>
                            <select id="cbocategoria" class="form-select">
                            </select>
                        </div>
                        @* Precio *@
                        <div class="mb-3">
                            <label class="form-label">Precio</label>
                            <input type="text" class="form-control" id="txtprecio" name="precio" autocomplete="off"/>
                        </div>
                    </div>

                    <div class="col-sm-3">
                        @* Stock *@
                        <div class="mb-3">
                            <label class="form-label">Stock</label>
                            <input type="number" class="form-control" id="txtstock" name="stock" autocomplete="off"/>
                        </div>
                        @* Activo *@
                        <div class="mb-3">
                            <label for="cboactivo" class="form-label">Activo</label>
                            <select id="cboactivo" class="form-select">
                                <option value="1">Si</option>
                                <option value="0">No</option>
                            </select>
                        </div>
                    </div>
                </form>

                @*Mensaje Error*@
                <div class="row mt-2">
                    <div class="col-12">
                        <div id="mensajeError" class="alert alert-danger" role="alert">
                            A simple danger alert—check it out!
                        </div>
                    </div>
                </div>

            </div>

            @*Pie de Modal*@
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                <button type="button" class="btn btn-primary" onclick="Guardar()">Guardar</button>
            </div>

        </div>

    </div>

</div>

@section scripts{
    <script>

    var tabladata;
    var filaSeleccionada;

    tabladata = $("#tabla").DataTable({
        responsive: true,
        ordering: false,
        "ajax": {
            url: '@Url.Action("ListarMarca", "Mantenedor")',
            type: "GET",
            dataType: "json"
        },
        "columns": [
            { "data": "Descripcion" },
            {
                "data": "Activo", "render": function (valor) {
                    if (valor)
                        return '<span class="badge bg-success">Si</span>';
                    else
                        return '<span class="badge bg-danger">No</span>';
                }
            },
            {
                "defaultContent": '<button type="button" class="btn btn-primary btn-sm btn-editar"><i class="fas fa-pen"></i></button>' +
                    '<button type="button" class="btn btn-danger btn-sm ms-2 btn-eliminar"><i class="fas fa-trash"></i></button>',
                "orderable": false,
                "searchable": false,
                "width": "90px"
            }
        ],
        "language": {
            "url": "https://cdn.datatables.net/plug-ins/1.11.4/i18n/es_es.json"
        }
    });

    function abrirModal(json) {

        $("#txtid").val(0);
        $("#txtdescripcion").val("");
        $("#cboactivo").val(1);

        $("#mensajeError").hide();

        if (json != null) {
            $("#txtid").val(json.IdMarca);
            $("#txtdescripcion").val(json.Descripcion);
            $("#cboactivo").val(json.Activo ? 1 : 0);
        }

        $("#FormModal").modal("show");
    }

    $("#tabla tbody").on("click", '.btn-editar', function () {

        filaSeleccionada = $(this).closest("tr");
        var data = tabladata.row(filaSeleccionada).data();
        abrirModal(data)
    })

    $("#tabla tbody").on("click", '.btn-eliminar', function () {

        var marcaseleccionada = $(this).closest("tr");
        var data = tabladata.row(marcaseleccionada).data();

        swal({
            title: "¿Está seguro?",
            text: "¿Desea eliminar la marca?",
            type: "warning",
            showCancelButton: true,
            confirmButtonClass: "btn-primary",
            confirmButtonText: "Si",
            cancelButtonText: "No",
            closeOnConfirm: true
        },
            function () {

                jQuery.ajax({
                    url: '@Url.Action("EliminarMarca", "Mantenedor")',
                    type: "POST",
                    data: JSON.stringify({ id: data.IdMarca }),
                    dataType: "json",
                    contentType: "application/json; charset=utf-8",
                    success: function (data) {

                        if (data.resultado) {

                            tabladata.row(marcaseleccionada).remove().draw();
                        }
                        else {

                            swal("No se puedo eliminar la marca.", data.mensaje, "error");
                        }
                    },
                    error: function (error) {
                        console.log(error);
                    },
                    beforeSend: function () { }
                });

            });
    })

    function Guardar() {

        var Marca = {
            IdMarca: $("#txtid").val(),
            Descripcion: $("#txtdescripcion").val(),
            Activo: $("#cboactivo").val() == 1 ? true : false
        }

        jQuery.ajax({
            url: '@Url.Action("GuardarMarca", "Mantenedor")',
            type: "POST",
            data: JSON.stringify({ objeto: Marca }),
            dataType: "json",
            contentType: "application/json; charset=utf-8",
            success: function (data) {

                $(".modal-body").LoadingOverlay("hide");

                // Marca Nueva
                if (Marca.IdMarca == 0) {

                    if (data.resultado != 0) {
                        Marca.IdMarca = data.resultado;
                        tabladata.row.add(Marca).draw(false);
                        $("#FormModal").modal("hide");
                    }
                    else {
                        $("#mensajeError").text(data.mensaje);
                        $("#mensajeError").show();
                    }
                }
                // Marca Editar
                else {
                    if (data.resultado) {
                        tabladata.row(filaSeleccionada).data(Marca).draw(false);
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
        }

    </script>
}

```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte20)
