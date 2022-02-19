# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 10

## Alerts

![image](https://user-images.githubusercontent.com/59342976/154784582-60f712ee-bf32-4b41-8b7f-2afb6eb786d6.png)

Implementaremos algo que mejor que ```alert()``` lo haremos utilizando [Alerts · Bootstrap v5.0](https://getbootstrap.com/docs/5.0/components/alerts/) especificamente utilizaremos **danger alert** colocandola luego de nuestros imputs pero dentro del cuerpo del **Modal**.

```html
<div class="row mt-2">
	<div class="col-12">
		<div id="mensajeError" class="alert alert-danger" role="alert">
			A simple danger alert—check it out!
		</div>
	</div>
</div>

```

y no olvidemos de esconderlo cada vez que se abra el modal, colocaremos lo siguiente dentro de la funcion ```abrirModal```

```js
$("#mensajeError").hide();

```

## HomeController

### GuardarUsuario

Crearemos una acción para nuestro controlador **Home** la cual será de tipo **POST**, por parametro solicitará el usuario a guardar bien sea para registrarlo o editarlo, devolvera un **Json** con la siguiente estructura:

```json
{
   "resultado":0,
   "mensaje":""
}

```

La propiedad resultado hace referencia al ID del Usuario, este será cero si la petición no se completó exitosamente y mensaje hará referencia al error producido, en caso de que no hay errores el mensaje será vacio y el resultado será un numero diferente a cero.

```c#
[HttpPost]
public JsonResult GuardarUsuario(Usuario objeto)
{
	object resultado;
	string mensaje = string.Empty;
	if (objeto.IdUsuario == 0)
	{
		resultado = new CN_Usuarios().Registrar(objeto, out mensaje);
	}
	else
	{
		resultado = new CN_Usuarios().Editar(objeto, out mensaje);
	}
	return Json(new { resultado = resultado, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
}

```

## Guardar

Para realizar la petición utilizaremos **Ajax** con las siguiente propiedades:

* La url será manera dinamica con ```Url.Action``` indicando nuestra acción y controlador
* El tipo de petición será de tipo **POST** ya que estaremos enviando datos.
* La propiedad **data** será un **Json** el cual tiene la siguiente estructura:

```Json
{
	"Activo":true,
	"Apellidos":"",
	"Correo":"",
	"IdUsuario":"",
	"Nombres":"",
	"Reestablecer":true
}

```

* El tipo de dato será **Json**.
* El tipo de contenido será una aplicacción json con formato utf-8.

```js
function Guardar() {

    var Usuario = {
        Activo: $("#cboactivo").val() == 1 ? true : false,
        Apellidos: $("#txtapellidos").val(),
        Correo: $("#txtcorreo").val(),
        IdUsuario: $("#txtid").val(),
        Nombres: $("#txtnombres").val(),
        Reestablecer: true
    }

    jQuery.ajax({
        url: '@Url.Action("GuardarUsuario", "Home")',
        type: "POST",
        data: JSON.stringify({ objeto: Usuario }),
        dataType: "json",
        contentType: "application/json; charset=utf-8",
        success: function (data) {
            
        },
        error: function (error) {
            console.log(error)
        },
        beforeSend: function () {

        }
    });
}

```

## Función Succes

Ahora tenemos que crear la función que nos muestre los cambios que han sido realizada. Pero hay unas consideración que tenemos que tomar, si bien la consultado fue catalogada como "exitosa" no tiene porque serlo segun nuestras reglas de negocio. Primero veamos los resultados de unas consultas que caerían dentro **success**.

### Registrar Exito

```json
{
    "resultado":1,
    "mensaje":""
}

```

### Registrar Error 

```json
{
    "resultado":0,
    "mensaje":"Mensaje de Error"
}

```

### Editar Exito

```json
{
    "resultado":true,
    "mensaje":""
}

```

### Editar Exito

```json
{
    "resultado":false,
    "mensaje":"mensaje de error"
}

```

Con estos ejemplos podemos crear nuestra función, ahora todas estas respuestas las catagaloremos con el nombre ```data```, tambien hay que tener en cuenta que nosotros todavía tenemos en memoría el Usuario que hemos mandado, es decir si el usuario fue enviado para su creación, el usuario tendrá un id igual a cero de lo contrario será para su edición. Ahora dentro de ambos casos tendremos un exito y un fracaso, en el caso de creación es exitoso si el id recibido (data.resultado) es diferente de 0 y para editar su resultado será verdadero.

```js
function (data) {
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
}

```

![image](https://user-images.githubusercontent.com/59342976/154811494-7f1b1798-f3ae-463b-9678-bde50627d392.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte11)
