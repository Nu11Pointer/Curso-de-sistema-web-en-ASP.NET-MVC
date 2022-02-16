# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 09

## Mostrar Datos al Momento de editar

Para saber cuando debemos crear un nuevo elemento o editar utilizaremos el siguiente input.

```html
<input id="txtid" type="hidden" value="0"/>
```

Utilizaremos la función declarada previamente ```abrirModal()``` solo que la modificaremos un poco quedando de la siguiente manera:

```js
function abrirModal(json) {
	$("#txtid").val(0);
	$("#txtnombres").val("");
	$("#txtapellidos").val("");
	$("#txtcorreo").val("");
	$("#cboactivo").val(1);
	if (json != null) {
		$("#txtid").val(json.IdUsuario);
		$("#txtnombres").val(json.Nombres)
		$("#txtapellidos").val(json.Apellidos)
		$("#txtcorreo").val(json.Correo)
		$("#cboactivo").val(json.Activo == true ? 1 : 0)
		}

	$("#FormModal").modal("show");
}
```

Tambien modificaremos el evento onclick del boton editar:

```javascript
$("#tabla tbody").on("click", '.btn-editar', function () {
	var filaSeleccionada = $(this).closest("tr");
	var data = tabladata.row(filaSeleccionada).data()
	abrirModal(data)
})
```

Corrección al momento de dar click fuera del modal, lo unico que añadiremos será especificar la propiedad ```data-bs-backdrop="static"``` al div correspondiente.

Vamos a establecer la manera en la vamos a guardar pero sin guardar en la base de datos (eso lo haremos en el siguiente capitulo). Para ello Crearemos la función Guardar dentro del script que tendrá la siguiente estructura.

```javascript
function Guardar() {
	var Usuario = {
	Activo: $("#cboactivo").val() == 1 ? true : false,
	Apellidos: $("#txtapellidos").val(),
	Correo: $("#txtcorreo").val(),
	IdUsuario: $("#txtid").val(),
	Nombres: $("#txtnombres").val(),
	Reestablecer: true
	}
	console.log(Usuario)
	if (Usuario.IdUsuario == 0) {
		tabladata.row.add(Usuario).draw(false);
	}
	else {
		tabladata.row(filaSeleccionada).data(Usuario).draw(false);
	}
	$("#FormModal").modal("hide");
}
```

Es necesario que la variable ```filaSeleccionada``` sea declarada como global.

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte09)
