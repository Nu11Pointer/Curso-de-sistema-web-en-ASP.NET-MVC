# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 22

# Validaciones en el formulario Producto

Para esta funcionalidad añadiremos la libreria mediante uso del administrador de paquetes nugger ```jquery.validate```. Esta referencia tendremos que agregar a nuestra configuración de paquetes de la siguiente forma:

```c#
bundles.Add(new Bundle("~/bundles/complementos").Include(
                      "~/Scripts/fontawesome/all.min.js",
                      "~/Scripts/DataTables/jquery.dataTables.js",
                      "~/Scripts/DataTables/dataTables.responsive.js",
                      "~/Scripts/loadingoverlay/loadingoverlay.min.js",
                      "~/Scripts/sweetalert.min.js",
                      "~/Scripts/jquery.validate.js",
                      "~/Scripts/scripts.js"
                      ));
```

Lo segundo a hacer es implementarlo de la cual requeriremos una regla de formato extra la cual creamos de la siguiente manera:

```js
jQuery.validator.addMethod("preciodecimal", function (value, element) {
        return this.optional(element) || /^\d{0,4}(\.\d{0,2})?$/i.test(value);
    }, "El formato correcto del precio es ##.##");
    
```

Y para finalizar requeriremos es especificar las reglas y los mensajes de error a utilizar en nuestro contenedor.

```js
$("#contenedor").validate({
    rules: {
        nombre: {
            required: true
        }
        ,
        descripcion: {
            required: true
        },
        precio: {
            required: true,
            preciodecimal: true
        },
        stock: {
            required: true,
            number: true
        }
    },
    messages: {
        nombre: "- El campo nombre es obligatorio."
        ,
        descripcion: "- El campo descripcion es obligatorio.",
        precio: { required: "- El campo precio es obligatorio.", preciodecimal: "El formato correcto del precio es ##.##." },
        stock: { required: "- El campo stock es obligatorio.", number: "- Debe ingresar solo numeros en el campo stock." }
    },
    errorElement: "div",
    errorLabelContainer: ".alert-danger"
});
    
```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte22)
