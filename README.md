# Curso de sistema web en ASP.NET MVC + SQL Server - Parte 01

## Presentación del sistema completo

El sistema web a desarrollar esta enfocado en un carrito de compras por ello vamos a desarrollar desde cero dos módulos. El primer modulo es para el administrador el cual va a gestionar todo lo que viene ser los productos dentro de la tienda y. El segundo modulo va a ser para el cliente donde va a poder realizar la compra de todos los productos.

El modulo de administración va a contar con inicio de sesión con este formulario el cual va a poner sus credenciales e iniciar el sistema.

![image-20220117175339653](https://user-images.githubusercontent.com/59342976/149868113-1e4143a3-534c-478b-b033-3850e2fb28c8.png)

Contara con el siguiente dashboard donde aparece un resumen del total de clientes, nuestras ventas y la cantidad de productos que tenemos actualmente, también contaremos con un historial de ventas donde podemos ver todas las ventas que se han realizado.

![image-20220117175738846](https://user-images.githubusercontent.com/59342976/149868132-1eeb6316-5e37-4c5c-a7da-8506ab2764bf.png)

El apartado de usuarios donde se manipulan estos con la acciones de crear, editar, eliminar.

![image-20220117180249300](https://user-images.githubusercontent.com/59342976/149868158-4b2e7baa-5246-403d-9b67-adc724d9dcd0.png)

El apartado de categorías es donde se puede crear, editar y eliminar todas las categorías que le correspondan a nuestros productos.

![image-20220117180343386](https://user-images.githubusercontent.com/59342976/149868187-9c3ee494-3cfe-49e4-a912-2907636749e2.png)

De igual forma con el apartado de marcas.

![image-20220117180439968](https://user-images.githubusercontent.com/59342976/149868232-0a4c205f-ba89-433b-af42-248c4958b065.png)

De la misma manera podemos crear, editar o eliminar productos.

![image-20220117180516356](https://user-images.githubusercontent.com/59342976/149868260-a4cc372b-7c5e-420a-9985-21e977b80548.png)

Este es el carrito de compras donde un cliente podrá realizar compras de los productos disponibles.

![image-20220117182009082](https://user-images.githubusercontent.com/59342976/149868285-ed59f1b4-abea-43de-8d8c-a16e3a0e4478.png)

![image-20220117182321332](https://user-images.githubusercontent.com/59342976/149868295-6b49499a-0523-4a40-8cca-833fd80c6325.png)

## Base de datos

```sql
USE master

IF EXISTS (
		SELECT *
		FROM sysdatabases
		WHERE (name = 'DBCARRITO')
		)
BEGIN
	DROP DATABASE DBCARRITO
END

CREATE DATABASE DBCARRITO

GO
USE DBCARRITO

CREATE TABLE CATEGORIA (
	IdCategoria INT PRIMARY KEY identity,
	Descripcion VARCHAR(100),
	Activo INT DEFAULT 1,
	FechaRegistro DATETIME DEFAULT getdate()
	)

CREATE TABLE MARCA (
	IdMarca INT PRIMARY KEY identity,
	Descripcion VARCHAR(100),
	Activo BIT DEFAULT 1,
	FechaRegistro DATETIME DEFAULT getdate()
	)

CREATE TABLE PRODUCTO (
	IdProducto INT PRIMARY KEY identity,
	Nombre VARCHAR(500),
	Descripcion VARCHAR(500),
	IdMarca INT REFERENCES Marca(IdMarca),
	IdCategoria INT REFERENCES Categoria(IdCategoria),
	Precio DECIMAL(10, 2) DEFAULT 0,
	Stock INT,
	RutaImagen VARCHAR(100),
	NombreImagen VARCHAR(100),
	Activo BIT DEFAULT 1,
	FechaRegistro DATETIME DEFAULT getdate()
	)

CREATE TABLE CLIENTE (
	IdCliente INT PRIMARY KEY identity,
	Nombres VARCHAR(100),
	Apellidos VARCHAR(100),
	Clave VARCHAR(150),
	Reestablecer BIT DEFAULT 0,
	FechaRegistro DATETIME DEFAULT getdate()
	)

CREATE TABLE CARRITO (
	IdCarrito INT PRIMARY KEY identity,
	IdCliente INT REFERENCES CLIENTE(IdCliente),
	IdProducto INT REFERENCES PRODUCTO(IdProducto),
	Cantidad INT
	)

CREATE TABLE VENTA (
	IdVenta INT PRIMARY KEY identity,
	IdCliente INT REFERENCES CLIENTE(idCliente),
	TotalProducto INT,
	MontoTotal DECIMAL(10, 2),
	Contacto VARCHAR(50),
	IdDistrito VARCHAR(10),
	Telefono VARCHAR(50),
	Direccion VARCHAR(500),
	IdTransaccion VARCHAR(50),
	FechaVenta DATETIME DEFAULT getdate()
	)

CREATE TABLE DETALLE_VENTA (
	IdDetalleVenta INT PRIMARY KEY identity,
	IdVenta INT REFERENCES VENTA(IdVenta),
	IdProducto INT REFERENCES PRODUCTO(IdProducto),
	Cantidad INT,
	Total DECIMAL(10, 2)
	)

CREATE TABLE USUARIO (
	IdUsuario INT PRIMARY KEY identity,
	Nombres VARCHAR(100),
	Apellidos VARCHAR(100),
	Correo VARCHAR(100),
	Clave VARCHAR(100),
	Reestablecer BIT DEFAULT 1,
	Activo BIT DEFAULT 1,
	FechaRegistro DATETIME DEFAULT getdate()
	)

CREATE TABLE DEPARTAMENTO (
	IdDepartamento VARCHAR(2) NOT NULL,
	Descripcion VARCHAR(45) NOT NULL
	)

CREATE TABLE PROVINCIA (
	IdProvincia VARCHAR(4) NOT NULL,
	Descripcion VARCHAR(45) NOT NULL,
	IdDepartamento VARCHAR(2) NOT NULL
	)

CREATE TABLE DISTRITO (
	IdDistrito VARCHAR(6) NOT NULL,
	Descripcion VARCHAR(45) NOT NULL,
	IdProvincia VARCHAR(4) NOT NULL,
	IdDepartamento VARCHAR(2) NOT NULL
	)
```
![image](https://user-images.githubusercontent.com/59342976/150464979-9a5e644e-ae2e-457a-bfbe-fe7c27790184.png)
