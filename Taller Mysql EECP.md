

# **Taller Mysql**

Este taller está diseñado para profundizar en el manejo y optimización de bases de datos MySQL. A través de ejercicios prácticos, se explorarán temas avanzados para reforzar el conocimiento en normalización, joins, consultas complejas, subconsultas, procedimientos almacenados, funciones definidas por el usuario y triggers.

BASE DE DATOS:

```sql
-- Creación de la base de datos
CREATE DATABASE vtaszfs;
USE vtaszfs;
-- Tabla Clientes
CREATE TABLE Clientes (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
email VARCHAR(100) UNIQUE
    );
-- Tabla UbicacionCliente
CREATE TABLE UbicacionCliente (
id INT PRIMARY KEY AUTO_INCREMENT,
cliente_id INT,
direccion VARCHAR(255),
ciudad VARCHAR(100),
estado VARCHAR(50),
codigo_postal VARCHAR(10),
pais VARCHAR(50),
FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
-- Tabla Empleados
CREATE TABLE Empleados (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),puesto VARCHAR(50),
salario DECIMAL(10, 2),
fecha_contratacion DATE
);
-- Tabla Proveedores
CREATE TABLE Proveedores (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
contacto VARCHAR(100),
telefono VARCHAR(20),
direccion VARCHAR(255)
);
-- Tabla TiposProductos
CREATE TABLE TiposProductos (
id INT PRIMARY KEY AUTO_INCREMENT,
tipo_nombre VARCHAR(100),
descripcion TEXT
);
-- Tabla Productos
CREATE TABLE Productos (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
precio DECIMAL(10, 2),
proveedor_id INT,
tipo_id INT,
FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
FOREIGN KEY (tipo_id) REFERENCES TiposProductos(id)
);
-- Tabla Pedidos
CREATE TABLE Pedidos (
id INT PRIMARY KEY AUTO_INCREMENT,
cliente_id INT,
fecha DATE,
total DECIMAL(10, 2),
FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
-- Tabla DetallesPedido
CREATE TABLE DetallesPedido (
id INT PRIMARY KEY AUTO_INCREMENT,
pedido_id INT,
producto_id INT,
cantidad INT,
precio DECIMAL(10, 2),
FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
FOREIGN KEY (producto_id) REFERENCES Productos(id)
);
    
```

#### **Normalización**

1. Crear una tabla HistorialPedidos que almacene cambios en los pedidos.

```sql
CREATE TABLE historialpedidos(
id INT AUTO_INCREMENT PRIMARY KEY,
id_pedidos INT,
fecha_cambio DATE,
total_cambio DECIMAL (10, 2),
id_empleado INT,
CONSTRAINT id_pedidos_fk FOREIGN KEY (id_pedidos) REFERENCES Pedidos(id),
CONSTRAINT id_empleados_fk FOREIGN KEY (id_empleados) REFERENCES Empleados(id)

);
```



2. Evaluar la tabla Clientes para eliminar datos redundantes y normalizar hasta 3NF.

```mysql


CREATE TABLE TipoDocumento (
  id INT AUTO_INCREMENT PRIMARY KEY,
  tipo VARCHAR(50) UNIQUE
);

CREATE TABLE TipoEmail (
id INT PRIMARY KEY AUTO_INCREMENT,
tipo_email VARCHAR(50) UNIQUE
);

CREATE TABLE Clientes (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
tipo_documento_id INT,
tipo_email_id INT,
CONSTRAINT fk_tipo_documento FOREIGN KEY (tipo_documento_id) REFERENCES TipoDocumento(id),
CONSTRAINT fk_tipo_email_id FOREIGN KEY (tipo_email_id) REFERENCES TipoEmail(id)
);

```

3. Separar la tabla Empleados en una tabla de DatosEmpleados y otra para Puestos .

```sql
CREATE TABLE Puesto (
id INT AUTO_INCREMENT PRIMARY KEY,
nombre_puesto VARCHAR(50)

)

CREATE TABLE DatosEmpleados (
id INT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(50),
salario DECIMAL(10,2),
fecha_contratacion DATE,
id_puesto INT,
CONSTRAINT id_puesto_fk FOREIGN KEY (id_puesto) REFERENCES Puesto(id)

);
```



4. Revisar la relación Clientes y UbicacionCliente para evitar duplicación de datos.

**Aquí debemos hacer tablas nuevas para ciudad, estado y país, esto con el fin de manejar varias direcciones si el cliente posee varias**.

```sql
CREATE TABLE UbicacionCliente (
  id INT AUTO_INCREMENT PRIMARY KEY,
  cliente_id INT,
  direccion VARCHAR(200),
  ciudad_id INT,
  codigo_postal VARCHAR(10),
  CONSTRAINT fk_cliente FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
  CONSTRAINT fk_ciudad FOREIGN KEY (ciudad_id) REFERENCES Ciudades(id)
);


CREATE TABLE Paises (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(50) UNIQUE
);

CREATE TABLE Estados (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(50),
  pais_id INT,
  CONSTRAINT fk_pais FOREIGN KEY (pais_id) REFERENCES Paises(id)
);


CREATE TABLE Ciudades (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(50),
  estado_id INT,
  CONSTRAINT fk_estado FOREIGN KEY (estado_id) REFERENCES Estados(id)
);

```

5. Normalizar Proveedores para tener ContactoProveedores en otra tabla.

```sql
CREATE TABLE Proveedores (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(50),
direccion VARCHAR(255)
);

CREATE TABLE ContactoProveedores(
id INT AUTO_INCREMENT PRIMARY KEY,
id_proveedor INT,
nombre_contacto VARCHAR(50),
telefono VARCHAR(20),
ciudades_id INT,
CONSTRAINT fk_ciudades_id FOREIGN KEY(ciudades_id) REFERENCES Ciudades(id),
CONSTRAINT fk_proveedor_id FOREIGN KEY (id_proveedor) REFERENCES Proveedores(id)

);
```

6. Crear una tabla de Telefonos para almacenar múltiples números por cliente.

```sql
CREATE TABLE telefonos (
id INT AUTO_INCREMENT PRIMARY KEY,
telefono VARCHAR(20),
id_cliente INT,
CONSTRAINT fk_id_cliente FOREIGN KEY (id_cliente) REFERENCES Clientes(id)
);
```

7. Transformar TiposProductos en una relación categórica jerárquica.

```sql

CREATE TABLE Categorias (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre_categoria VARCHAR(100)
);

CREATE TABLE Subcategorias (
  id INT AUTO_INCREMENT PRIMARY KEY,
  categoria_id INT,
  nombre_subcategoria VARCHAR(100),
  descripcion TEXT,
  CONSTRAINT fk_categoria FOREIGN KEY (categoria_id) REFERENCES Categorias(id)
);


CREATE TABLE Productos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100),
  precio DECIMAL(10, 2),
  proveedor_id INT,
  subcategoria_id INT,
  CONSTRAINT fk_proveedor FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
  CONSTRAINT fk_subcategoria FOREIGN KEY (subcategoria_id) REFERENCES Subcategorias(id)
);

```

8. Normalizar Pedidos y DetallesPedido para evitar inconsistencias de precios.

```sql
CREATE TABLE Pedidos (
id INT PRIMARY KEY AUTO_INCREMENT,
cliente_id INT,
fecha DATE,
total DECIMAL(10, 2),
FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

CREATE TABLE DetallesPedido (
id INT PRIMARY KEY AUTO_INCREMENT,
pedido_id INT,
producto_id INT,
cantidad INT,
precio_unidad DECIMAL(10, 2),
FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
FOREIGN KEY (producto_id) REFERENCES Productos(id)
);
```



9. Usar una relación de muchos a muchos para Empleados y Proveedores 

```sql
CREATE TABLE EmpleadosProveedores (
  id INT AUTO_INCREMENT PRIMARY KEY,
  empleado_id INT,
  proveedor_id INT,
  rol VARCHAR(50),
  CONSTRAINT fk_empleado FOREIGN KEY (empleado_id) REFERENCES DatosEmpleados(id),
  CONSTRAINT proveedor_fk FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);

```



10. Convertir la tabla UbicacionCliente en una relación genérica de Ubicaciones .

```sql
CREATE TABLE Ubicaciones (
  id INT AUTO_INCREMENT PRIMARY KEY,
  direccion VARCHAR(200),
  ciudad VARCHAR(80),
  estado VARCHAR(50),
  codigo_postal VARCHAR(10),
  pais VARCHAR(50)
);

CREATE TABLE UbicacionEntidad (
  id INT AUTO_INCREMENT PRIMARY KEY,
  ubicacion_id INT,
  tipo_entidad ENUM('Cliente', 'Proveedor', 'Empleado'),
  entidad_id INT,
  CONSTRAINT fk_ubicacion FOREIGN KEY (ubicacion_id) REFERENCES Ubicaciones(id)
);
```

INGRESO DE DATOS:

Se ingresaron solo 5 usuarios, cada uno con diferente tipo de documento de identidad y de email.

```SQL
INSERT INTO TipoDocumento (tipo) VALUES ('CC'),('CE'),('PASAPORTE'),('TI');
INSERT INTO TipoEmail (tipo_email) VALUES ('Personal'), ('Empresarial'), ('Otro');
INSERT INTO Clientes (nombre, tipo_documento_id, tipo_email_id)
    VALUES 
    ('María Fernanda López', 1, 1),
    ('Carlos Andrés Rodríguez', 2, 2),
    ('Ana Sofía Ramírez', 3, 1),
    ('Juan Camilo Martínez', 1, 3),
    ('Laura Valentina Torres', 2, 2);
    
```

Ingresamos ahora las categorias de los productos:

```sql
INSERT INTO Categorias (nombre_categoria) VALUES ('Electrónica')('Cocina')('Automovil')('Aseo');
```

Ingresamos las subcategorias :

```sql
INSERT INTO Subcategorias (nombre_subcategoria, categoria_id) VALUES ('Celulares', 1),('Televisores', 1),('Utensilios de cocina', 2),('Llantas', 3),('Accesorios para auto', 3),('Limpieza general', 4);

```

Ingresamos los proveedores

```sql
INSERT INTO Proveedores (nombre) VALUES 
    ('TecnoDistribuciones S.A.S'),
    ('ElectroMundo Ltda.'),
    ('Importaciones Celulares y Más'),
    ('Distribuidora Global Tech')
```

Ingresamos los productos

```sql
INSERT INTO Productos (nombre, precio, proveedor_id, subcategoria_id) VALUES ('iPhone 14 Pro', 5600000.00, 1, 1),('Samsung Galaxy S22', 4200000.00, 1, 1),
   ('Xiaomi Redmi Note 11', 980000.00, 1, 1),
    ('Motorola Edge 30', 1500000.00, 1, 1),
    ('Huawei Nova 11', 2200000.00, 1, 1);
    ('LG Smart TV 55"', 499000.00, 2, 2),          
    ('Sony Bravia 50"', 600000.00, 2, 2),           
    ('Set de cuchillos inox', 39999.00, 4, 3),  
    ('Accesorios para auto - Kit limpieza', 45000.00, 4, 4)
```

Ingresamos los pedidos

```sql
INSERT INTO Pedidos (cliente_id, fecha, total) VALUES
(1, '2025-06-01', 1049000.00),  
(2, '2025-06-03', 600000.00),   
(3, '2025-06-05', 84999.00),    
(4, '2025-06-07', 39999.00),   
(5, '2025-06-09', 45000.00),    
(2, '2025-06-10', 1140000.00),  
(1, '2025-06-12', 499000.00),   
(3, '2025-06-13', 159000.00),  
(5, '2025-06-15', 600000.00),   
(4, '2025-06-17', 165000.00);   

```



#### **Joins**

1. ##### Obtener la lista de todos los pedidos con los nombres de clientes usando INNER JOIN .

```sql
SELECT 
    Pedidos.id AS pedido_id,
    Clientes.nombre AS cliente,
    Pedidos.fecha,
    Pedidos.total
FROM 
    Pedidos
INNER JOIN 
    Clientes ON Pedidos.cliente_id = Clientes.id;

```

![image-20250624145829184](/home/camper/.config/Typora/typora-user-images/image-20250624145829184.png)



2. Listar los productos y proveedores que los suministran con INNER JOIN

```sql
SELECT 
    Productos.nombre AS producto,
    Productos.precio,
    Proveedores.nombre AS proveedor
FROM 
    Productos
INNER JOIN 
    Proveedores ON Productos.proveedor_id = Proveedores.id;

```

![Captura desde 2025-06-24 15-05-23](/home/camper/Imágenes/Capturas de pantalla/Captura desde 2025-06-24 15-05-23.png)

3. Mostrar los pedidos y las ubicaciones de los clientes con LEFT JOIN .

```sql
SELECT
    p.id AS pedido_id,
    c.nombre AS cliente,
    p.fecha,
    p.total,
    uc.direccion,
    ci.nombre AS ciudad,
    es.nombre AS estado,
    pa.nombre AS pais,
    uc.codigo_postal
FROM Pedidos p
LEFT JOIN Clientes c ON p.cliente_id = c.id
LEFT JOIN UbicacionCliente uc ON uc.cliente_id = c.id
LEFT JOIN Ciudades ci ON uc.ciudad_id = ci.id
LEFT JOIN Estados es ON ci.estado_id = es.id
LEFT JOIN Paises pa ON es.pais_id = pa.id;

```



![image-20250624152520979](/home/camper/.config/Typora/typora-user-images/image-20250624152520979.png)



4. Consultar los empleados que han registrado pedidos, incluyendo empleados sin pedidos
   ( LEFT JOIN ).

```sql
SELECT
    e.id AS empleado_id,
    e.nombre,
    COUNT(p.id) AS total_pedidos
FROM Empleados e
LEFT JOIN Pedidos p ON p.empleado_id = e.id
GROUP BY e.id, e.nombre
ORDER BY total_pedidos DESC;

```





![image-20250624153339238](/home/camper/.config/Typora/typora-user-images/image-20250624153339238.png)



5. Obtener el tipo de producto y los productos asociados con INNER JOIN .

```sql
SELECT 
    sc.nombre_subcategoria AS tipo_producto,
    p.nombre AS producto,
    p.precio
FROM Productos p
INNER JOIN Subcategorias sc ON p.subcategoria_id = sc.id
ORDER BY sc.nombre_subcategoria, p.nombre;

```

![image-20250624153528538](/home/camper/.config/Typora/typora-user-images/image-20250624153528538.png)



6. Listar todos los clientes y el número de pedidos realizados con COUNT y GROUP BY

```sql
SELECT 
    c.id AS cliente_id,
    c.nombre AS cliente,
    COUNT(p.id) AS total_pedidos
FROM Clientes c
LEFT JOIN Pedidos p ON p.cliente_id = c.id
GROUP BY c.id, c.nombre
ORDER BY total_pedidos DESC;

```

![image-20250624153631793](/home/camper/.config/Typora/typora-user-images/image-20250624153631793.png)



7.  Combinar Pedidos y Empleados para mostrar qué empleados gestionaron pedidos específicos

```sql
SELECT
    p.id AS pedido_id,
    p.fecha,
    p.total,
    e.id AS empleado_id,
    e.nombre AS empleado
FROM Pedidos p
INNER JOIN Empleados e ON p.empleado_id = e.id
ORDER BY p.fecha DESC;

```

![image-20250624153750264](/home/camper/.config/Typora/typora-user-images/image-20250624153750264.png)

8. Mostrar productos que no han sido pedidos (usando RIGHT JOIN)

```sql
SELECT
    p.id AS pedido_id,
    pr.id AS producto_id,
    pr.nombre AS producto
FROM Pedidos p
RIGHT JOIN DetallesPedido dp ON dp.pedido_id = p.id
RIGHT JOIN Productos pr ON dp.producto_id = pr.id
WHERE p.id IS NULL
ORDER BY pr.nombre;

```

![image-20250624153822809](/home/camper/.config/Typora/typora-user-images/image-20250624153822809.png)

9.Mostrar el total de pedidos y ubicación de clientes usando múltiples JOIN



```sql
SELECT
    c.id AS cliente_id,
    c.nombre AS cliente,
    COUNT(p.id) AS total_pedidos,
    uc.direccion,
    ci.nombre AS ciudad,
    uc.codigo_postal
FROM Clientes c
LEFT JOIN Pedidos p ON p.cliente_id = c.id
LEFT JOIN UbicacionCliente uc ON uc.cliente_id = c.id
LEFT JOIN Ciudades ci ON uc.ciudad_id = ci.id
GROUP BY c.id, c.nombre, uc.direccion, ciudad, uc.codigo_postal
ORDER BY total_pedidos DESC;

```

![image-20250624154531545](/home/camper/.config/Typora/typora-user-images/image-20250624154531545.png)

10. Unir Proveedores , Productos , y TiposProductos para un listado completo de inventario.

```sql
SELECT
    p.id AS producto_id,
    p.nombre AS producto,
    p.precio,
    sc.nombre_subcategoria AS subcategoria,
    pr.nombre AS proveedor
FROM Productos p
LEFT JOIN Subcategorias sc ON p.subcategoria_id = sc.id
LEFT JOIN Proveedores pr ON p.proveedor_id = pr.id
ORDER BY p.nombre;

```



![image-20250624154824043](/home/camper/.config/Typora/typora-user-images/image-20250624154824043.png)



#### Consultas Simples:

1. **Seleccionar todos los productos con precio mayor a $50**

```sql
SELECT * FROM Productos
WHERE precio > 50;

```

![image-20250624154945492](/home/camper/.config/Typora/typora-user-images/image-20250624154945492.png)

2. Consultar clientes registrados en una ciudad específica

```sql
SELECT c.*
FROM Clientes c
JOIN UbicacionCliente uc ON c.id = uc.cliente_id
JOIN Ciudades ci ON uc.ciudad_id = ci.id
WHERE ci.nombre = 'Bogotá';

```

![image-20250624155101305](/home/camper/.config/Typora/typora-user-images/image-20250624155101305.png)

3. Mostrar empleados contratados en los últimos 5 años

```sql
SELECT * FROM Empleados
WHERE fecha_contratacion >= DATE_SUB(CURDATE(), INTERVAL 5 YEAR);

```

![image-20250624155240220](/home/camper/.config/Typora/typora-user-images/image-20250624155240220.png)



4. Seleccionar proveedores que suministran más de 5 productos



```sql
SELECT pr.id, pr.nombre, COUNT(p.id) AS total_productos
FROM Proveedores pr
JOIN Productos p ON p.proveedor_id = pr.id
GROUP BY pr.id, pr.nombre
HAVING COUNT(p.id) > 5;

```

![image-20250624155349747](/home/camper/.config/Typora/typora-user-images/image-20250624155349747.png)

6. Calcular el total de ventas por cada cliente

```sql
SELECT
    c.id AS cliente_id,
    c.nombre,
    SUM(p.total) AS total_ventas
FROM Clientes c
JOIN Pedidos p ON p.cliente_id = c.id
GROUP BY c.id, c.nombre;

```

![image-20250624155643177](/home/camper/.config/Typora/typora-user-images/image-20250624155643177.png)

7. Mostrar el salario promedio de los empleados

```sql
SELECT AVG(salario) AS salario_promedio
FROM Empleados;

```

​	

![image-20250624155742001](/home/camper/.config/Typora/typora-user-images/image-20250624155742001.png)

8.  Consultar el tipo de productos disponibles

```sql
SELECT DISTINCT nombre_subcategoria
FROM Subcategorias;

```

![image-20250624155833825](/home/camper/.config/Typora/typora-user-images/image-20250624155833825.png)

9. Seleccionar los 3 productos más caros

```sql
SELECT *
FROM Productos
ORDER BY precio DESC
LIMIT 3;

```

![image-20250624155920709](/home/camper/.config/Typora/typora-user-images/image-20250624155920709.png)

10.  Consultar el cliente con el mayor número de pedidos



```sql
SELECT
    c.id,
    c.nombre,
    COUNT(p.id) AS total_pedidos
FROM Clientes c
JOIN Pedidos p ON p.cliente_id = c.id
GROUP BY c.id, c.nombre
ORDER BY total_pedidos DESC
LIMIT 1;

```

![image-20250624160007384](/home/camper/.config/Typora/typora-user-images/image-20250624160007384.png)

#### Consultas Multitabla

1. Listar todos los pedidos y el cliente asociado.

```sql
SELECT p.id AS pedido_id, p.fecha, c.id AS cliente_id, c.nombre AS cliente
FROM Pedidos p
JOIN Clientes c ON p.cliente_id = c.id;

```

![image-20250624160613038](/home/camper/.config/Typora/typora-user-images/image-20250624160613038.png)

2. Mostrar la ubicación de cada cliente en sus pedidos

```sql
SELECT p.id AS pedido_id, c.nombre AS cliente, uc.direccion, ci.nombre AS ciudad
FROM Pedidos p
JOIN Clientes c ON p.cliente_id = c.id
LEFT JOIN UbicacionCliente uc ON c.id = uc.cliente_id
LEFT JOIN Ciudades ci ON uc.ciudad_id = ci.id;

```



![image-20250624160641551](/home/camper/.config/Typora/typora-user-images/image-20250624160641551.png)

3.  Listar productos junto con el proveedor y tipo de producto

```sql
SELECT
    p.id AS producto_id,
    p.nombre AS producto,
    pr.nombre AS proveedor,
    sc.nombre_subcategoria AS tipo_producto
FROM Productos p
LEFT JOIN Proveedores pr ON p.proveedor_id = pr.id
LEFT JOIN Subcategorias sc ON p.subcategoria_id = sc.id;

```

![image-20250624160728363](/home/camper/.config/Typora/typora-user-images/image-20250624160728363.png)

4. Consultar todos los empleados que gestionan pedidos de clientes en una ciudad específica

```sql
SELECT DISTINCT e.id, e.nombre
FROM Empleados e
JOIN Pedidos p ON p.empleado_id = e.id
JOIN Clientes c ON p.cliente_id = c.id
JOIN UbicacionCliente uc ON uc.cliente_id = c.id
JOIN Ciudades ci ON uc.ciudad_id = ci.id
WHERE ci.nombre = 'Bogotá';  -- Cambia por la ciudad que desees

```

![image-20250624160817078](/home/camper/.config/Typora/typora-user-images/image-20250624160817078.png)



6. Obtener la cantidad total de pedidos por cliente y ciudad

   ```sql
   SELECT c.id AS cliente_id, c.nombre, ci.nombre AS ciudad, COUNT(p.id) AS total_pedidos
   FROM Clientes c
   JOIN Pedidos p ON p.cliente_id = c.id
   JOIN UbicacionCliente uc ON uc.cliente_id = c.id
   JOIN Ciudades ci ON uc.ciudad_id = ci.id
   GROUP BY c.id, c.nombre, ci.nombre;
   
   ```

   

![image-20250624161004308](/home/camper/.config/Typora/typora-user-images/image-20250624161004308.png)

8. Mostrar el total de ventas agrupado por tipo de producto.

```sql
SELECT 
    sc.nombre_subcategoria AS tipo_producto,
    SUM(dp.cantidad * dp.precio_unidad) AS total_ventas
FROM DetallesPedido dp
JOIN Productos p ON dp.producto_id = p.id
JOIN Subcategorias sc ON p.subcategoria_id = sc.id
GROUP BY sc.nombre_subcategoria
ORDER BY total_ventas DESC;

```

![image-20250624161743074](/home/camper/.config/Typora/typora-user-images/image-20250624161743074.png)



9. **Listar empleados que gestionan pedidos de productos de un proveedor específico**!



```sql
SELECT DISTINCT 
    e.id AS empleado_id,
    e.nombre AS empleado
FROM Empleados e
JOIN Pedidos p ON p.empleado_id = e.id
JOIN DetallesPedido dp ON dp.pedido_id = p.id
JOIN Productos prd ON dp.producto_id = prd.id
WHERE prd.proveedor_id = 2;  

```



![image-20250624162237016](/home/camper/.config/Typora/typora-user-images/image-20250624162237016.png)

10.  **Obtener el ingreso total de cada proveedor a partir de los productos vendidos**



```sql
SELECT 
    pr.id AS proveedor_id,
    pr.nombre AS proveedor,
    SUM(dp.cantidad * dp.precio_unidad) AS ingreso_total
FROM Proveedores pr
JOIN Productos p ON p.proveedor_id = pr.id
JOIN DetallesPedido dp ON dp.producto_id = p.id
GROUP BY pr.id, pr.nombre
ORDER BY ingreso_total DESC;

```

![image-20250624162318518](/home/camper/.config/Typora/typora-user-images/image-20250624162318518.png)



#### Subconsultas



1.  **Producto más caro en cada categoría**

```sql
SELECT *
FROM Productos p
WHERE precio = (
    SELECT MAX(p2.precio)
    FROM Productos p2
    JOIN Subcategorias sc ON p2.subcategoria_id = sc.id
    WHERE sc.categoria_id = (
        SELECT sc2.categoria_id FROM Subcategorias sc2 WHERE sc2.id = p.subcategoria_id
    )
);

```

![image-20250624162519616](/home/camper/.config/Typora/typora-user-images/image-20250624162519616.png)



2. Cliente con mayor total en pedidos

   ```sql
   SELECT *
   FROM Clientes
   WHERE id = (
       SELECT cliente_id
       FROM Pedidos
       GROUP BY cliente_id
       ORDER BY SUM(total) DESC
       LIMIT 1
   );
   
   ```

   

![image-20250624162605257](/home/camper/.config/Typora/typora-user-images/image-20250624162605257.png)



3. Empleados que ganan más que el salario promedio

```sql
SELECT *
FROM Empleados
WHERE salario > (
    SELECT AVG(salario)
    FROM Empleados
);

```

![image-20250624162702342](/home/camper/.config/Typora/typora-user-images/image-20250624162702342.png)



5. Pedidos cuyo total es mayor al promedio de todos los pedidos

```sql
SELECT *
FROM Pedidos
WHERE total > (
    SELECT AVG(total)
    FROM Pedidos
);

```

![image-20250624162743423](/home/camper/.config/Typora/typora-user-images/image-20250624162743423.png)



6. 3 proveedores con más productos



```sql
SELECT proveedor_id, COUNT(*) AS total_productos
FROM Productos
GROUP BY proveedor_id
ORDER BY total_productos DESC
LIMIT 3;

```



![image-20250624162834383](/home/camper/.config/Typora/typora-user-images/image-20250624162834383.png)



7.  **Productos con precio superior al promedio de su tipo (subcategoría)**

```sql
SELECT *
FROM Productos p
WHERE precio > (
    SELECT AVG(p2.precio)
    FROM Productos p2
    WHERE p2.subcategoria_id = p.subcategoria_id
);

```

​	

![image-20250624162907418](/home/camper/.config/Typora/typora-user-images/image-20250624162907418.png)



8.  **Clientes que han realizado más pedidos que la media**

```sql
SELECT *
FROM Clientes
WHERE id IN (
    SELECT cliente_id
    FROM Pedidos
    GROUP BY cliente_id
    HAVING COUNT(*) >= (
        SELECT AVG(cnt)
        FROM (
            SELECT COUNT(*) AS cnt
            FROM Pedidos
            GROUP BY cliente_id
        ) AS sub
    )
);

```

![image-20250624163059008](/home/camper/.config/Typora/typora-user-images/image-20250624163059008.png)

9. Encontrar productos cuyo precio es mayor que el promedio de todos los productos



```sql
SELECT *
FROM Productos
WHERE precio > (
    SELECT AVG(precio)
    FROM Productos
);

```



![image-20250624163144010](/home/camper/.config/Typora/typora-user-images/image-20250624163144010.png)





10. Mostrar empleados cuyo salario es menor al promedio del departamento



```sql
SELECT *
FROM Empleados e
WHERE salario < (
    SELECT AVG(e2.salario)
    FROM Empleados e2
    WHERE e2.puesto = e.puesto
);

```

![image-20250624163302752](/home/camper/.config/Typora/typora-user-images/image-20250624163302752.png)

#### Procedimientos almacenados

1.Actualizar el precio de todos los productos de un proveedor

![image-20250624164259064](/home/camper/.config/Typora/typora-user-images/image-20250624164259064.png)



```sql
DELIMITER //

CREATE PROCEDURE actualizar_precios_proveedor(
    IN p_proveedor_id INT,
    IN p_porcentaje DECIMAL(5,2)
)
BEGIN
    UPDATE Productos
    SET precio = precio * (1 + p_porcentaje / 100)
    WHERE proveedor_id = p_proveedor_id;
END;
//

DELIMITER ;

```



2. Un procedimiento que devuelva la dirección de un cliente por ID.

![image-20250624164324899](/home/camper/.config/Typora/typora-user-images/image-20250624164324899.png)

```sql
DELIMITER //

CREATE PROCEDURE obtener_direccion_cliente(
    IN p_cliente_id INT
)
BEGIN
    SELECT uc.direccion, uc.codigo_postal
    FROM UbicacionCliente uc
    WHERE uc.cliente_id = p_cliente_id;
END;
//

DELIMITER ;

```



3. Registrar un pedido nuevo y sus detalles



![image-20250624164408571](/home/camper/.config/Typora/typora-user-images/image-20250624164408571.png)

```sql
DELIMITER //

CREATE PROCEDURE registrar_pedido(
    IN p_cliente_id INT,
    IN p_empleado_id INT,
    IN p_fecha DATE,
    IN p_total DECIMAL(10,2),
    IN p_producto_id INT,
    IN p_cantidad INT,
    IN p_precio DECIMAL(10,2)
)
BEGIN
    DECLARE v_pedido_id INT;

    INSERT INTO Pedidos (cliente_id, fecha, total, empleado_id)
    VALUES (p_cliente_id, p_fecha, p_total, p_empleado_id);

    SET v_pedido_id = LAST_INSERT_ID();

    INSERT INTO DetallesPedido (pedido_id, producto_id, cantidad, precio_unidad)
    VALUES (v_pedido_id, p_producto_id, p_cantidad, p_precio);
END;
//

DELIMITER ;

```



4. Calcular el total de ventas de un cliente



```sql
DELIMITER //

CREATE PROCEDURE total_ventas_cliente(
    IN p_cliente_id INT
)
BEGIN
    SELECT SUM(total) AS total_ventas
    FROM Pedidos
    WHERE cliente_id = p_cliente_id;
END;
//

DELIMITER ;

```



5.  **Obtener los empleados por puesto**

```sql
DELIMITER //

CREATE PROCEDURE empleados_por_puesto(
    IN p_puesto VARCHAR(50)
)
BEGIN
    SELECT * FROM Empleados
    WHERE puesto = p_puesto;
END;
//

DELIMITER ;

```



6.  **Actualizar el salario de empleados por puesto**

![image-20250624164546514](/home/camper/.config/Typora/typora-user-images/image-20250624164546514.png)

```sql
DELIMITER //

CREATE PROCEDURE actualizar_salario_por_puesto(
    IN p_puesto VARCHAR(50),
    IN p_incremento DECIMAL(5,2)
)
BEGIN
    UPDATE Empleados
    SET salario = salario * (1 + p_incremento / 100)
    WHERE puesto = p_puesto;
END;
//

DELIMITER ;

```



7.  **Listar pedidos entre dos fechas**

   ![image-20250624164449989](/home/camper/.config/Typora/typora-user-images/image-20250624164449989.png)

   ```sql
   DELIMITER //
   
   CREATE PROCEDURE pedidos_entre_fechas(
       IN p_inicio DATE,
       IN p_fin DATE
   )
   BEGIN
       SELECT * FROM Pedidos
       WHERE fecha BETWEEN p_inicio AND p_fin;
   END;
   //
   
   DELIMITER ;
   
   ```

   

8. Aplicar descuento a productos de una categoría

![image-20250624164656323](/home/camper/.config/Typora/typora-user-images/image-20250624164656323.png)

```sql
DELIMITER //

CREATE PROCEDURE aplicar_descuento_categoria(
    IN p_categoria_id INT,
    IN p_descuento DECIMAL(5,2)
)
BEGIN
    UPDATE Productos p
    JOIN Subcategorias sc ON p.subcategoria_id = sc.id
    SET p.precio = p.precio * (1 - p_descuento / 100)
    WHERE sc.categoria_id = p_categoria_id;
END;
//

DELIMITER ;

```



9. Listar todos los proveedores de un tipo de producto (subcategoría)

![image-20250624164640482](/home/camper/.config/Typora/typora-user-images/image-20250624164640482.png)



```sql
DELIMITER //

CREATE PROCEDURE proveedores_por_tipo(
    IN p_subcategoria_id INT
)
BEGIN
    SELECT DISTINCT pr.id, pr.nombre
    FROM Proveedores pr
    JOIN Productos p ON p.proveedor_id = pr.id
    WHERE p.subcategoria_id = p_subcategoria_id;
END;
//

DELIMITER ;

```

10. Devolver el pedido de mayor valor

![image-20250624164618826](/home/camper/.config/Typora/typora-user-images/image-20250624164618826.png)

```sql
DELIMITER //

CREATE PROCEDURE pedido_mayor_valor()
BEGIN
    SELECT *
    FROM Pedidos
    ORDER BY total DESC
    LIMIT 1;
END;
//

DELIMITER ;

```

#### Funciones Definidas por el Usuario

1. Crear una función que reciba una fecha y devuelva los días transcurridos.

   ```sql
   DELIMITER //
   
   CREATE FUNCTION dias_transcurridos(fecha DATE)
   RETURNS INT
   DETERMINISTIC
   BEGIN
       RETURN DATEDIFF(CURDATE(), fecha);
   END;
   //
   
   DELIMITER ;
   
   ```

   ![image-20250624164827421](/home/camper/.config/Typora/typora-user-images/image-20250624164827421.png)

2. Crear una función para calcular el total con impuesto de un monto.

```sql
DELIMITER //

CREATE FUNCTION total_con_impuesto(monto DECIMAL(10,2), porcentaje DECIMAL(5,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN monto * (1 + porcentaje / 100);
END;
//

DELIMITER ;

```

![image-20250624164909458](/home/camper/.config/Typora/typora-user-images/image-20250624164909458.png)

3. Una función que devuelva el total de pedidos de un cliente específico.

   ```sql
   DELIMITER //
   
   CREATE FUNCTION total_pedidos_cliente(cliente_id INT)
   RETURNS INT
   DETERMINISTIC
   BEGIN
       DECLARE total INT;
       SELECT COUNT(*) INTO total
       FROM Pedidos
       WHERE cliente_id = cliente_id;
       RETURN total;
   END;
   //
   
   DELIMITER ;
   
   ```

   

   ![image-20250624165005665](/home/camper/.config/Typora/typora-user-images/image-20250624165005665.png)

4. Crear una función para aplicar un descuento a un producto. 

​	

```sql
DELIMITER //

CREATE FUNCTION aplicar_descuento(precio DECIMAL(10,2), porcentaje DECIMAL(5,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN precio * (1 - porcentaje / 100);
END;
//

DELIMITER ;

```

​	

![image-20250624165053381](/home/camper/.config/Typora/typora-user-images/image-20250624165053381.png)

5. Una función que indique si un cliente tiene dirección registrada.

```sql
DELIMITER //

CREATE FUNCTION cliente_con_direccion(p_cliente_id INT)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE tiene_direccion BOOLEAN;
    SELECT COUNT(*) > 0 INTO tiene_direccion
    FROM UbicacionCliente
    WHERE cliente_id = p_cliente_id;
    RETURN tiene_direccion;
END;
//

DELIMITER ;

```

![image-20250624165156261](/home/camper/.config/Typora/typora-user-images/image-20250624165156261.png)

6. Crear una función que devuelva el salario anual de un empleado.

```sql
DELIMITER //

CREATE FUNCTION salario_anual(salario_mensual DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN salario_mensual * 12;
END;
//

DELIMITER ;

```

![image-20250624165239119](/home/camper/.config/Typora/typora-user-images/image-20250624165239119.png)

7. Una función para calcular el total de ventas de un tipo de producto.

```sql
DELIMITER //

CREATE FUNCTION total_ventas_subcategoria(p_subcat_id INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);
    SELECT SUM(dp.cantidad * dp.precio_unidad) INTO total
    FROM DetallesPedido dp
    JOIN Productos p ON dp.producto_id = p.id
    WHERE p.subcategoria_id = p_subcat_id;
    RETURN IFNULL(total, 0.00);
END;
//

DELIMITER ;

```



![image-20250624165331804](/home/camper/.config/Typora/typora-user-images/image-20250624165331804.png)

8. Crear una función para devolver el nombre de un cliente por ID.

```sql
DELIMITER //

CREATE FUNCTION nombre_cliente(p_id INT)
RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
    DECLARE nombre VARCHAR(100);
    SELECT c.nombre INTO nombre
    FROM Clientes c
    WHERE c.id = p_id;
    RETURN nombre;
END;
//

DELIMITER ;

```

![image-20250624165411893](/home/camper/.config/Typora/typora-user-images/image-20250624165411893.png)



9. Una función que reciba el ID de un pedido y devuelva su total.

```sql
DELIMITER //

CREATE FUNCTION total_pedido(p_pedido_id INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);
    SELECT SUM(dp.cantidad * dp.precio_unidad) INTO total
    FROM DetallesPedido dp
    WHERE dp.pedido_id = p_pedido_id;
    RETURN IFNULL(total, 0.00);
END;
//

DELIMITER ;

```

![image-20250624165448613](/home/camper/.config/Typora/typora-user-images/image-20250624165448613.png)

10. Crear una función que indique si un producto está en inventario.

```sql
DELIMITER //

CREATE FUNCTION producto_en_stock(p_id INT)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE en_stock BOOLEAN;
    SELECT CASE WHEN p.id IS NOT NULL THEN TRUE ELSE FALSE END INTO en_stock
    FROM Productos p
    WHERE p.id = p_id;
    RETURN en_stock;
END;
//

DELIMITER ;

```

![image-20250624165523958](/home/camper/.config/Typora/typora-user-images/image-20250624165523958.png)



#### Triggers

1. Crear un trigger que registre en HistorialSalarios cada cambio de salario de empleados. 		

   ```sql
   CREATE TABLE IF NOT EXISTS HistorialSalarios (
       id INT AUTO_INCREMENT PRIMARY KEY,
       empleado_id INT,
       salario_anterior DECIMAL(10,2),
       salario_nuevo DECIMAL(10,2),
       fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   ```

   ```sql
   DELIMITER //
   
   CREATE TRIGGER tr_salario_update
   BEFORE UPDATE ON Empleados
   FOR EACH ROW
   BEGIN
       IF OLD.salario <> NEW.salario THEN
           INSERT INTO HistorialSalarios (empleado_id, salario_anterior, salario_nuevo)
           VALUES (OLD.id, OLD.salario, NEW.salario);
       END IF;
   END;
   //
   
   DELIMITER ;
   
   ```

   

2. Crear un trigger que evite borrar productos con pedidos activos.    

   ```sql
   DELIMITER //
   
   CREATE TRIGGER tr_prevent_delete_producto
   BEFORE DELETE ON Productos
   FOR EACH ROW
   BEGIN
       IF EXISTS (
           SELECT 1 FROM DetallesPedido WHERE producto_id = OLD.id
       ) THEN
           SIGNAL SQLSTATE '45000'
           SET MESSAGE_TEXT = 'No se puede eliminar un producto con pedidos activos';
       END IF;
   END;
   //
   
   DELIMITER ;
   
   ```

   ​                                                             
3. Un trigger que registre en HistorialPedidos cada actualización en Pedidos .

   ```sql
   CREATE TABLE IF NOT EXISTS HistorialPedidos (
       id INT AUTO_INCREMENT PRIMARY KEY,
       pedido_id INT,
       fecha_modificacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       total_anterior DECIMAL(10,2),
       total_nuevo DECIMAL(10,2)
   );
   
   ```

   
4. Crear un trigger que actualice el inventario al registrar un pedido. 

   ```sql
   DELIMITER //
   
   CREATE TRIGGER tr_update_stock
   AFTER INSERT ON DetallesPedido
   FOR EACH ROW
   BEGIN
       UPDATE Productos
       SET stock = stock - NEW.cantidad
       WHERE id = NEW.producto_id;
   END;
   //
   
   DELIMITER ;
   
   ```

   
5. Un trigger que evite actualizaciones de precio a menos de $1. 

   ```sql
   DELIMITER //
   
   CREATE TRIGGER tr_precio_minimo
   BEFORE UPDATE ON Productos
   FOR EACH ROW
   BEGIN
       IF NEW.precio < 1 THEN
           SIGNAL SQLSTATE '45000'
           SET MESSAGE_TEXT = 'El precio no puede ser menor a $1';
       END IF;
   END;
   //
   
   DELIMITER ;
   
   ```

   
6. Crear un trigger que registre la fecha de creación de un pedido en HistorialPedidos . 

   ```sql
   DELIMITER //
   
   CREATE TRIGGER tr_nuevo_pedido
   AFTER INSERT ON Pedidos
   FOR EACH ROW
   BEGIN
       INSERT INTO HistorialPedidos (pedido_id, total_anterior, total_nuevo)
       VALUES (NEW.id, 0, NEW.total);
   END;
   //
   
   DELIMITER ;
   
   ```

   
7. Un trigger que mantenga el precio total de cada pedido en Pedidos . 

   ```sql
   DELIMITER //
   
   CREATE TRIGGER tr_total_pedido
   AFTER INSERT ON DetallesPedido
   FOR EACH ROW
   BEGIN
       UPDATE Pedidos
       SET total = (
           SELECT SUM(cantidad * precio_unidad)
           FROM DetallesPedido
           WHERE pedido_id = NEW.pedido_id
       )
       WHERE id = NEW.pedido_id;
   END;
   //
   
   DELIMITER ;
   
   ```

   
8. Crear un trigger para validar que UbicacionCliente no esté vacío al crear un cliente. 

   ```
   
   ```

   
9. Un trigger que registre en LogActividades cada modificación en Proveedores . 

   ```sql
   DELIMITER //
   
   CREATE TRIGGER tr_log_update_proveedores
   AFTER UPDATE ON Proveedores
   FOR EACH ROW
   BEGIN
       INSERT INTO LogActividades (entidad, entidad_id, accion, usuario)
       VALUES ('Proveedor', OLD.id, 'Actualización', USER());
   END;
   //
   
   DELIMITER ;
   
   ```

   
10. Crear un trigger que registre en HistorialContratos cada cambio en Empleados . 

```sql
CREATE TABLE IF NOT EXISTS HistorialContratos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    empleado_id INT,
    puesto_anterior VARCHAR(50),
    puesto_nuevo VARCHAR(50),
    salario_anterior DECIMAL(10,2),
    salario_nuevo DECIMAL(10,2),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```



```sql
DELIMITER //

CREATE TRIGGER tr_historial_contratos
BEFORE UPDATE ON Empleados
FOR EACH ROW
BEGIN
    IF OLD.puesto <> NEW.puesto OR OLD.salario <> NEW.salario THEN
        INSERT INTO HistorialContratos (
            empleado_id, puesto_anterior, puesto_nuevo, 
            salario_anterior, salario_nuevo
        )
        VALUES (
            OLD.id, OLD.puesto, NEW.puesto,
            OLD.salario, NEW.salario
        );
    END IF;
END;
//

DELIMITER ;

```

#### Busquedas Avanzadas

1. Función de Descuento por Categoría de Producto

```sql
DELIMITER //

CREATE FUNCTION CalcularDescuento(subcat_id INT, precio DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE nombre_subcat VARCHAR(100);
    DECLARE precio_descuento DECIMAL(10,2);

    -- Obtener el nombre de la subcategoría
    SELECT nombre_subcategoria INTO nombre_subcat
    FROM Subcategorias
    WHERE id = subcat_id;

    -- Aplicar descuento si es Electrónica
    IF nombre_subcat = 'Electrónica' THEN
        SET precio_descuento = precio * 0.9;
    ELSE
        SET precio_descuento = precio;
    END IF;

    RETURN precio_descuento;
END;
//

DELIMITER ;

```

2. Función para Obtener la Edad de un Cliente y Filtrar Clientes Mayores de
   Edad



```sql
DELIMITER //

CREATE FUNCTION CalcularEdad(fecha_nac DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, fecha_nac, CURDATE());
END;
//

DELIMITER ;

```

3. Función de Cálculo de Impuesto y Consulta de Productos con Precio Final

```sql
DELIMITER //

CREATE FUNCTION CalcularImpuesto(precio DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN precio * 1.15;
END;
//

DELIMITER ;

```

4. Función para Calcular el Total de Pedidos de un Cliente

```sql
DELIMITER //

CREATE FUNCTION TotalPedidosCliente(clienteId INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);

    SELECT SUM(total) INTO total
    FROM Pedidos
    WHERE cliente_id = clienteId;

    RETURN IFNULL(total, 0);
END;
//

DELIMITER ;

```

5. Función para Calcular el Salario Anual de un Empleado

```sql
DELIMITER //

CREATE FUNCTION SalarioAnual(salario_mensual DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN salario_mensual * 12;
END;
//

DELIMITER ;

```

6. Función de Bonificación y Consulta de Salarios Ajustados



```sql
DELIMITER //

CREATE FUNCTION Bonificacion(salario DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN salario * 0.10;
END;
//

DELIMITER ;

```

7. Función para Calcular Días Transcurridos Desde el Último Pedido

```sql
DELIMITER //

CREATE FUNCTION DiasDesdeUltimoPedido(clienteId INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE dias INT;
    
    SELECT DATEDIFF(CURDATE(), MAX(fecha)) INTO dias
    FROM Pedidos
    WHERE cliente_id = clienteId;

    RETURN IFNULL(dias, 9999);  -- Si no tiene pedidos, devuelve un valor grande
END;
//

DELIMITER ;

```



8. Función para Calcular el Total en Inventario de un Producto



```sql
DELIMITER //

CREATE FUNCTION TotalInventarioProducto(precio DECIMAL(10,2), cantidad INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN precio * cantidad;
END;
//

DELIMITER ;

```

9. Creación de un Historial de Precios de Productos



```sql
DELIMITER //

CREATE TRIGGER RegistroCambioPrecio
BEFORE UPDATE ON Productos
FOR EACH ROW
BEGIN
    IF OLD.precio <> NEW.precio THEN
        INSERT INTO HistorialPrecios (producto_id, precio_anterior, precio_nuevo)
        VALUES (OLD.id, OLD.precio, NEW.precio);
    END IF;
END;
//

DELIMITER ;

```

10. Procedimiento para Generar Reporte de Ventas Mensuales por Empleado

```sql
DELIMITER //

CREATE PROCEDURE ReporteVentasMensuales(IN mes INT, IN anio INT)
BEGIN
    SELECT 
        e.id AS empleado_id,
        e.nombre AS empleado,
        SUM(p.total) AS total_ventas
    FROM Empleados e
    JOIN Pedidos p ON p.empleado_id = e.id
    WHERE MONTH(p.fecha) = mes AND YEAR(p.fecha) = anio
    GROUP BY e.id, e.nombre
    ORDER BY total_ventas DESC;
END;
//

DELIMITER ;

```

11. Subconsulta para Obtener el Producto Más Vendido por Cada Proveedor

```sql
SELECT 
    pr.nombre AS proveedor,
    p.nombre AS producto,
    SUM(dp.cantidad) AS cantidad_vendida
FROM Productos p
JOIN Proveedores pr ON p.proveedor_id = pr.id
JOIN DetallesPedido dp ON dp.producto_id = p.id
GROUP BY pr.id, p.id
HAVING SUM(dp.cantidad) = (
    SELECT MAX(SUM_INNER)
    FROM (
        SELECT SUM(dp2.cantidad) AS SUM_INNER
        FROM Productos p2
        JOIN DetallesPedido dp2 ON dp2.producto_id = p2.id
        WHERE p2.proveedor_id = pr.id
        GROUP BY p2.id
    ) AS subquery
);

```

12. Función para Calcular el Estado de Stock de un Producto



```sql
DELIMITER //

CREATE FUNCTION EstadoStock(cantidad INT)
RETURNS VARCHAR(10)
DETERMINISTIC
BEGIN
    DECLARE estado VARCHAR(10);

    IF cantidad >= 100 THEN
        SET estado = 'Alto';
    ELSEIF cantidad >= 50 THEN
        SET estado = 'Medio';
    ELSE
        SET estado = 'Bajo';
    END IF;

    RETURN estado;
END;
//

DELIMITER ;

```
13. Trigger para Control de Inventario en Pedidos



```sql
DELIMITER //

CREATE TRIGGER ActualizarInventario
BEFORE INSERT ON DetallesPedido
FOR EACH ROW
BEGIN
    DECLARE stock_actual INT;

    -- Obtener el stock disponible del producto
    SELECT cantidad INTO stock_actual
    FROM Productos
    WHERE id = NEW.producto_id;

    -- Verificar si hay suficiente stock
    IF stock_actual < NEW.cantidad THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Stock insuficiente para completar el pedido';
    ELSE
        -- Disminuir el stock
        UPDATE Productos
        SET cantidad = cantidad - NEW.cantidad
        WHERE id = NEW.producto_id;
    END IF;
END;
//

DELIMITER ;


```



14. Procedimiento para Generar Informe de Clientes Inactivos

```sql

DELIMITER //

CREATE PROCEDURE ClientesInactivos()
BEGIN
    SELECT c.id AS cliente_id, c.nombre AS cliente
    FROM Clientes c
    WHERE c.id NOT IN (
        SELECT DISTINCT cliente_id
        FROM Pedidos
        WHERE fecha >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
    );
END;
//

DELIMITER ;

```



















