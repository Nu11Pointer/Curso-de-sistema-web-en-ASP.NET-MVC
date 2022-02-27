# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 21

# Monstrar Imagen

Esta función como su nombre lo indica actualizara la fuente de la etiqueta ```img_producto``` para mostrar la imagen que ha sido subida mediante el form-control de tipo file.

```html
<input class="form-control" type="file" id="fileProducto"
accept="image/png, image/jpg, image/jpeg" onchange="montrarImagen(this)" />

```

```js
function montrarImagen(input) {
    if (input.files) {
        var reader = new FileReader();

        reader.onload = function (e) {
            $("#img_producto").attr("src", e.target.result).width(200).height(197);
        }

        reader.readAsDataURL(input.files[0]);
    }
}

```

# Cargar Marcas

Es necesario que le carguemos las marcas que tenemos a nuestro formulario para ello tendremos que realizar una petición mediante JQuery con referencia a la acción ListarMarca del controlador Mantenedor. Si la petición fue exitosa podemos añadir etiquetas option mediante un ciclo for atravez de los datos obtenidos.

```js
jQuery.ajax({
    url: '@Url.Action("ListarMarca", "Mantenedor")',
    type: "GET",
    data: null,
    dataType: "json",
    contentType: "application/json; charset=utf-8",
    success: function (data) {

        $("<option>").attr({ "value": "0", "disabled": "true" }).text("Seleccionar").appendTo("#cbomarca");

        $.each(data.data, function (index, valor) {
            $("<option>").attr({ "value": valor.IdMarca }).text(valor.Descripcion).appendTo("#cbomarca");
        })
    },
    error: function (error) {
        console.log(error);
    }
});

```

# Cargar Categorias

De la misma manera que en el apartado anterior haremos esto para categorias.

```js
jQuery.ajax({
    url: '@Url.Action("ListarCategorias", "Mantenedor")',
    type: "GET",
    data: null,
    dataType: "json",
    contentType: "application/json; charset=utf-8",
    success: function (data) {

        $("<option>").attr({ "value": "0", "disabled": "true" }).text("Seleccionar").appendTo("#cbocategoria");

        $.each(data.data, function (index, valor) {
            $("<option>").attr({ "value": valor.IdCategoria }).text(valor.Descripcion).appendTo("#cbocategoria");
        })
    },
    error: function (error) {
        console.log(error);
    }
});

```

# Mostrar Productos

Podremos copiar y pegar el codigo de apartados anteriores, simplemente modificaremos las columnas.

```js
tabladata = $("#tabla").DataTable({
    responsive: true,
    ordering: false,
    "ajax": {
        url: '@Url.Action("ListarProducto", "Mantenedor")',
        type: "GET",
        dataType: "json"
    },
    "columns": [
        { "data": "Nombre" },
        { "data": "Descripcion" },
        { "data": "oMarca.Descripcion" },
        { "data": "oCategoria.Descripcion" },
        { "data": "Precio" },
        { "data": "Stock" },
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

```

# Abrir Modal

Al tener diferentes campos es necesario anexar y moficar los datos que estan siendo limpiados, tambien los que son pintados al momento de editar.

```js
function abrirModal(json) {

    $("#txtid").val(0);
    $("#img_producto").removeAttr("src");
    $("#fileProducto").val("");
    $("#txtnombre").val("");
    $("#txtdescripcion").val("");
    $("#cbomarca").val($("#cbomarca option:first").val());
    $("#cbocategoria").val($("#cbocategoria option:first").val());
    $("#txtprecio").val("");
    $("#txtstock").val("");
    $("#cboactivo").val(1);

    $("#mensajeError").hide();

    if (json != null) {
        $("#txtid").val(json.IdProducto);
        $("#txtnombre").val(json.Nombre);
        $("#txtdescripcion").val(json.Descripcion);
        $("#cbomarca").val(json.oMarca.IdMarca);
        $("#cbocategoria").val(json.oCategoria.IdCategoria);
        $("#txtprecio").val(json.Precio);
        $("#txtstock").val(json.Stock);
        $("#cboactivo").val(json.Activo ? 1 : 0);
    }

    $("#FormModal").modal("show");
}

```

# Guardar Producto

El cambio mas significativo y complejo es para esta funcionalidad ya que no estamos unicamente trabajando con jsons sino que aparte tendremos que trabajar con archivos binarios, es decir la imagen del producto. Es por eso que como petición utilizaremos un objeto de tipo FormData y apartir de ahí la logica será la misma que hemos trabajado.

```js
function Guardar() {

    var ImagenSeleccionada = $("#fileProducto")[0].files[0];

    var Producto = {
        IdProducto: $("#txtid").val(),
        Nombre: $("#txtnombre").val(),
        Descripcion: $("#txtdescripcion").val(),
        oMarca: {
            IdMarca: $("#cbomarca option:selected").val(),
            Descripcion: $("#cbomarca option:selected").text()
        },
        oCategoria: {
            IdCategoria: $("#cbocategoria option:selected").val(),
            Descripcion: $("#cbocategoria option:selected").text()
        },
        PrecioTexto: $("#txtprecio").val(),
        Precio: $("#txtprecio").val(),
        Stock: $("#txtstock").val(),
        Activo: $("#cboactivo").val() == 1 ? true : false
    }

    var request = new FormData();
    request.append("objeto", JSON.stringify(Producto));
    request.append("archivoImagen", ImagenSeleccionada);

    jQuery.ajax({
        url: '@Url.Action("GuardarProducto", "Mantenedor")',
        type: "POST",
        data: request,
        processData: false,
        contentType: false,
        success: function (data) {

            $(".modal-body").LoadingOverlay("hide");

            // Producto Nuevo
            if (Producto.IdProducto == 0) {

                if (data.idGenerado != 0) {
                    Producto.IdProducto = data.idGenerado;
                    tabladata.row.add(Producto).draw(false);
                    $("#FormModal").modal("hide");
                }
                else {
                    $("#mensajeError").text(data.mensaje);
                    $("#mensajeError").show();
                }
            }
            // Producto Editar
            else {
                if (data.operacion_exitosa) {
                    tabladata.row(filaSeleccionada).data(Producto).draw(false);
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

```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte21)
