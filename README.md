# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 13

# Implementación Eliminar Usuario

## Accion Eliminar Usuario

Con el metodo/accion ```EliminarUsuario``` del controlador ```Home``` se nos posibilitará realizar dicha operación y seguirá la misma logica que hemos aplicado previamente, retornara un json donde el atributo resultado será verdadero si la acción fue realizada con exito de lo contrario será un false y adjuntaremos un mensaje.

```c#
[HttpPost]
public JsonResult EliminarUsuario(int id)
{
	bool respuesta = false;
	string mensaje = string.Empty;
	
	respuesta = new CN_Usuarios().Eliminar(id, out mensaje);
	
	return Json(new { resultado = respuesta, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
}
	
```

## Script Usuarios

Lo primero que tendremos que hacer será agragarle una clase de nombre ```btn-eliminar``` al boton de eliminar, nos quedará algo asi:

```html
<button type="button" class="btn btn-danger btn-sm ms-2 btn-eliminar"><i class="fas fa-trash"></i></button>

```

Lo siguiente será llamar al evento ```onclick``` de cada uno de nuestros botones que tengan la clase ```btn-eliminar``` de la siguiente forma:

```js

$("#tabla tbody").on("click", '.btn-eliminar', function () {});

```

![image](https://user-images.githubusercontent.com/59342976/154889232-3131633f-9cc1-45fe-89a6-7a1750327113.png)

Dentro de está función que será ejecutada cada que vez que demos al boton eliminar tendremos que mostrar un Sweet Alert como el que se ve en la imagen anterior.

```js
swal({
	title: "¿Está seguro?",
	text: "¿Desea eliminar el usuario?",
	type: "warning",
	showCancelButton: true,
	confirmButtonClass: "btn-primary",
	confirmButtonText: "Si",
	cancelButtonText: "No",
	closeOnConfirm: true
	}, function () {});
	
```

En caso de darle al boton de confirmar ("Si") ejecutaremos lo siguiente:

```js
function () {

    jQuery.ajax({
        url: '@Url.Action("EliminarUsuario", "Home")',
        type: "POST",
        data: JSON.stringify({ id: data.IdUsuario }),
        dataType: "json",
        contentType: "application/json; charset=utf-8",
        success: function (data) {

            if (data.resultado) {

                tabladata.row(usuarioseleccionado).remove().draw();
            }
            else {

                swal("No se puedo eliminar el usuario.", data.mensaje, "error");
            }
        },
        error: function (error) {
            console.log(error);
        },
        beforeSend: function () { }
    });

```

Al final obtendremos algo como esto:

```js

$("#tabla tbody").on("click", '.btn-eliminar', function () {

    var usuarioseleccionado = $(this).closest("tr");
    var data = tabladata.row(usuarioseleccionado).data();

    swal({
        title: "¿Está seguro?",
        text: "¿Desea eliminar el usuario?",
        type: "warning",
        showCancelButton: true,
        confirmButtonClass: "btn-primary",
        confirmButtonText: "Si",
        cancelButtonText: "No",
        closeOnConfirm: true
    },
        function () {

            jQuery.ajax({
                url: '@Url.Action("EliminarUsuario", "Home")',
                type: "POST",
                data: JSON.stringify({ id: data.IdUsuario }),
                dataType: "json",
                contentType: "application/json; charset=utf-8",
                success: function (data) {

                    if (data.resultado) {

                        tabladata.row(usuarioseleccionado).remove().draw();
                    }
                    else {

                        swal("No se puedo eliminar el usuario.", data.mensaje, "error");
                    }
                },
                error: function (error) {
                    console.log(error);
                },
                beforeSend: function () { }
            });

        });
});

```

# Implementación Enviar Correo Electronico

Para enviar correos electronicos necesitaremos de las siguientes referencias y una cuenta de gmail.

``` c#
using System;
using System.Net;
using System.Net.Mail;

```

![image](https://user-images.githubusercontent.com/59342976/154890120-a6dd21e4-f07a-4d5a-896c-93f5389b6a10.png)

Dentro de la configuraciones de Gmail nos dirigiremos a la sección de **seguridad**, activaremos la **verificación en 2 pasos**. Lo segundo que haremos será dirigirnos a **Contraseña de Aplicaciones** nos pedirá iniciar sesión nuevamente y generaremos una contraseña. La función es bastante sencilla y autoexplicativa basicamente es crear un objeto de tipo Mensaje y configurar nuestro cliente smtp el cual es protocolo para enviar mensaje atravez de internet.

```c#
public static bool EnviarCorreo(string correo, string asunto, string mensaje)
{
	bool resultado = false;
	try
	{
		var mail = new MailMessage(from: "stevenwerr@gmail.com", to: correo)
		{
			Subject = asunto,
			Body = mensaje,
			IsBodyHtml = true
                };
		var smtp = new SmtpClient()
                {
			Credentials = new NetworkCredential(userName: "stevenwerr@gmail.com", password: "woibjdshycukncma"),
			Host = "smtp.gmail.com",
			Port = 587,
			EnableSsl = true
                };
		
		smtp.Send(mail);
		resultado = true;
	}
	catch
	{
		resultado = false;
	}
	return resultado;
}

```

Tambien crearemos una función pare crear una contraseña aleatoria.

```c#
public static string GenerarClave()
{
	string clave = Guid.NewGuid().ToString("N").Substring(0, 6);
	return clave;
}

```

![image](https://user-images.githubusercontent.com/59342976/154889815-55e91424-75e5-4c2d-b947-55925f21df3e.png)


El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte13)
