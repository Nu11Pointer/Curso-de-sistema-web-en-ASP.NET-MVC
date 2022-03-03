# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 25

# Implementando Reporte de Ventas

Tendremos que crear un procedimiento almacenado que reciba la fecha de inicio, la fecha de fin y el id de transacción si es que se pasa por parametro y asi poder filtrar las ventas que se han realizado. Hay que tomar varias consideraciones como por ejemplo es necesario establecer el formato de la fecha dentro de "sp", verificar los valores que vamos a mostrar en este caso, queremos mostrar la fecha de la venta, esta esta en la tabla de ventas, el nombre del cliente esta en la tabla de clientes por ende hay que hacer inner joins a varias tablas por ejemplo el detalle de la venta para conocer el precio del producto que lleva, la cantidad entre otras cosas.

```sql
USE DBCARRITO
GO

CREATE PROCEDURE sp_ReporteVentas (
	@fechainicio VARCHAR(10),
	@fechafin VARCHAR(10),
	@idtransaccion VARCHAR(50)
	)
AS
BEGIN
	SET DATEFORMAT dmy;

	SELECT CONVERT(CHAR(10), V.FechaVenta, 103) AS [FechaVenta],
		CONCAT (
			C.Nombres,
			C.Apellidos
			) AS [Cliente],
		P.Nombre AS [Producto],
		P.Precio,
		DV.Cantidad,
		DV.Total,
		V.IdTransaccion
	FROM DETALLE_VENTA AS DV
	INNER JOIN PRODUCTO AS P ON P.IdProducto = DV.IdProducto
	INNER JOIN VENTA AS V ON V.IdVenta = DV.IdVenta
	INNER JOIN CLIENTE C ON C.IdCliente = V.IdCliente
	WHERE CONVERT(DATE, V.FechaVenta) BETWEEN @fechainicio
			AND @fechafin
		AND V.IdTransaccion = IIF(@idtransaccion = '', V.IdTransaccion, @idtransaccion)
END

```

Lo siguiente a realizar es una entidad que nos represente el reporte de ventas.

```c#
namespace CapaEntidad
{
    public class Reporte
    {
        public string FechaVenta { get; set; }

        public string Cliente { get; set; }

        public string Producto { get; set; }

        public decimal Precio { get; set; }

        public int Cantidad { get; set; }

        public decimal Total { get; set; }

        public string IdTransaccion { get; set; }
    }
}

```

Dentro de la clase de Datos que habiamos creado previamente, de nombre ```CD_Reporte``` crearemos la siguiente función para mandar a traer nuestro resultado de la base de datos.

```c#
public List<Reporte> Ventas(string fechainicio, string fechafin, string idtransaccion)
{
    var lista = new List<Reporte>();

    try
    {
	using (var oconexion = new SqlConnection(Conexion.cn))
	{
	    string query = "sp_ReporteVentas";

	    var cmd = new SqlCommand()
	    {
		CommandText = query,
		Connection = oconexion,
		CommandType = CommandType.StoredProcedure
	    };
	    cmd.Parameters.AddWithValue("fechainicio", DateTime.Parse(fechainicio));
	    cmd.Parameters.AddWithValue("fechafin", DateTime.Parse(fechafin));
	    cmd.Parameters.AddWithValue("idtransaccion", idtransaccion);

	    oconexion.Open();

	    using (SqlDataReader dr = cmd.ExecuteReader())
	    {
		while (dr.Read())
		{
		    lista.Add(new Reporte()
		    {
			FechaVenta = dr["FechaVenta"].ToString(),
			Cliente = dr["Cliente"].ToString(),
			Producto = dr["Apellidos"].ToString(),
			Precio = Convert.ToDecimal(dr["Precio"], new CultureInfo("es-NI")),
			Cantidad = Convert.ToInt32(dr["Cantidad"]),
			Total = Convert.ToDecimal(dr["Total"], new CultureInfo("es-NI")),
			IdTransaccion = dr["IdTransaccion"].ToString()
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

```

Como es de esperarse a este punto, tendremos que agregar el metodo correspondiente a su clase correspondiente en la capa de negocio.

```c#
public List<Reporte> Ventas(string fechainicio, string fechafin, string idtransaccion)
{
    return objCapaDato.Ventas(fechainicio, fechafin, idtransaccion);
}

```

Y ya casi finalizando al controlador correspondiente en este caso Home.

```c#
[HttpGet]
public JsonResult ListaReporte(string fechainicio, string fechafin, string idtransaccion)
{
    List<Reporte> olista = new CN_Reporte().Ventas(fechainicio, fechafin, idtransaccion);

    return Json(new { resultado = olista }, JsonRequestBehavior.AllowGet);
}

```

Ahora tendremos que añadir la libreria ```jQuery.UI``` y añadir sus respectivas referencias al ```BundleConfig``` para poder aplicar formularios DataPicker a nuestra web.

![image](https://user-images.githubusercontent.com/59342976/156467396-0f3cda77-89f7-47dc-b33c-0e8e1ea81272.png)

```c#
bundles.Add(new Bundle("~/bundles/complementos").Include(
                      "~/Scripts/fontawesome/all.min.js",
                      "~/Scripts/DataTables/jquery.dataTables.js",
                      "~/Scripts/DataTables/dataTables.responsive.js",
                      "~/Scripts/loadingoverlay/loadingoverlay.min.js",
                      "~/Scripts/sweetalert.min.js",
                      "~/Scripts/jquery.validate.js",
                      "~/Scripts/jquery-ui.js",
                      "~/Scripts/scripts.js"
                      ));
		      
bundles.Add(new StyleBundle("~/Content/css").Include(
                "~/Content/site.css",
                "~/Content/DataTables/css/jquery.dataTables.css",
                "~/Content/DataTables/css/responsive.dataTables.css",
                "~/Content/jquery-ui.css",
                "~/Content/sweetalert.css"
                ));

```

Ahora si, implementemos el datapicker.

```js
$("#txtfechainicio").datepicker({dateFormat : 'dd/mm/yy'}).datepicker('setDate', new Date());
$("#txtfechafin").datepicker({ dateFormat: 'dd/mm/yy' }).datepicker('setDate', new Date());

```

![image](https://user-images.githubusercontent.com/59342976/156467873-2e9aa545-4940-4f2f-89c8-67e258ead542.png)

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte25)
