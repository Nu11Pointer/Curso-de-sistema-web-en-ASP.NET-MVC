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
   resultado:0,
   mensaje:""
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

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte11)
