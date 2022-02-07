# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 08

## Cambiar El Idioma del DataTable

Para cambiar el idioma del DataTable podemos ver la siguiente documentación: [Spanish translation for DataTable](https://datatables.net/plug-ins/i18n/Spanish). Utilizaremos la propiedad ```languaje``` de la siguiente forma:

```json
"languaje": {
	"url": "https://cdn.datatables.net/plug-ins/1.11.4/i18n/es_es.json"
}
```

## Formulario Modal

![image](https://user-images.githubusercontent.com/59342976/152732519-a566e0fc-6826-4299-9e69-3d8a6680f9a4.png)

Un [Modal](https://getbootstrap.com/docs/5.0/components/modal/#live-demo) es basicamente un formulario que aparece cuando ocurre un evento y que ocupa gran parte de la pantalla. Colocaremos el siguiente codigo:

```html
<!-- Modal -->
<div class="modal fade" id="FormModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="exampleModalLabel">Modal title</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
                ...
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                <button type="button" class="btn btn-primary">Save changes</button>
            </div>
        </div>
    </div>
</div>
```

Le añadiremos al boton de **Crear Nuevo** el evento onClick:

```html
<button type="button" class="btn btn-success" onclick="abrirModal()">Crear Nuevo</button>
```

Y la función dentro del script:

```js
function abrirModal() {
	$("#FormModal").modal("show");
}
```

Dentro del modal lo personalizaremos y añadiremos un [Form](https://getbootstrap.com/docs/5.0/forms/overview/). Aparte de que tambien editaremos algunas clases para poder llamarlas luego haciendo uso del script.

```html
<!-- Modal -->
<div class="modal fade" id="FormModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-dark text-white">
                <h5 class="modal-title" id="exampleModalLabel">Usuario</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body g-2">
                <div class="row">

                    <div class="col-sm-6">
                        <label for="txtnombres" class="form-label">Nombres</label>
                        <input type="text" class="form-control" id="txtnombres">
                    </div>

                    <div class="col-sm-6">
                        <label for="txtapellidos" class="form-label">Apellidos</label>
                        <input type="text" class="form-control" id="txtapellidos">
                    </div>

                    <div class="col-sm-6">
                        <label for="txtcorreo" class="form-label">Correo</label>
                        <input type="text" class="form-control" id="txtcorreo">
                    </div>

                    <div class="col-sm-6">
                        <label for="cboactivo" class="form-label">Activo</label>
                        <select id="cboactivo" class="form-select">
                            <option value="1">Si</option>
                            <option value="2">No</option>
                        </select>
                    </div>

                </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                <button type="button" class="btn btn-primary">Guardar</button>
            </div>
        </div>
    </div>
</div>
```

Y para realizar el evento **onclick** dentro de la fila de la tabla utilizaremos el siguiente codigo:

```js
$("#tabla tbody").on("click", '.btn-editar', function () {
	var filaSeleccionada = $(this).closest("tr");
	var data = tabladata.row(filaSeleccionada).data()
})
```

![image](https://user-images.githubusercontent.com/59342976/152860066-075cc1f2-3e96-461f-8173-92bb8bc853e5.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte08)
