# Curso de sistema web en ASP.NET MVC + SQL Server - Parte 02

## Añadiendo Registros a la Base de Datos

```sql
USE DBCARRITO

INSERT INTO USUARIO (
	Nombres,
	Apellidos,
	Correo,
	Clave
	)
VALUES (
	'test nombre',
	'test apellido',
	'test@example.com',
	'ecd71870d1963316a97e3ac3408c9835ad8cf0f3c1bc703527c30265534f75ae'
	)

INSERT INTO CATEGORIA (Descripcion)
VALUES ('Tecnologia'),
	('Muebles'),
	('Dormitorio'),
	('Deportes')

INSERT INTO MARCA (Descripcion)
VALUES ('SONYTE'),
	('HPTE'),
	('LGTE'),
	('HYUNDAITE'),
	('CANONTE'),
	('ROBERTA ALLENTE')

INSERT INTO DEPARTAMENTO (
	IdDepartamento,
	Descripcion
	)
VALUES (
	'01',
	'Arequipa'
	),
	(
	'02',
	'Ica'
	),
	(
	'03',
	'Lima'
	)

INSERT INTO PROVINCIA (
	IdProvincia,
	Descripcion,
	IdDepartamento
	)
VALUES (
	'0101',
	'Arequipa',
	'01'
	),
	(
	'0102',
	'Camaná',
	'01'
	),
	--ICA - PROVINCIAS
	(
	'0201',
	'Ica ',
	'02'
	),
	(
	'0202',
	'Chincha ',
	'02'
	),
	--LIMA - PROVINCIAS
	(
	'0301',
	'Lima ',
	'03'
	),
	(
	'0302',
	'Barranca ',
	'03'
	)

INSERT INTO DISTRITO (
	IdDistrito,
	Descripcion,
	IdProvincia,
	IdDepartamento
	)
VALUES (
	'010101',
	'Nieva',
	'0101',
	'01'
	),
	(
	'010102',
	'El Cenepa',
	'0101',
	'01'
	),
	(
	'010201',
	'Camaná',
	'0102',
	'01'
	),
	(
	'010202',
	'José María Quimper',
	'0102',
	'01'
	),
	--ICA - DISTRITO
	(
	'020101',
	'Ica',
	'0201',
	'02'
	),
	(
	'020102',
	'La Tinguiña',
	'0201',
	'02'
	),
	(
	'020201',
	'Chincha Alta',
	'0202',
	'02'
	),
	(
	'020202',
	'Alto Laran',
	'0202',
	'02'
	),
	--LIMA - DISTRITO
	(
	'030101',
	'Lima',
	'0301',
	'03'
	),
	(
	'030102',
	'Ancón',
	'0301',
	'03'
	),
	(
	'030201',
	'Barranca',
	'0302',
	'03'
	),
	(
	'030202',
	'Paramonga',
	'0302',
	'03'
	)
```

## Creando Proyecto

Utilizaremos la plantilla **Aplicación web ASP.NET (.NET Framework) de Visual Studio 2019**.

![image](https://user-images.githubusercontent.com/59342976/150624287-69f133d6-90d0-4d9c-9e98-ce8f84e34f7b.png)

Utilizaremos las siguientes configuraciónes para crear el proyecto.

![image](https://user-images.githubusercontent.com/59342976/150624335-f25fb88c-ecb6-4bab-9258-2b2ed3713239.png)

![image](https://user-images.githubusercontent.com/59342976/150624357-6045e57f-d378-4e2b-9098-0dbc376b4b80.png)

Agregamos un nuevo proyecto

![image](https://user-images.githubusercontent.com/59342976/150624478-c441ab4a-ca00-430f-bbda-56ea4db6ddff.png)

![image](https://user-images.githubusercontent.com/59342976/150624503-4882abe5-232f-411d-b08e-e71289db8cd8.png)

![image](https://user-images.githubusercontent.com/59342976/150624521-9d4c0ecd-d098-42e5-bcd5-3a6fa0f8f014.png)

![image](https://user-images.githubusercontent.com/59342976/150624525-274781f0-7c5a-4e62-965f-65da73552773.png)

Al proyecto CapaPresentacionAdmin lo estableceremos como proyecto de inicio.

![image](https://user-images.githubusercontent.com/59342976/150624559-c3ddc2e6-bd83-4362-8ad1-b0f1b9ebda5e.png)

Una vez ejecutamos el proyecto podemos observar la plantilla por defecto.

![image](https://user-images.githubusercontent.com/59342976/150624829-b3c9a3f7-5867-45a3-94d5-66682f46d18f.png)
