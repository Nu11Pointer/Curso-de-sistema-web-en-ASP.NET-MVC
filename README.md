# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 14

# Creando Variable de Entorno Para Contraseña del Cliente SMTP

Una de las fallas de seguridad que hemos estado cometiendo anteriormente es "**HardCodear**", en este caso lo hemos hecho como bien dice el titulo con la clave de nuestro cliente SMTP para ello utilizaremos el buscador e ingresaremos a **Editar las variables de entorno del sistema**, luego click en **Variables de entorno...** y luego click en nueva tal como se muestra en la imagen, le agregaremos un nombre para identificarla y tambien un valor que será nuestra clave.

![image](https://user-images.githubusercontent.com/59342976/155039935-7dd961f6-10ad-4a67-82c2-801e14b5121e.png)

Una vez dentro del codigo simplemente la llamaremos de esta manera:

```c#
var smtp = new SmtpClient()
{
	Credentials = new NetworkCredential(userName: Environment.GetEnvironmentVariable("APPLICATION USERNAME CMVC")), password: Environment.GetEnvironmentVariable("APPLICATION PASSWORD CMVC")),
	Host = "smtp.gmail.com",
	Port = 587,
	EnableSsl = true
};

```

# Creando Procediminetos Almacenados

Crearemos los procedimientos almacenados correspondientes para crear nuestro CRUD de categorias, ya hemos hecho esto así que será mucho mas sencillo.

```sql
USE DBCARRITO
GO

CREATE PROCEDURE sp_RegistrarCategoria (
	@Descripcion VARCHAR(100),
	@Activo BIT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado INT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM CATEGORIA
			WHERE Descripcion = @Descripcion
			)
	BEGIN
		INSERT INTO CATEGORIA (
			Descripcion,
			Activo
			)
		VALUES (
			@Descripcion,
			@Activo
			)

		SET @Resultado = SCOPE_IDENTITY()
	END
	ELSE
		SET @Mensaje = 'La categoria ya existe.'
END
GO

CREATE PROCEDURE sp_EditarCategoria (
	@IdCategoria INT,
	@Descripcion VARCHAR(100),
	@Activo BIT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado BIT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM CATEGORIA
			WHERE Descripcion = @Descripcion
				AND IdCategoria != @IdCategoria
			)
	BEGIN
		UPDATE TOP (1) CATEGORIA
		SET Descripcion = @Descripcion,
			Activo = @Activo
		WHERE IdCategoria = @IdCategoria

		SET @Resultado = 1
	END
	ELSE
		SET @Mensaje = 'La categoria ya existe.'
END
GO

CREATE PROCEDURE sp_EliminarCategoria (
	@IdCategoria INT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado BIT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM PRODUCTO AS P
			INNER JOIN CATEGORIA AS C ON C.IdCategoria = P.IdCategoria
			WHERE P.IdCategoria = @IdCategoria
			)
	BEGIN
		DELETE TOP (1)
		FROM CATEGORIA
		WHERE IdCategoria = @IdCategoria

		SET @Resultado = 1
	END
	ELSE
		SET @Mensaje = 'La categoria se encuentra relacionada a un producto.'
END


```

## Capada Datos Categoria

```c#
using CapaEntidad;
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaDatos
{
    public class CD_Categoria
    {
        public List<Categoria> Listar()
        {
            var lista = new List<Categoria>();

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    string query = "select IdCategoria, Descripcion, Activo from CATEGORIA";

                    var cmd = new SqlCommand()
                    {
                        CommandText = query,
                        Connection = oconexion,
                        CommandType = CommandType.Text
                    };

                    oconexion.Open();

                    using (SqlDataReader dr = cmd.ExecuteReader())
                    {
                        while (dr.Read())
                        {
                            lista.Add(new Categoria()
                            {
                                IdCategoria = Convert.ToInt32(dr["IdCategoria"]),
                                Descripcion = dr["Descripcion"].ToString(),
                                Activo = Convert.ToBoolean(dr["Activo"])
                            });
                        }
                    }
                }
            }
            catch (Exception)
            {
            }
            return lista;
        }

        public int Registrar(Categoria obj, out string Mensaje)
        {
            int idautogenerado = 0;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var cmd = new SqlCommand()
                    {
                        CommandText = "sp_RegistrarCategoria",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("Descripcion", obj.Descripcion);
                    cmd.Parameters.AddWithValue("Activo", obj.Activo);
                    cmd.Parameters.Add("Resultado", SqlDbType.Int).Direction = ParameterDirection.Output;
                    cmd.Parameters.Add("Mensaje", SqlDbType.VarChar, 500).Direction = ParameterDirection.Output;

                    oconexion.Open();

                    cmd.ExecuteNonQuery();

                    idautogenerado = Convert.ToInt32(cmd.Parameters["Resultado"].Value);
                    Mensaje = cmd.Parameters["Mensaje"].Value.ToString();
                }
            }
            catch (Exception e)
            {
                idautogenerado = 0;
                Mensaje = e.Message;
            }
            return idautogenerado;
        }

        public bool Editar(Categoria obj, out string Mensaje)
        {
            bool resultado = false;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var cmd = new SqlCommand()
                    {
                        CommandText = "sp_EditarCategoria",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("IdCategoria", obj.IdCategoria);
                    cmd.Parameters.AddWithValue("Descripcion", obj.Descripcion);
                    cmd.Parameters.AddWithValue("Activo", obj.Activo);
                    cmd.Parameters.Add("Mensaje", SqlDbType.VarChar, 500).Direction = ParameterDirection.Output;
                    cmd.Parameters.Add("Resultado", SqlDbType.Int).Direction = ParameterDirection.Output;

                    oconexion.Open();

                    cmd.ExecuteNonQuery();

                    resultado = Convert.ToBoolean(cmd.Parameters["Resultado"].Value);
                    Mensaje = cmd.Parameters["Mensaje"].Value.ToString();
                }
            }
            catch (Exception e)
            {
                resultado = false;
                Mensaje = e.Message;
            }

            return resultado;
        }

        public bool Eliminar(int id, out string Mensaje)
        {
            bool resultado = false;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var cmd = new SqlCommand()
                    {
                        CommandText = "sp_EliminarCategoria",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("IdCategoria", id);
                    cmd.Parameters.Add("Mensaje", SqlDbType.VarChar, 500).Direction = ParameterDirection.Output;
                    cmd.Parameters.Add("Resultado", SqlDbType.Int).Direction = ParameterDirection.Output;

                    oconexion.Open();

                    cmd.ExecuteNonQuery();

                    resultado = Convert.ToBoolean(cmd.Parameters["Resultado"].Value);
                    Mensaje = cmd.Parameters["Mensaje"].Value.ToString();
                }
            }
            catch (Exception e)
            {
                resultado = false;
                Mensaje = e.Message;
            }

            return resultado;
        }
    }
}

```

## Capa Negocio Categoria 

```c#
using CapaDatos;
using CapaEntidad;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaNegocio
{
    public class CN_Categoria
    {
        private CD_Categoria objCapaDato = new CD_Categoria();

        public List<Categoria> Listar()
        {
            return objCapaDato.Listar();
        }

        public int Registrar(Categoria obj, out string Mensaje)
        {
            Mensaje = string.Empty;

            if (string.IsNullOrEmpty(obj.Descripcion) || string.IsNullOrWhiteSpace(obj.Descripcion))
            {
                Mensaje = "La descripción de la categoria no puede ser vacio.";
            }

            if (string.IsNullOrEmpty(Mensaje))
            {
                return objCapaDato.Registrar(obj, out Mensaje);
            }

            return 0;
        }

        public bool Editar(Categoria obj, out string Mensaje)
        {
            Mensaje = string.Empty;

            if (string.IsNullOrEmpty(obj.Descripcion) || string.IsNullOrWhiteSpace(obj.Descripcion))
            {
                Mensaje = "La descripción de la categoria no puede ser vacio.";
            }

            if (string.IsNullOrEmpty(Mensaje))
            {
                return objCapaDato.Editar(obj, out Mensaje);
            }

            return false;
        }

        public bool Eliminar(int id, out string Mensaje)
        {
            return objCapaDato.Eliminar(id, out Mensaje);
        }
    }
}

```

## Controlador Mantenedor

Vamos a añadirle al controlador Mantenedor las siguientes Acciones.

```c#
[HttpGet]
public JsonResult ListarCategorias()
{
	List<Categoria> oLista = new CN_Categoria().Listar();
	return Json(new { data = oLista }, JsonRequestBehavior.AllowGet);
}

[HttpPost]
public JsonResult GuardarCategorias(Categoria objeto)
{
	object resultado;
	string mensaje = string.Empty;

	if (objeto.IdCategoria == 0)
	{
		resultado = new CN_Categoria().Registrar(objeto, out mensaje);
	}
	else
	{
		resultado = new CN_Categoria().Editar(objeto, out mensaje);
	}

	return Json(new { resultado = resultado, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
}

[HttpPost]
public JsonResult EliminarCategorias(int id)
{
	bool respuesta = false;
	string mensaje = string.Empty;

	respuesta = new CN_Categoria().Eliminar(id, out mensaje);

	return Json(new { resultado = respuesta, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
}
	
```

## Vista Categoria

Para la vista simplemente vamos a copiar y pegar lo que hemos hecho en Usuario, modificandola un poco, al final obtenemos lo siguiente:

```cshtml

@{
    ViewBag.Title = "Categoria";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h1 class="mt-4">Categorias</h1>

<ol class="breadcrumb mb-4">
    <li class="breadcrumb-item"><a href="@Url.Action("Categoria", "Mantenedor")">Mantenimiento</a></li>
    <li class="breadcrumb-item active">Categoria</li>
</ol>

@*Form Tabla*@
<div class="card">

    @*Titulo*@
    <div class="card-header">
        <i class="fas fa-users me-1"></i> Lista de Categorias
    </div>

    @*Cuerpo*@
    <div class="card-body">

        @*Botones*@
        <div class="row">
            <div class="col-12">
                <button type="button" class="btn btn-success" onclick="abrirModal()">Crear Nuevo</button>
            </div>
        </div>

        <hr />

        @*Tabla*@
        <table id="tabla" class="display cell-border" style="width: 100%">

            <thead>
                <tr>
                    <th>Descripcion</th>
                    <th>Activo</th>
                    <th></th>
                </tr>
            </thead>

            <tbody>
            </tbody>

        </table>

    </div>

</div>

<!-- Modal -->
<div class="modal fade" id="FormModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true"
     data-bs-backdrop="static">

    <div class="modal-dialog">

        @*Contenido*@
        <div class="modal-content">

            @*Cabecera*@
            <div class="modal-header bg-dark text-white">

                <h5 class="modal-title" id="exampleModalLabel">Categoria</h5>
                <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal"
                        aria-label="Close"></button>

            </div>

            @*Cuerpo*@
            <div class="modal-body">

                <input id="txtid" type="hidden" value="0" />

                @*Inputs*@
                <div class="row g-1">

                    <div class="col-sm-6">
                        <label for="txtdescripcion" class="form-label">Descripcion</label>
                        <input type="text" class="form-control" id="txtdescripcion" autocomplete="off">
                    </div>

                    <div class="col-sm-6">
                        <label for="cboactivo" class="form-label">Activo</label>
                        <select id="cboactivo" class="form-select">
                            <option value="1">Si</option>
                            <option value="2">No</option>
                        </select>
                    </div>

                </div>

                @*Mensaje Error*@
                <div class="row mt-2">
                    <div class="col-12">
                        <div id="mensajeError" class="alert alert-danger" role="alert">
                            A simple danger alert—check it out!
                        </div>
                    </div>
                </div>

            </div>

            @*Pie de Modal*@
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                <button type="button" class="btn btn-primary" onclick="Guardar()">Guardar</button>
            </div>

        </div>

    </div>

</div>

@section scripts{
    <script>

    var tabladata;
    var filaSeleccionada;

    tabladata = $("#tabla").DataTable({
        responsive: true,
        ordering: false,
        "ajax": {
            url: '@Url.Action("ListarUsuarios", "Home")',
            type: "GET",
            dataType: "json"
        },
        "columns": [
            { "data": "Nombres" },
            { "data": "Apellidos" },
            { "data": "Correo" },
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

    function abrirModal(json) {

        $("#txtid").val(0);
        $("#txtnombres").val("");
        $("#txtapellidos").val("");
        $("#txtcorreo").val("");
        $("#cboactivo").val(1);
        $("#mensajeError").hide();

        if (json != null) {
            $("#txtid").val(json.IdUsuario);
            $("#txtnombres").val(json.Nombres);
            $("#txtapellidos").val(json.Apellidos);
            $("#txtcorreo").val(json.Correo);
            $("#cboactivo").val(json.Activo == true ? 1 : 2);
        }

        $("#FormModal").modal("show");
    }

    $("#tabla tbody").on("click", '.btn-editar', function () {

        filaSeleccionada = $(this).closest("tr");
        var data = tabladata.row(filaSeleccionada).data();
        abrirModal(data)
    })

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
    })

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

                $(".modal-body").LoadingOverlay("hide");

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
    </script>
}

```

![image](https://user-images.githubusercontent.com/59342976/155039207-daff233c-f1b1-432d-b446-d5aec68e4378.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte15)
