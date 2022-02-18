# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 10

## Procedimientos Almacenados

## sp_RegistrarUsuario

```sql
USE DBCARRITO
GO

CREATE PROCEDURE sp_RegistrarUsuario (
	@Nombres VARCHAR(100),
	@Apellidos VARCHAR(100),
	@Correo VARCHAR(100),
	@Clave VARCHAR(100),
	@Activo BIT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado INT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM USUARIO
			WHERE CORREO = @Correo
			)
	BEGIN
		INSERT INTO USUARIO (
			Nombres,
			Apellidos,
			Correo,
			Clave,
			Activo
			)
		VALUES (
			@Nombres,
			@Apellidos,
			@Correo,
			@Clave,
			@Activo
			)

		SET @Resultado = SCOPE_IDENTITY()
	END
	ELSE
		SET @Mensaje = 'El correo del usuario ya existe'
END

```

## sp_EditarUsuario

```sql
USE DBCARRITO
GO

CREATE PROCEDURE sp_EditarUsuario (
	@IdUsuario INT,
	@Nombres VARCHAR(100),
	@Apellidos VARCHAR(100),
	@Correo VARCHAR(100),
	@Activo BIT,
	@Mensaje VARCHAR(500) OUTPUT,
	@Resultado BIT OUTPUT
	)
AS
BEGIN
	SET @Resultado = 0

	IF NOT EXISTS (
			SELECT *
			FROM USUARIO
			WHERE Correo = @Correo
				AND IdUsuario != @IdUsuario
			)
	BEGIN
		UPDATE TOP (1) USUARIO
		SET Nombres = @Nombres,
			Apellidos = @Apellidos,
			Correo = @Correo,
			Activo = @Activo
		WHERE IdUsuario = @IdUsuario

		SET @Resultado = 1
	END
	ELSE
		SET @Mensaje = 'El Correo del usuario ya existe'
END

```

# Capada Datos

## CD_Usuarios

### Registrar

```c#
public int Registrar(Usuario obj, out string Mensaje)
{
    int idautogenerado = 0;
    Mensaje = string.Empty;
    try
    {
        using (var oconexion = new SqlConnection(Conexion.cn))
        {
            var cmd = new SqlCommand()
            {
                CommandText = "sp_RegistrarUsuario",
                Connection = oconexion,
                CommandType = CommandType.StoredProcedure
            };
            cmd.Parameters.AddWithValue("Nombres", obj.Nombres);
            cmd.Parameters.AddWithValue("Apellidos", obj.Apellidos);
            cmd.Parameters.AddWithValue("Correo", obj.Correo);
            cmd.Parameters.AddWithValue("Clave", obj.Clave);
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

```

### Editar

```c#
public bool Editar(Usuario obj, out string Mensaje)
{
    bool resultado = false;
    Mensaje = string.Empty;
    try
    {
        using (var oconexion = new SqlConnection(Conexion.cn))
        {
            var cmd = new SqlCommand()
            {
                CommandText = "sp_EditarUsuario",
                Connection = oconexion,
                CommandType = CommandType.StoredProcedure
            };
            cmd.Parameters.AddWithValue("IdUsuario", obj.IdUsuario);
            cmd.Parameters.AddWithValue("Nombres", obj.Nombres);
            cmd.Parameters.AddWithValue("Apellidos", obj.Apellidos);
            cmd.Parameters.AddWithValue("Correo", obj.Correo);
            cmd.Parameters.AddWithValue("Clave", obj.Clave);
            cmd.Parameters.AddWithValue("Activo", obj.Activo);
            cmd.Parameters.Add("Resultado", SqlDbType.Int).Direction = ParameterDirection.Output;
            cmd.Parameters.Add("Mensaje", SqlDbType.VarChar, 500).Direction = ParameterDirection.Output;
            
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

```

### Eliminar

```c#
public bool Eliminar(int id, out string Mensaje)
{
    bool resultado = false;
    Mensaje = string.Empty;
    try
    {
        using (var oconexion = new SqlConnection(Conexion.cn))
        {
            var cmd = new SqlCommand("dele top (1) from usuario where IdUsuario = @id", oconexion);
            cmd.Parameters.AddWithValue("@id", id);
            oconexion.Open();
            resultado = cmd.ExecuteNonQuery() > 0;
        }
    }
    catch (Exception e)
    {
        resultado = false;
        Mensaje = e.Message;
    }
    return resultado;
}

```

# Capa Negocio

## CN_Recursos

```c#
using System.Security.Cryptography;
using System.Text;

namespace CapaNegocio
{
    public class CN_Recursos
    {
        public static string ConvertirSha256(string texto)
        {
            StringBuilder Sb = new StringBuilder();

            using (SHA256 hash = SHA256.Create())
            {
                Encoding enc = Encoding.UTF8;
                byte[] result = hash.ComputeHash(enc.GetBytes(texto));

                foreach (byte b in result)
                {
                    Sb.Append(b.ToString("x2"));
                }
            }

            return Sb.ToString();
        }
    }
}

```

## CN_Usuarios

### Registrar

```c#
public int Registrar(Usuario obj, out string Mensaje)
{
    Mensaje = string.Empty;
    if (string.IsNullOrEmpty(obj.Nombres) || string.IsNullOrWhiteSpace(obj.Nombres))
    {
        Mensaje = "El nombre del usuario no puede ser vacio.";
    }
    else if (string.IsNullOrEmpty(obj.Apellidos) || string.IsNullOrWhiteSpace(obj.Apellidos))
    {
        Mensaje = "El apellido del usuario no puede ser vacio.";
    }
    else if (string.IsNullOrEmpty(obj.Correo) || string.IsNullOrWhiteSpace(obj.Correo))
    {
        Mensaje = "El correo del usuario no puede ser vacio.";
    }
    if (string.IsNullOrEmpty(Mensaje))
    {
        string clave = "test123";
        obj.Clave = CN_Recursos.ConvertirSha256(clave);
        return objCapaDato.Registrar(obj, out Mensaje);
    }
    return 0;
}

```

### Editar

```c#
public bool Editar(Usuario obj, out string Mensaje)
{
    Mensaje = string.Empty;
    if (string.IsNullOrEmpty(obj.Nombres) || string.IsNullOrWhiteSpace(obj.Nombres))
    {
        Mensaje = "El nombre del usuario no puede ser vacio.";
    }
    else if (string.IsNullOrEmpty(obj.Apellidos) || string.IsNullOrWhiteSpace(obj.Apellidos))
    {
        Mensaje = "El apellido del usuario no puede ser vacio.";
    }
    else if (string.IsNullOrEmpty(obj.Correo) || string.IsNullOrWhiteSpace(obj.Correo))
    {
        Mensaje = "El correo del usuario no puede ser vacio.";
    }
    if (string.IsNullOrEmpty(Mensaje))
    {
        return objCapaDato.Editar(obj, out Mensaje);
    }
    return false;
}

```


### Eliminar

```c#
public bool Eliminar(int id, out string Mensaje)
{
    return objCapaDato.Eliminar(id, out Mensaje);
}

```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte10)
