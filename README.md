# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 16-17

Ambos Capitulos se basan en realizar exactamente lo mismo que hemos hecho para la entidad Usuarios.

# CD_Marca.cs

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
    public class CD_Marca
    {
        public List<Marca> Listar()
        {
            var lista = new List<Marca>();

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    string query = "select IdMarca, Descripcion, Activo from MARCA";

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
                            lista.Add(new Marca()
                            {
                                IdMarca = Convert.ToInt32(dr["IdMarca"]),
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

        public int Registrar(Marca obj, out string Mensaje)
        {
            int idautogenerado = 0;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var cmd = new SqlCommand()
                    {
                        CommandText = "sp_RegistrarMarca",
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

        public bool Editar(Marca obj, out string Mensaje)
        {
            bool resultado = false;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var cmd = new SqlCommand()
                    {
                        CommandText = "sp_EditarMarca ",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("IdMarca", obj.IdMarca);
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
                        CommandText = "sp_EliminarMarca",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("IdMarca", id);
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

# CN_Marca.cs

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
    public class CN_Marca
    {
        private CD_Marca objCapaDato = new CD_Marca();

        public List<Marca> Listar()
        {
            return objCapaDato.Listar();
        }

        public int Registrar(Marca obj, out string Mensaje)
        {
            Mensaje = string.Empty;

            if (string.IsNullOrEmpty(obj.Descripcion) || string.IsNullOrWhiteSpace(obj.Descripcion))
            {
                Mensaje = "La descripción de la marca no puede ser vacio.";
            }

            if (string.IsNullOrEmpty(Mensaje))
            {
                return objCapaDato.Registrar(obj, out Mensaje);
            }

            return 0;
        }

        public bool Editar(Marca obj, out string Mensaje)
        {
            Mensaje = string.Empty;

            if (string.IsNullOrEmpty(obj.Descripcion) || string.IsNullOrWhiteSpace(obj.Descripcion))
            {
                Mensaje = "La descripción de la marca no puede ser vacio.";
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

## ManetenedorController

```c#
using CapaEntidad;
using CapaNegocio;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace CapaPresentacionAdmin.Controllers
{
    public class MantenedorController : Controller
    {
        // GET: Mantenedor
        public ActionResult Categoria()
        {
            return View();
        }

        public ActionResult Marca()
        {
            return View();
        }

        public ActionResult Producto()
        {
            return View();
        }

        #region Categoria
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
        public JsonResult EliminarCategoria(int id)
        {
            bool respuesta = false;
            string mensaje = string.Empty;

            respuesta = new CN_Categoria().Eliminar(id, out mensaje);

            return Json(new { resultado = respuesta, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
        }
        #endregion

        #region Marca
        [HttpGet]
        public JsonResult ListarMarca()
        {
            List <Marca> oLista = new CN_Marca().Listar();
            return Json(new { data = oLista }, JsonRequestBehavior.AllowGet);
        }

        [HttpPost]
        public JsonResult GuardarMarca(Marca objeto)
        {
            object resultado;
            string mensaje = string.Empty;

            if (objeto.IdMarca == 0)
            {
                resultado = new CN_Marca().Registrar(objeto, out mensaje);
            }
            else
            {
                resultado = new CN_Marca().Editar(objeto, out mensaje);
            }

            return Json(new { resultado = resultado, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
        }

        [HttpPost]
        public JsonResult EliminarMarca(int id)
        {
            bool respuesta = false;
            string mensaje = string.Empty;

            respuesta = new CN_Marca().Eliminar(id, out mensaje);

            return Json(new { resultado = respuesta, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
        }
        #endregion

    }
}

``` 

# Categoria.cshtml

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
                            <option value="0">No</option>
                            <option value="1">Si</option>
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
            url: '@Url.Action("ListarCategorias", "Mantenedor")',
            type: "GET",
            dataType: "json"
        },
        "columns": [
            { "data": "Descripcion" },
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
        $("#txtdescripcion").val("");
        $("#cboactivo").val(1);

        $("#mensajeError").hide();

        if (json != null) {
            $("#txtid").val(json.IdCategoria);
            $("#txtdescripcion").val(json.Descripcion);
            $("#cboactivo").val(json.Activo ? 1 : 0);
        }

        $("#FormModal").modal("show");
    }

    $("#tabla tbody").on("click", '.btn-editar', function () {

        filaSeleccionada = $(this).closest("tr");
        var data = tabladata.row(filaSeleccionada).data();
        abrirModal(data)
    })

    $("#tabla tbody").on("click", '.btn-eliminar', function () {

        var categoriaseleccionada = $(this).closest("tr");
        var data = tabladata.row(categoriaseleccionada).data();

        swal({
            title: "¿Está seguro?",
            text: "¿Desea eliminar la categoria?",
            type: "warning",
            showCancelButton: true,
            confirmButtonClass: "btn-primary",
            confirmButtonText: "Si",
            cancelButtonText: "No",
            closeOnConfirm: true
        },
            function () {

                jQuery.ajax({
                    url: '@Url.Action("EliminarCategoria", "Mantenedor")',
                    type: "POST",
                    data: JSON.stringify({ id: data.IdCategoria }),
                    dataType: "json",
                    contentType: "application/json; charset=utf-8",
                    success: function (data) {

                        if (data.resultado) {

                            tabladata.row(categoriaseleccionada).remove().draw();
                        }
                        else {

                            swal("No se puedo eliminar la categoria.", data.mensaje, "error");
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

        var Categoria = {
            IdCategoria: $("#txtid").val(),
            Descripcion: $("#txtdescripcion").val(),
            Activo: $("#cboactivo").val() == 1 ? true : false
        }

        jQuery.ajax({
            url: '@Url.Action("GuardarCategorias", "Mantenedor")',
            type: "POST",
            data: JSON.stringify({ objeto: Categoria }),
            dataType: "json",
            contentType: "application/json; charset=utf-8",
            success: function (data) {

                $(".modal-body").LoadingOverlay("hide");

                // Categoria Nueva
                if (Categoria.IdCategoria == 0) {

                    if (data.resultado != 0) {
                        Categoria.IdCategoria = data.resultado;
                        tabladata.row.add(Categoria).draw(false);
                        $("#FormModal").modal("hide");
                    }
                    else {
                        $("#mensajeError").text(data.mensaje);
                        $("#mensajeError").show();
                    }
                }
                // Categoria Editar
                else {
                    if (data.resultado) {
                        tabladata.row(filaSeleccionada).data(Categoria).draw(false);
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

# Marca.cshtml

```cshtml

@{
    ViewBag.Title = "Marca";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h1 class="mt-4">Categorias</h1>

<ol class="breadcrumb mb-4">
    <li class="breadcrumb-item"><a href="@Url.Action("Marca", "Mantenedor")">Mantenimiento</a></li>
    <li class="breadcrumb-item active">Marca</li>
</ol>

@*Form Tabla*@
<div class="card">

    @*Titulo*@
    <div class="card-header">
        <i class="fas fa-users me-1"></i> Lista de Marcas
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

                <h5 class="modal-title" id="exampleModalLabel">Marca</h5>

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
                            <option value="0">No</option>
                            <option value="1">Si</option>
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
            url: '@Url.Action("ListarMarca", "Mantenedor")',
            type: "GET",
            dataType: "json"
        },
        "columns": [
            { "data": "Descripcion" },
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
        $("#txtdescripcion").val("");
        $("#cboactivo").val(1);

        $("#mensajeError").hide();

        if (json != null) {
            $("#txtid").val(json.IdMarca);
            $("#txtdescripcion").val(json.Descripcion);
            $("#cboactivo").val(json.Activo ? 1 : 0);
        }

        $("#FormModal").modal("show");
    }

    $("#tabla tbody").on("click", '.btn-editar', function () {

        filaSeleccionada = $(this).closest("tr");
        var data = tabladata.row(filaSeleccionada).data();
        abrirModal(data)
    })

    $("#tabla tbody").on("click", '.btn-eliminar', function () {

        var marcaseleccionada = $(this).closest("tr");
        var data = tabladata.row(marcaseleccionada).data();

        swal({
            title: "¿Está seguro?",
            text: "¿Desea eliminar la marca?",
            type: "warning",
            showCancelButton: true,
            confirmButtonClass: "btn-primary",
            confirmButtonText: "Si",
            cancelButtonText: "No",
            closeOnConfirm: true
        },
            function () {

                jQuery.ajax({
                    url: '@Url.Action("EliminarMarca", "Mantenedor")',
                    type: "POST",
                    data: JSON.stringify({ id: data.IdMarca }),
                    dataType: "json",
                    contentType: "application/json; charset=utf-8",
                    success: function (data) {

                        if (data.resultado) {

                            tabladata.row(marcaseleccionada).remove().draw();
                        }
                        else {

                            swal("No se puedo eliminar la marca.", data.mensaje, "error");
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

        var Marca = {
            IdMarca: $("#txtid").val(),
            Descripcion: $("#txtdescripcion").val(),
            Activo: $("#cboactivo").val() == 1 ? true : false
        }

        jQuery.ajax({
            url: '@Url.Action("GuardarMarca", "Mantenedor")',
            type: "POST",
            data: JSON.stringify({ objeto: Marca }),
            dataType: "json",
            contentType: "application/json; charset=utf-8",
            success: function (data) {

                $(".modal-body").LoadingOverlay("hide");

                // Marca Nueva
                if (Marca.IdMarca == 0) {

                    if (data.resultado != 0) {
                        Marca.IdMarca = data.resultado;
                        tabladata.row.add(Marca).draw(false);
                        $("#FormModal").modal("hide");
                    }
                    else {
                        $("#mensajeError").text(data.mensaje);
                        $("#mensajeError").show();
                    }
                }
                // Marca Editar
                else {
                    if (data.resultado) {
                        tabladata.row(filaSeleccionada).data(Marca).draw(false);
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

![image](https://user-images.githubusercontent.com/59342976/155243489-cfb4772e-6b1f-4428-8b05-f77adc075bea.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte16)
