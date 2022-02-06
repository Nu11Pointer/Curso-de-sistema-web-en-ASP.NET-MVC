# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 06

## Agregando un nuevo registro a la tabla USUARIO

Agregaremos el siguiente registro:

```sql
INSERT INTO USUARIO (
	Nombres,
	Apellidos,
	Correo,
	Clave
	)
VALUES (
	'test 02',
	'user 02',
	'user02@example.com',
	'ecd71870d1963316a97e3ac3408c9835ad8cf0f3c1bc703527c30265534f75ae'
	)
```

![image](https://user-images.githubusercontent.com/59342976/152661573-447df0bf-5a5f-4976-b9c2-f890dabc6e32.png)

Es un poco dificil de ver en este estado, pero si lo reordenamos un poco este es nuestro resultado:

```Json
[
   {
      "IdUsuario":1,
      "Nombres":"test nombre",
      "Apellidos":"test apellido",
      "Correo":"test@example.com",
      "Clave":"ecd71870d1963316a97e3ac3408c9835ad8cf0f3c1bc703527c30265534f75ae",
      "Reestablecer":true,
      "Activo":true
   },
   {
      "IdUsuario":2,
      "Nombres":"test 02",
      "Apellidos":"user 02",
      "Correo":"user02@example.com",
      "Clave":"ecd71870d1963316a97e3ac3408c9835ad8cf0f3c1bc703527c30265534f75ae",
      "Reestablecer":true,
      "Activo":true
   }
]
```

## Diseñar Formulario Usuario

![image](https://user-images.githubusercontent.com/59342976/152664240-629a8003-7a11-4e9e-85e5-321f91b86886.png)

El formulario de usuarios de momento solo tiene un titulo que nos indica que en efecto estamos en formulario Usuarios. Siguiendo el ejemplo de la plantilla **sb-admin** colocaremos el siguiente fragmento de codigo:

```html

@{
    ViewBag.Title = "Usuarios";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h1 class="mt-4">Usuarios</h1>

<ol class="breadcrumb mb-4">
	<li class="breadcrumb-item"><a href="index.html">Resumen</a></li>
	<li class="breadcrumb-item active">Usuarios</li>
</ol>
```

![image](https://user-images.githubusercontent.com/59342976/152664272-05fe74ee-d0c1-4caf-ab58-dd5104c0aa73.png)

Luego para darle un poco de diseño utilizaremos el siguiente ejemplo de [Cards Bootstrap v5.0](https://getbootstrap.com/docs/5.0/components/card/#header-and-footer).

```html
<div class="card">
	<h5 class="card-header">Featured</h5>
	<div class="card-body">
		<h5 class="card-title">Special title treatment</h5>
		<p class="card-text">With supporting text below as a natural lead-in to additional content.</p>
		<a href="#" class="btn btn-primary">Go somewhere</a>
	</div>
</div>
```

Lo modificaremos un poco y le agregaremos un icono usando **FontAwesome** y eliminaremos el cuerpo:

```html
<div class="card">
	<div class="card-header">
		<i class="fas fa-users me-1"></i> Lista de Usuarios
	</div>
	<div class="card-body"></div>
</div>
```

Y dentro de **card-body** colocaremos un boton y una tabla con las columnas que necesitamos pero vacía.

```html
<div class="row">
	<div class="col-12">
		<button type="button" class="btn btn-success">Crear Nuevo</button>
	</div>
</div>
<hr />
<table id="tabla" class="display cell-border" style="width: 100%">
	<thead>
		<tr>
			<th>Nombres</th>
			<th>Apellidos</th>
			<th>Correo</th>
			<th>Activo</th>
		</tr></thead>
	<tbody>
	</tbody>
</table>
```

### ¿Qué es DataTables?

![image](https://user-images.githubusercontent.com/59342976/152664361-f34f6ff3-6e0c-4c26-81ff-eb4ae2d9a440.png)


Como bien lo describe su web oficial [DataTable](https://datatables.net/) es un plugin de jQuery para crear tablas avanzadas e incliuye funciones como:

* Paginación.
* Busqueda instantanea.
* Ordenamiento multicolumna.
* Entre otras muchas cosas.

### ¿Qué es Ajax?

Ajax (Asynchronous JavaScript and XML) se refiere a un grupo de tecnologías que se utilizan para desarrollar aplicaciones web. En palabras sencillas Ajax nos permite generar solicitudes al servidor sin necesidad de recargar la pagina.

#### Usando Ajax

Haciendo uso de **@RenderSection("scripts", required: false)** que se encuentra en **_Layaout.cshtml** el cual nos permite generar scripts de la siguiente manera:

```html
@section scripts{
    <script>
	    ...
    </script>
}
```

Lo utilizaremos para mostrar en la consola del navegador nuestra lista de usuarios, de la siguiente manera:

```html
@section scripts{
    <script>
        jQuery.ajax({
            url: '@Url.Action("ListarUsuarios", "Home")',
            type: "GET",
            dataType: "json",
            contentType: "application/json; charset=utf-8",
            success: function (data) {
                console.log(data)
            }
        })
    </script>
}
```

Y este es el resultado:

![image](https://user-images.githubusercontent.com/59342976/152665082-1f8b723b-7eee-400b-ae73-4e563f9578fc.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte06)
