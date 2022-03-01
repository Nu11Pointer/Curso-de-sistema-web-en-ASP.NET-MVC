# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 22

# Mostrar Imagen Al Editar

Para esta funcionalidad añadiremos una petición mediante Ajax a nuestra acción ```ImagenProducto```, a la cual le pasaremos el id del producto al cual estemos solicitando la imagen y simplemente añadiremos como origen del contenido a la ruta devuelta, de la siguiente forma.

```js
jQuery.ajax({
    url: '@Url.Action("ImagenProducto", "Mantenedor")',
    type: "POST",
    data: JSON.stringify({ id: json.IdProducto }),
    datatype: "json",
    contentType: "application/json; charset=utf-8",
    success: function (data) {

        $("#img_producto").LoadingOverlay("hide");
        if (data.conversion) {
            $("#img_producto").attr({ "src": "data:image/" + data.extension + ";base64," + data.textoBase64})
        }
    },
    error: function (error) {

        $("#img_producto").LoadingOverlay("hide");
        $("#mensajeError").show();
        $("#mensajeError").text("Error al mostrar imagen");
    },
    beforeSend: function () {

        $("#img_producto").LoadingOverlay("show");
    }

})

```

# Arreglando Eliminar Producto

Simplemente hay que reemplazar algunas cadenas y nombres de variables tal como se muestra en la sig imagen:

![image](https://user-images.githubusercontent.com/59342976/156220515-a6ce250f-e41f-458f-af46-7570bbc91879.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte23)
