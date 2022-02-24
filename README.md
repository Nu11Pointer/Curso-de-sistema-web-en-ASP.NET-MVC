# Curso de sistema web en ASP.NET MVC 5 + SQL Server - Parte 19

# Modificación Entidad Productos

Ha esta entidad le fue añadida tres nuevas propiedades que nos serán de utilidad en nuestro sistema, estas son: ```PrecioTexto```, ```Base64``` y ```Extension```.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Producto
    {
        public int IdProducto { get; set; }

        public string Nombre { get; set; }

        public string Descripcion { get; set; }

        public Marca oMarca { get; set; }

        public Categoria oCategoria { get; set; }

        public decimal Precio { get; set; }

        public string PrecioTexto { get; set; }

        public int Stock { get; set; }

        public string RutaImagen { get; set; }

        public string NombreImagen { get; set; }

        public bool Activo { get; set; }

        public string Base64 { get; set; }

        public string Extension { get; set; }
    }
}

```

# Capa Negocio Producto

Tambien se añadio esta clase, es practicamente igual a las demas a excepción del metodo guardar, en este metodo tendremos que realizar una operación extra, la cual depende de dos factore, el primero es si la operación de edición o registro fue realizada con exito, si esto sucede entramos al segundo factor el cual es guardar la imagen del producto si esta fue guardada con exito, el codigo esta comentariado así que puedes darle un vistaso.

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
    public class CN_Producto
    {
        private CD_Producto objCapaDato = new CD_Producto();

        public List<Producto> Listar()
        {
            return objCapaDato.Listar();
        }

        public int Registrar(Producto obj, out string Mensaje)
        {
            Mensaje = string.Empty;

            if (string.IsNullOrEmpty(obj.Nombre) || string.IsNullOrWhiteSpace(obj.Nombre))
            {
                Mensaje = "La nombre del Producto no puede ser vacio.";
            }
            else if (string.IsNullOrEmpty(obj.Descripcion) || string.IsNullOrWhiteSpace(obj.Descripcion))
            {
                Mensaje = "La descripción del Producto no puede ser vacio.";
            }
            else if (obj.oMarca.IdMarca == 0)
            {
                Mensaje = "Debe seleccionar una marca.";
            }
            else if (obj.oCategoria.IdCategoria == 0)
            {
                Mensaje = "Debe seleccionar una categoria.";
            }
            else if (obj.Precio == 0)
            {
                Mensaje = "Debe ingresar el precio del producto.";
            }
            else if (obj.Stock == 0)
            {
                Mensaje = "Debe ingresar el stock del producto.";
            }


            if (string.IsNullOrEmpty(Mensaje))
            {
                return objCapaDato.Registrar(obj, out Mensaje);
            }

            return 0;
        }

        public bool Editar(Producto obj, out string Mensaje)
        {
            Mensaje = string.Empty;

            if (string.IsNullOrEmpty(obj.Nombre) || string.IsNullOrWhiteSpace(obj.Nombre))
            {
                Mensaje = "La nombre del Producto no puede ser vacio.";
            }
            else if (string.IsNullOrEmpty(obj.Descripcion) || string.IsNullOrWhiteSpace(obj.Descripcion))
            {
                Mensaje = "La descripción del Producto no puede ser vacio.";
            }
            else if (obj.oMarca.IdMarca == 0)
            {
                Mensaje = "Debe seleccionar una marca.";
            }
            else if (obj.oCategoria.IdCategoria == 0)
            {
                Mensaje = "Debe seleccionar una categoria.";
            }
            else if (obj.Precio == 0)
            {
                Mensaje = "Debe ingresar el precio del producto.";
            }
            else if (obj.Stock == 0)
            {
                Mensaje = "Debe ingresar el stock del producto.";
            }

            if (string.IsNullOrEmpty(Mensaje))
            {
                return objCapaDato.Editar(obj, out Mensaje);
            }

            return false;
        }

        public bool GuardarDatosImagen(Producto obj, out string Mensaje)
        {
            return objCapaDato.GuardarDatosImagen(obj, out Mensaje);
        }

        public bool Eliminar(int id, out string Mensaje)
        {
            return objCapaDato.Eliminar(id, out Mensaje);
        }
    }
}

```

## Acciones Producto en el Controlador Mantenedor

De igual manera hemos añadido nuevas acciones al controlador.

```c#
#region Producto
[HttpGet]
public JsonResult ListarProducto()
{
    List<Producto> oLista = new CN_Producto().Listar();
    return Json(new { data = oLista }, JsonRequestBehavior.AllowGet);
}

[HttpPost]
public JsonResult GuardarProducto(string objeto, HttpPostedFileBase archivoImagen)
{
    string mensaje = string.Empty;
    bool operacion_exitosa = true;
    bool guardar_imagen_exito = true;
    Producto oProducto = new Producto();

    // Deserializamos el Objeto Json
    oProducto = JsonConvert.DeserializeObject<Producto>(objeto);

    decimal precio;

    // Intentamos Transformar el Precio (Como texto) a decimal.
    if (decimal.TryParse(oProducto.PrecioTexto,
        NumberStyles.AllowDecimalPoint,
        new CultureInfo("es-NI"), out precio))
    {
        oProducto.Precio = precio;
    }
    else
    {
        return Json(new { operacionExitosa = false, mensaje = "El formato del precio debe ser ##.##" },
            JsonRequestBehavior.AllowGet);
    }

    // ¿Registrar o Editar?
    if (oProducto.IdProducto == 0)
    {
        // Registramos el Producto
        int idProductoGenerado = new CN_Producto().Registrar(oProducto, out mensaje);

        // Verificamos la operación
        if (idProductoGenerado != 0)
        {
            oProducto.IdProducto = idProductoGenerado;
        }
        else
        {
            operacion_exitosa = false;
        }
    }
    else
    {
        // Editamos el producto
        operacion_exitosa = new CN_Producto().Editar(oProducto, out mensaje);
    }

    // Si la operación fue exitosa
    if (operacion_exitosa)
    {
        // ¿Se adjunto una imagen?
        if (archivoImagen != null)
        {
            // Meta-Datos de la imagen.
            string ruta_guardar = ConfigurationManager.AppSettings["ServidorFotos"];
            string extension = Path.GetExtension(archivoImagen.FileName);
            string nombre_imagen = string.Concat(oProducto.IdProducto.ToString(), extension);

            try
            {
                // Guardar Imagen
                archivoImagen.SaveAs(Path.Combine(ruta_guardar, nombre_imagen));
            }
            catch (Exception ex)
            {
                string msg = ex.Message;
                guardar_imagen_exito = false;
            }

            // ¿Se realizó con exito?
            if (guardar_imagen_exito)
            {
                oProducto.RutaImagen = ruta_guardar;
                oProducto.NombreImagen = nombre_imagen;
                bool rspta = new CN_Producto().GuardarDatosImagen(oProducto, out mensaje);
            }
            else
            {
                mensaje = "Se guardó el producto pero hubo problemas con la imagen";
            }
        }
    }

    return Json(new
    {
        operacion_exitosa = true,
        idGenerado = oProducto.IdProducto,
        mensaje = mensaje
    }, JsonRequestBehavior.AllowGet);
}

[HttpPost]
public JsonResult EliminarProducto(int id)
{
    bool respuesta = false;
    string mensaje = string.Empty;

    respuesta = new CN_Producto().Eliminar(id, out mensaje);

    return Json(new { resultado = respuesta, mensaje = mensaje }, JsonRequestBehavior.AllowGet);
}
#endregion

```

El proyecto hasta este punto lo puedes encontrar en el siguiente enlace: [Ver Proyecto](https://github.com/Nu11Pointer/CursoMVC/tree/Parte19)
