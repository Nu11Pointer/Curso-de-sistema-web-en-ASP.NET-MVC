# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 14

# Agregando Funcionalidad de Correo Electronico Al Registrar un Usuario

Dentro de nuestra capa de negocios realizaremos los llamados hacia nuestra funciones creadas en el capitulo anterior ```GenerarClave``` y ```EnviarCorreo```. Donde antes tendrímos una clave predeterminada "test123" asignaremos una clave aleatoria, generaremos un ausunto, el cuerpo del mensaje y enviaremos el correo electronico, si este se envia satisfactoriamente entonces registraremos nuestro usuario, en caso contrario retornaremos 0 y enviaremos un mensaje de error.

```c#
if (string.IsNullOrEmpty(Mensaje))
{
	string clave = CN_Recursos.GenerarClave();
	string asunto = "Creación de cuenta";
	string mensaje_correo = $"<h3>Su cuenta fue creada correctamente</h3><p>Su contraseña para acceder es: {clave}</p>";
	bool respuesta = CN_Recursos.EnviarCorreo(obj.Correo, asunto, mensaje_correo);

	if (respuesta)
	{
		obj.Clave = CN_Recursos.ConvertirSha256(clave);
		return objCapaDato.Registrar(obj, out Mensaje);
	}
	else
	{
		Mensaje = "No se puede enviar el correo.";
		return 0;
	}
}

```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte14)
