# Curso de sistema web en ASP.NET MVC + SQL Server - Parte 05

## Creando Controladores y vistas

![image](https://user-images.githubusercontent.com/59342976/152604226-654779ae-54e7-4351-a2f4-6671f89895a9.png)

Cuando le damos click en algun boton, nos redirecciona al sitio que se muestra en la imagen anterior indicandonos que ha ocurrido un error y el enlace al sitio solicitado no existe. 

![image](https://user-images.githubusercontent.com/59342976/152615973-90d1fd79-e7cf-4148-b290-9ed73d8d65b4.png)


Esto es porque no tenemos vistas funcionales en nuestro proyecto, las unicas que tenemos actuales son del controlador **HomeControler** (**Index**, **About** y **Contact**). Para solucionar este problema entraremos al controlador **Home** y eliminaremos todas los metodos a excepcion de **Index()**, también eliminaremos las vistas correspondientes a dichos metodos las cuales se cuentran en **Views/Home/**. Creamos el metodo **Usuario()** y con click derecho agregamos una vista MVC5. Nos debería quedar algo como lo siguiente:

### HomeController

```c#
namespace CapaPresentacionAdmin.Controllers
{
    public class HomeController : Controller
    {
        public ActionResult Index()
        {
            return View();
        }
        
        public ActionResult Usuarios()
        {
            return View();
        }
    }
}
```

![image](https://user-images.githubusercontent.com/59342976/152616269-95b25cb5-05c9-427f-9f18-4c9c9e914035.png)

Tambien crearemos el controlador **Mantenedor** como se muestra en la imagen anterior, con sus respectivos metodos y vistas:

```c#
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
    }
}
```

![image](https://user-images.githubusercontent.com/59342976/152616500-77586bc8-03b7-4248-8a3e-b3fbe2f922d2.png)

## Formulario Usuarios

Lo primero que haremos será crear nuestra cadena de conexión, dentro de **Web.config** en mi caso es la siguiente:

```xml
<connectionStrings>
	<add name="cadena" providerName="System.Data.ProviderName" connectionString="Data Source=(local);Initial Catalog=DBCARRITO;User ID=sa;Password=123;"/>
</connectionStrings>
```

![image](https://user-images.githubusercontent.com/59342976/152628255-0f35082a-b2b3-4e59-9734-c3458aad020d.png)

Verificaremos que tengamos la referencia a **System.Configuration** para esto abriremos el Administrador de referencias del proyecto **CapaDatos** haciendo click derecho > Agregar > Referencia > Ensamblados. Y procederemos a crear la clase **Conexion** que tendra esta estructura:

### Conexion

```c#
using System.Configuration;

namespace CapaDatos
{
    public class Conexion
    {
        public static string cn = ConfigurationManager.ConnectionStrings["cadena"].ToString();
    }
}
```

Tambien deberemos agregar las siguientes referencias:

* CapaDatos
	* CapaEntidad
* CapaNegocio
	* CapaEntidad
	* CapaDatos
* CapaPresentacionAdmin
	* CapaEntidad
	* CapaNegocio
* CapaPresentacionTienda
	* CapaEntidad
	* CapaNegocio

Dentro de **CapaDatos** crearemos la clase **CD_Usuarios** que tendrá la siguiente estructura:

```c#
using CapaDatos;
using CapaEntidad;
using System.Collections.Generic;

namespace CapaNegocio
{
    public class CN_Usuarios
    {
        private CD_Usuarios objCapaDato = new CD_Usuarios();

        public List<Usuario> Listar()
        {
            return objCapaDato.Listar();
        }
    }
}
```
Luego dentro de **CapaNegocio** crearemos la clase **CN_Usuarios**

```C#
using CapaEntidad;
using System;
using System.Collections.Generic;
using System.Data.SqlClient;
using System.Data;

namespace CapaDatos
{
    public class CD_Usuarios
    {
        public List<Usuario> Listar()
        {
            var lista = new List<Usuario>();
            try
            {
                using (var oconexion = new SqlConnection(Conexion.cn))
                {
                    string query = "select IdUsuario, Nombres, Apellidos, Correo, Clave, Reestablecer, Activo from USUARIO";
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
                            lista.Add(new Usuario()
                            {
                                IdUsuario = Convert.ToInt32(dr["IdUsuario"]),
                                Nombres = dr["Nombres"].ToString(),
                                Apellidos = dr["Apellidos"].ToString(),
                                Correo = dr["Correo"].ToString(),
                                Clave = dr["Clave"].ToString(),
                                Reestablecer = Convert.ToBoolean(dr["Reestablecer"]),
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
    }
}
```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte05)
