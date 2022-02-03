# Curso de sistema web en ASP.NET MVC + SQL Server - Parte 04

## Correciones al diseño

![image](https://user-images.githubusercontent.com/59342976/152249787-cd19e446-63b2-4a0e-8b93-89e3388b91f8.png)

En el capitulo anterior olvidamos copiar la clase presentada en nuestro **_Layaout.cshtml** por lo que procederemos a pegarla.

```html
<body class="sb-nav-fixed">
```

## Creación de Capas

![image](https://user-images.githubusercontent.com/59342976/152250382-40f1baac-61d0-4d83-9a7e-98da51b21cf2.png)

Crearemos nuevos proyectos de tipo **Biblioteca de clases (.NET Framework)** tal como se muestra en la imagen anterior.

![image](https://user-images.githubusercontent.com/59342976/152251976-1e3cd91f-4bae-4f71-8f98-0f38ffcd3525.png)

Procederemos a crear las capas mostradas en la imagen anterior:

* CapaDatos
* CapaEntidad
* CapaNegocio

## Capa Entidad

![image](https://user-images.githubusercontent.com/59342976/152261384-e16f5630-ebb5-438c-87f8-124093eba7bf.png)

La capa entidad es la que se encarga de representar las tablas de nuestra base de datos. Por ende tendremos que crear una clase para cada tabla como se muestra en la imagen.

### Categoria

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Categoria
    {
        public int IdCategoria { get; set; }

        public string Descrípcion { get; set; }

        public bool Actovp { get; set; }
    }
}
```

### Marca

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Marca
    {
        public int IdMarca { get; set; }

        public string Descripcion { get; set; }

        public bool Activo { get; set; }
    }
}
```

### Producto

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

        public int Stock { get; set; }

        public string RutaImagen { get; set; }

        public string NombreImagen { get; set; }

        public bool Activo { get; set; }
    }
}
```

### Cliente

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Cliente
    {
        public int IdCliente { get; set; }

        public string Nombres { get; set; }

        public string Correo { get; set; }

        public string Clave { get; set; }

        public bool Restablecer { get; set; }
    }
}
```

### Carrito

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Carrito
    {
        public int IdCarrito { get; set; }

        public Cliente oCliente { get; set; }

        public Producto oProducto { get; set; }

        public int Cantidad { get; set; }
    }
}
```

### DetalleVenta

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class DetalleVenta
    {
        public int IdDetalleVenta { get; set; }

        public int IdVenta { get; set; }

        public Producto oProducto { get; set; }

        public int Cantidad { get; set; }

        public decimal Total { get; set; }
    }
}
```

### Venta

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Venta
    {
        public int IdVenta { get; set; }

        public int IdCliente { get; set; }

        public int TotalProducto { get; set; }

        public decimal MontoTotal { get; set; }

        public string Contacto { get; set; }

        public string IdDistrito { get; set; }

        public string Telefono { get; set; }

        public string Direccion { get; set; }

        public string FechaTexto { get; set; }

        public string IdTransaccion { get; set; }

        public List<DetalleVenta> oDetalleVenta { get; set; }
    }
}
```

### Usuario

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Usuario
    {
        public int IdUsuario { get; set; }

        public string Nombres { get; set; }

        public string Apellidos { get; set; }

        public string Correo { get; set; }

        public string Clave { get; set; }

        public bool Reestablecer { get; set; }

        public bool Activo { get; set; }
    }
}
```

### Departamento

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Departamento
    {
        public string IdDepartamento { get; set; }

        public string Descripcion { get; set; }
    }
}
```

### Provincia

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Provincia
    {
        public string IdProvincia { get; set; }

        public string Descripcion { get; set; }
    }
}
```

### Distrito

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CapaEntidad
{
    public class Distrito
    {
        public string IdDistrito { get; set; }

        public string Descripcion { get; set; }
    }
}
```

## Modificando el NavBar

![image](https://user-images.githubusercontent.com/59342976/152263913-eac4be79-55b7-4533-8ea0-6bbea5da1df5.png)

Empezaremos modifcando la marca del navbar.

```html
<!-- Navbar Brand-->
<a class="navbar-brand ps-3" href="index.html">Mi Tienda</a>
```

![image](https://user-images.githubusercontent.com/59342976/152264410-06cf1a72-97c1-4239-8254-2536f6affe8e.png)

Luego procederemos a quitar el NavBar Search y cambiar las propiedades de Navbar.

```html
<!-- Navbar-->
<ul class="navbar-nav ms-auto me-0 me-md-3 my-2 my-md-0">
	<li class="nav-item dropdown">
		<a class="nav-link dropdown-toggle" id="navbarDropdown" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false"><i class="fas fa-user fa-fw"></i></a>
		<ul class="dropdown-menu dropdown-menu-end" aria-labelledby="navbarDropdown">
			<li><a class="dropdown-item" href="#!">Correo del Usuario</a></li>
			<li><hr class="dropdown-divider" /></li>
			<li><a class="dropdown-item" href="#!">Cerrar Sesion</a></li>
		</ul>
	</li>
</ul>
```

![image](https://user-images.githubusercontent.com/59342976/152266273-6aa0f490-0357-4ed2-b6e1-9b76de8e0c7c.png)

Luego eliminaremos el siguiente fragmento de codigo:

```html
<div class="sb-sidenav-menu-heading">Interface</div>
<a class="nav-link collapsed">...</a>
<div id="collapseLayouts">...</a>
<a class="nav-link collapsed">...</a>
<div id="collapsePages" class="collapse">...</div>
```

y para finalizar modificaremos **sb-sidenav-menu** de la siguiente forma:

```html
<div class="sb-sidenav-menu">
	<div class="nav">
		<div class="sb-sidenav-menu-heading">Resumen</div>
		<a class="nav-link" href="index.html">
			<div class="sb-nav-link-icon"><i class="fas fa-tachometer-alt"></i></div>
			Dashboard
		</a>
		<a class="nav-link" href="index.html">
			<div class="sb-nav-link-icon"><i class="fas fa-users"></i></div>
			Usuarios
		</a>
		<div class="sb-sidenav-menu-heading">Mantenimiento</div>
		<a class="nav-link" href="charts.html">
			<div class="sb-nav-link-icon"><i class="fas fa-table"></i></div>
			Categorias
		</a>
		<a class="nav-link" href="tables.html">
			<div class="sb-nav-link-icon"><i class="fas fa-bookmark"></i></div>
			Marcas
		</a>
		<a class="nav-link" href="tables.html">
			<div class="sb-nav-link-icon"><i class="fas fa-boxes"></i></div>
			Productos
		</a>
	</div>
</div>
```
