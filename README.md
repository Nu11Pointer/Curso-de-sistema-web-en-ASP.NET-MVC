# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 24

# Reportes en DashBoard

Lo primero lo primero será crear un procedimiento almacenado que nos devuelva los valores que queremos mostrar en el dashboard.

```sql
USE DBCARRITO
GO

CREATE PROC sp_ReporteDashboard
AS
BEGIN
	SELECT (
			SELECT COUNT(*)
			FROM CLIENTE
			) [TotalCliente],
		(
			SELECT ISNULL(SUM(Cantidad), 0)
			FROM DETALLE_VENTA
			) [TotalVenta],
		(
			SELECT COUNT(*)
			FROM PRODUCTO
			) [TotalProducto]
END

```

Para esta funcionalidad añadiremos una nueva entidad ```DashBoard```.

```c#
namespace CapaEntidad
{
    public class DashBoard
    {
        public int TotalCliente { get; set; }

        public int TotalVenta { get; set; }

        public int TotalProducto { get; set; }
    }
}

```

Luego tendremos que agregar en la capa de datos, le llamaremos ```CD_Reporte```.

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
    public class CD_Reporte
    {
        public DashBoard VerDashBoard()
        {
            var objeto = new DashBoard();

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    string query = "sp_ReporteDashboard";

                    var cmd = new SqlCommand()
                    {
                        CommandText = query,
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };

                    oconexion.Open();

                    using (SqlDataReader dr = cmd.ExecuteReader())
                    {
                        while (dr.Read())
                        {
                            objeto = new DashBoard()
                            {
                                TotalCliente = Convert.ToInt32(dr["TotalCliente"]),
                                TotalProducto = Convert.ToInt32(dr["TotalProducto"]),
                                TotalVenta = Convert.ToInt32(dr["TotalVenta"])
                            };
                        }
                    }
                }
            }
            catch (Exception)
            {
            }
            return objeto;
        }
    }
}

```

Y de igual forma en la capa de negocio 


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
    public class CN_Reporte
    {
        private CD_Reporte objCapaDato = new CD_Reporte();

        public DashBoard VerDashBoard()
        {
            return objCapaDato.VerDashBoard();
        }
    }
}

```

Añadiremos una nueva accion al controlador Home.

```c#
[HttpGet]
public JsonResult VistaDashBoard()
{
    DashBoard objeto = new CN_Reporte().VerDashBoard();

    return Json(new { resultado = objeto }, JsonRequestBehavior.AllowGet);
}

```

Para finalizar modificaremos el index.

```cshtml
@{
    ViewBag.Title = "Dashboard";
}

<h1 class="mt-4">Dashboard</h1>
<ol class="breadcrumb mb-4">
    <li class="breadcrumb-item active">Dashboard</li>
</ol>

<div class="row">
    <div class="col-xl-4 col-md-6">
        <div class="card bg-success text-white mb-4">
            <div class="card-body">
                <div class="row">
                    <div class="col-9">
                        <h6>Cantidad Clientes</h6>
                        <h6 id="totalcliente"></h6>
                    </div>
                    <div class="col-3">
                        <i class="fas fa-user fa-3x"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="col-xl-4 col-md-6">
        <div class="card bg-warning text-white mb-4">
            <div class="card-body">
                <div class="row">
                    <div class="col-9">
                        <h6>Cantidad Ventas</h6>
                        <h6 id="totalventa"></h6>
                    </div>
                    <div class="col-3">
                        <i class="fas fa-shopping-bag fa-3x"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="col-xl-4 col-md-6">
        <div class="card bg-secondary text-white mb-4">
            <div class="card-body">
                <div class="row">
                    <div class="col-9">
                        <h6>Cantidad Productos</h6>
                        <h6 id="totalproducto"></h6>
                    </div>
                    <div class="col-3">
                        <i class="fas fa-boxes fa-3x"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<div class="card mb-4">
    <div class="card-header">
        <i class="fas fa-tags me-1"></i>
        Historial De Ventas
    </div>
    <div class="card-body">
        <form>
            <div class="row align-items-end">
                <div class="col-2">
                    <div class="mb-2">
                        <label class="form-label">Fecha de Inicio:</label>
                        <input class="form-control" type="text" id="txtfechainicio" name="fechainicio" />
                    </div>
                </div>

                <div class="col-2">
                    <div class="mb-2">
                        <label class="form-label">Fecha de Fin:</label>
                        <input class="form-control" type="text" id="txtfechafin" name="fechafin" />
                    </div>
                </div>

                <div class="col-2">
                    <div class="mb-2">
                        <label class="form-label">Id de Transaccion:</label>
                        <input class="form-control" type="text" id="txtidtransaccion" name="idtransaccion" />
                    </div>
                </div>

                <div class="col-2">
                    <div class="d-grid mb-2">
                        <button class="btn btn-primary" id="btnbuscar" type="button"><i class="fas fa-search"></i> Buscar </button>
                    </div>
                </div>

                <div class="col-2">
                    <div class="d-grid mb-2">
                        <button class="btn btn-success" type="submit"><i class="fas fa-file-excel"></i> Exportar </button>
                    </div>
                </div>
            </div>
        </form>

        <hr />

        <div class="row">

            <div class="col-sm-12">
                <table id="tabla" class="display cell-border" style="width:100%">

                    <thead>
                        <tr>
                            <th>Fecha Venta</th>
                            <th>Cliente</th>
                            <th>Producto</th>
                            <th>Precio</th>
                            <th>Cantidad</th>
                            <th>Total</th>
                            <th>ID Transaccion</th>
                        </tr>
                    </thead>

                    <tbody>
                    </tbody>

                </table>
            </div>

        </div>



    </div>
</div>

@section scripts {
    <script>
        jQuery.ajax({
            url: '@Url.Action("VistaDashBoard", "Home")',
            type: "GET",
            data: null,
            dataType: "json",
            contentType: "application/json charset=utf-8",
            success: function (data) {
                var objeto = data.resultado;

                $("#totalcliente").text(objeto.TotalCliente);
                $("#totalventa").text(objeto.TotalVenta);
                $("#totalproducto").text(objeto.TotalProducto);
            },
            error: function (error) {
                console.log(error.responseText);
            },
            beforeSend: function () {}
        });
    </script>
}

```


El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte24)
