# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 18

# Procedimientos Almacenados Productos

```sql
USE DBCARRITO
GO

CREATE PROC sp_EliminarProducto(
	@IdProducto INT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado BIT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM DETALLE_VENTA AS dv
			INNER JOIN PRODUCTO AS P ON P.IdProducto = dv.IdProducto
			WHERE P.IdProducto = @IdProducto
			)
	BEGIN
		DELETE TOP (1)
		FROM PRODUCTO
		WHERE IdProducto = @IdProducto

		SET @Resultado = 1
	END
	ELSE
		SET @Mensaje = 'El producto se encuentra relacionado a una venta'
END

GO

CREATE PROC sp_EditarProducto (
	@IdProducto INT,
	@Nombre VARCHAR(100),
	@Descripcion VARCHAR(100),
	@IdMarca VARCHAR(100),
	@IdCategoria VARCHAR(100),
	@Precio DECIMAL(10, 2),
	@Stock INT,
	@Activo BIT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado BIT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM PRODUCTO
			WHERE Nombre = @Nombre
				AND IdProducto != @IdProducto
			)
	BEGIN
		UPDATE PRODUCTO
		SET Nombre = @Nombre,
			Descripcion = @Descripcion,
			IdMarca = @IdMarca,
			IdCategoria = @IdCategoria,
			Precio = @Precio,
			Stock = @Stock,
			Activo = @Activo
		WHERE IdProducto = @IdProducto

		SET @Resultado = 1
	END
	ELSE
		SET @Mensaje = 'El producto ya existe'
END

GO

CREATE PROCEDURE sp_RegistrarProducto (
	@Nombre VARCHAR(100),
	@Descripcion VARCHAR(100),
	@IdMarca VARCHAR(100),
	@IdCategoria VARCHAR(100),
	@Precio DECIMAL(10, 2),
	@Stock INT,
	@Activo BIT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado INT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM PRODUCTO
			WHERE Nombre = @Nombre
			)
	BEGIN
		INSERT INTO PRODUCTO (
			Nombre,
			Descripcion,
			IdMarca,
			IdCategoria,
			Precio,
			Stock,
			Activo
			)
		VALUES (
			@Nombre,
			@Descripcion,
			@IdMarca,
			@IdCategoria,
			@Precio,
			@Stock,
			@Activo
			)

		SET @Resultado = SCOPE_IDENTITY()
	END
	ELSE
		SET @Mensaje = 'El producto ya existe'
END

```

## CD_Productos

```c#
using CapaEntidad;
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.Globalization;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaDatos
{
    public class CD_Producto
    {
        public List<Producto> Listar()
        {
            var lista = new List<Producto>();

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var sb = new StringBuilder();

                    sb.AppendLine("SELECT P.IdProducto,");
                    sb.AppendLine("	p.Nombre,");
                    sb.AppendLine("	P.Descripcion,");
                    sb.AppendLine("	M.IdMarca,");
                    sb.AppendLine("	M.Descripcion [DesMarca],");
                    sb.AppendLine("	C.IdCategoria,");
                    sb.AppendLine("	C.Descripcion [DesCategoria],");
                    sb.AppendLine("	P.Precio,");
                    sb.AppendLine("	P.Stock,");
                    sb.AppendLine("	P.RutaImagen,");
                    sb.AppendLine("	P.NombreImagen,");
                    sb.AppendLine("	P.Activo");
                    sb.AppendLine("FROM PRODUCTO AS P");
                    sb.AppendLine("INNER JOIN MARCA AS M ON M.IdMarca = P.IdMarca");
                    sb.AppendLine("INNER JOIN CATEGORIA AS C ON C.IdCategoria = P.IdCategoria");

                    var cmd = new SqlCommand()
                    {
                        CommandText = sb.ToString(),
                        Connection = oconexion,
                        CommandType = CommandType.Text
                    };

                    oconexion.Open();

                    using (SqlDataReader dr = cmd.ExecuteReader())
                    {
                        while (dr.Read())
                        {
                            lista.Add(new Producto()
                            {
                                IdProducto = Convert.ToInt32(dr["IdProducto"]),
                                Nombre = dr["Nombre"].ToString(),
                                Descripcion = dr["Descripcion"].ToString(),
                                oMarca = new Marca()
                                {
                                    IdMarca = Convert.ToInt32(dr["IdMarca"]),
                                    Descripcion = dr["DesMarca"].ToString()
                                },
                                oCategoria = new Categoria()
                                {
                                    IdCategoria = Convert.ToInt32(dr["IdCategoria"]),
                                    Descripcion = dr["DesCategoria"].ToString()
                                },
                                Precio = Convert.ToDecimal(dr["Precio"], new CultureInfo("es-NI")),
                                Stock = Convert.ToInt32(dr["Stock"]),
                                RutaImagen = dr["RutaImagen"].ToString(),
                                NombreImagen = dr["NombreImagen"].ToString(),
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

        public int Registrar(Producto obj, out string Mensaje)
        {
            int idautogenerado = 0;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var cmd = new SqlCommand()
                    {
                        CommandText = "sp_RegistrarProducto",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("Nombre", obj.Nombre);
                    cmd.Parameters.AddWithValue("Descripcion", obj.Descripcion);
                    cmd.Parameters.AddWithValue("IdMarca", obj.oMarca.IdMarca);
                    cmd.Parameters.AddWithValue("IdCategoria", obj.oCategoria.IdCategoria);
                    cmd.Parameters.AddWithValue("Precio", obj.Precio);
                    cmd.Parameters.AddWithValue("Stock", obj.Stock);
                    cmd.Parameters.AddWithValue("Activo", obj.Activo);
                    cmd.Parameters.Add("Mensaje", SqlDbType.VarChar, 500).Direction = ParameterDirection.Output;
                    cmd.Parameters.Add("Resultado", SqlDbType.Int).Direction = ParameterDirection.Output;

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

        public bool Editar(Producto obj, out string Mensaje)
        {
            bool resultado = false;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    var cmd = new SqlCommand()
                    {
                        CommandText = "sp_EditarProducto ",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("IdProducto", obj.IdProducto);
                    cmd.Parameters.AddWithValue("Nombre", obj.Nombre);
                    cmd.Parameters.AddWithValue("Descripcion", obj.Descripcion);
                    cmd.Parameters.AddWithValue("IdMarca", obj.oMarca.IdMarca);
                    cmd.Parameters.AddWithValue("IdCategoria", obj.oCategoria.IdCategoria);
                    cmd.Parameters.AddWithValue("Precio", obj.Precio);
                    cmd.Parameters.AddWithValue("Stock", obj.Stock);
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

        public bool GuardarDatosImagen(Producto obj, out string Mensaje)
        {
            bool resultado = false;
            Mensaje = string.Empty;

            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    string query = "update producto set RutaImagen = @rutaimagen, NombreImagen = @nombreimagen where IdProducto = @idproducto";
                    var cmd = new SqlCommand()
                    {
                        CommandText = query,
                        Connection = oconexion,
                        CommandType = CommandType.Text
                    };

                    cmd.Parameters.AddWithValue("rutaimagen", obj.RutaImagen);
                    cmd.Parameters.AddWithValue("nombreimagen", obj.NombreImagen);
                    cmd.Parameters.AddWithValue("idproducto", obj.IdProducto);
                    cmd.Parameters.Add("Mensaje", SqlDbType.VarChar, 500).Direction = ParameterDirection.Output;
                    cmd.Parameters.Add("Resultado", SqlDbType.Int).Direction = ParameterDirection.Output;

                    oconexion.Open();

                    

                    if (cmd.ExecuteNonQuery() > 0)
                    {
                        resultado = true;
                    }
                    else
                    {
                        Mensaje = "No se pudo actualizar imagen";
                    }
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
                        CommandText = "sp_EliminarProducto",
                        Connection = oconexion,
                        CommandType = CommandType.StoredProcedure
                    };
                    cmd.Parameters.AddWithValue("IdProducto", id);
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

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte18)
