# Práctica 4 — Base de datos de viveros

## Autores

- Sergio Ávila González

## Descripción

Diseño e implementación en PostgreSQL del modelo relacional de Tajinaste S.A. La base de datos permite gestionar los viveros y sus zonas, el stock de productos, el historial de puestos y tareas de los empleados, los clientes, los pedidos y el programa de fidelización Tajinaste Plus.

El proyecto incluye las claves primarias y foráneas, las restricciones de integridad, los datos de prueba y dos vistas para calcular atributos derivados sin almacenarlos de forma repetida.

## Archivos del repositorio

- `init.sql`: crea la base de datos, las tablas, restricciones, vistas y datos de prueba.
- `diagrama.drawio`: modelo entidad/relación editable en Draw.io.
- `diagrama.png`: imagen del modelo entidad/relación.
- `relacional.png`: imagen del modelo relacional.
- `README.md`: explicación y comprobaciones de la práctica.

## Diagramas

![Diagrama entidad-relación](diagrama.png)

![Modelo relacional](relacional.png)

## Tablas del modelo

| Tabla | Filas de prueba | Descripción |
|---|---:|---|
| `viveros` | 5 | Viveros de la empresa y su georreferenciación. |
| `zonas` | 5 | Zonas de los viveros, como Exterior, Almacén o Invernadero. |
| `empleados` | 5 | Datos personales de los empleados. |
| `puestos` | 5 | Puestos disponibles en cada vivero. |
| `asignaciones` | 7 | Historial de puestos ocupados por los empleados. |
| `tareas` | 7 | Periodos en los que una asignación trabaja en una zona. |
| `productos` | 5 | Catálogo de productos con su precio actual. |
| `stock` | 7 | Cantidad de cada producto disponible en cada zona. |
| `clientes` | 6 | Datos básicos de los clientes. |
| `clientes_correos` | 6 | Correos de contacto; permite varios por cliente. |
| `tajinaste_plus` | 5 | Periodos de pertenencia de los clientes al programa de fidelización. |
| `pedidos` | 6 | Pedidos realizados, junto con el cliente y el empleado responsable. |
| `pedidos_contenido` | 9 | Productos, cantidades y precios unitarios de cada pedido. |
| `bonificaciones` | 5 | Descuentos concedidos a clientes Plus para productos concretos. |

Se han consultado las **14 tablas** del modelo y todas contienen los datos de prueba esperados. No falta ningún `SELECT` de tabla en la lista de comprobación.

## Relaciones principales

- Un vivero puede tener varias zonas y varios puestos.
- Un empleado puede ocupar distintos puestos a lo largo del tiempo. Cada periodo se guarda como una fila diferente en `asignaciones` mediante `inicio` y `fin`.
- Una asignación puede realizar tareas en zonas del vivero correspondiente.
- Un producto puede estar en varias zonas y una zona puede contener varios productos. La relación N:M se representa mediante `stock`.
- Un cliente puede tener varios correos.
- Un cliente puede pertenecer a Tajinaste Plus durante uno o varios periodos.
- Un cliente puede realizar varios pedidos y cada pedido tiene un único empleado responsable.
- Un pedido puede contener varios productos y un producto puede aparecer en varios pedidos. La relación N:M se representa mediante `pedidos_contenido`.
- Un cliente puede recibir varias bonificaciones y un producto puede aparecer en diferentes bonificaciones.

## Restricciones destacadas

- Las cantidades de `stock` no pueden ser negativas.
- La cantidad de un producto dentro de un pedido debe ser positiva.
- Los precios y descuentos no pueden ser negativos.
- En los intervalos cerrados, la fecha de fin debe ser posterior a la fecha de inicio.
- Un valor `NULL` en `fin` indica que la asignación o pertenencia a Tajinaste Plus continúa activa.
- Los periodos de puestos de un mismo empleado no pueden solaparse. La extensión `btree_gist` permite aplicar esta restricción en PostgreSQL.
- La combinación de pedido y producto identifica cada línea de `pedidos_contenido` y evita repetir un mismo producto dentro del mismo pedido.
- La combinación de zona y producto identifica cada fila de `stock`.
- `precio_unitario` conserva el precio aplicado cuando se realizó el pedido. De esta forma, los pedidos antiguos no cambian si se modifica el precio actual de `productos`.
- Al borrar un pedido, sus filas de `pedidos_contenido` se eliminan mediante `ON DELETE CASCADE`.

## Ejecución

El script debe ejecutarse desde `psql`, conectado a una base de datos diferente de `viveros` y con permisos para crear bases de datos y la extensión `btree_gist`:

```bash
psql -U postgres -d postgres -f init.sql
```

Después se puede entrar en la base de datos con:

```bash
psql -U postgres -d viveros
```

## Comprobación de las tablas

En este apartado se muestra el contenido de todas las tablas después de ejecutar el script y cargar los datos de prueba.

### Tabla `viveros`

```sql
SELECT * FROM viveros;
```

| id | longitud | latitud |
|---:|---:|---:|
| 1 | -16.315000 | 28.487000 |
| 2 | -16.250000 | 28.463000 |
| 3 | -16.410000 | 28.480000 |
| 4 | -16.520000 | 28.390000 |
| 5 | -16.680000 | 28.090000 |

### Tabla `zonas`

```sql
SELECT * FROM zonas;
```

| id | id_vivero | nombre | longitud | latitud |
|---:|---:|---|---:|---:|
| 1 | 1 | Exterior | -16.315100 | 28.487100 |
| 2 | 2 | Almacén | -16.250100 | 28.463100 |
| 3 | 3 | Invernadero | -16.410100 | 28.480100 |
| 4 | 4 | Exposición | -16.520100 | 28.390100 |
| 5 | 5 | Sombra | -16.680100 | 28.090100 |

### Tabla `empleados`

```sql
SELECT * FROM empleados;
```

| id | dni | nombre | apellido | nacimiento |
|---:|---|---|---|---|
| 1 | `12345678Z` | Ana | Hernández | 1988-04-12 |
| 2 | `87654321X` | Carlos | Pérez | 1990-09-23 |
| 3 | `23456789D` | Lucía | González | 1985-02-08 |
| 4 | `34567890A` | Mario | Díaz | 1995-11-17 |
| 5 | `45678901R` | Elena | Rodríguez | 1992-06-30 |

### Tabla `puestos`

```sql
SELECT * FROM puestos;
```

| id | id_vivero | nombre | descripción |
|---:|---:|---|---|
| 1 | 1 | Jardinería | Mantenimiento y cuidado de las plantas de exterior. |
| 2 | 2 | Almacén | Recepción, organización y reposición de productos. |
| 3 | 3 | Producción | Cuidado y control de plantas de invernadero. |
| 4 | 4 | Ventas | Atención a clientes y asesoramiento comercial. |
| 5 | 5 | Encargado | Coordinación de las operaciones del vivero. |

### Tabla `asignaciones`

```sql
SELECT * FROM asignaciones;
```

| id | id_puesto | id_empleado | inicio | fin |
|---:|---:|---:|---|---|
| 1 | 1 | 1 | 2023-01-01 00:00:00 | 2024-01-01 00:00:00 |
| 2 | 2 | 1 | 2024-01-01 00:00:00 | `NULL` |
| 3 | 3 | 2 | 2025-01-01 00:00:00 | `NULL` |
| 4 | 4 | 3 | 2023-02-01 00:00:00 | 2024-01-01 00:00:00 |
| 5 | 4 | 3 | 2024-01-01 00:00:00 | `NULL` |
| 6 | 5 | 4 | 2024-04-01 00:00:00 | `NULL` |
| 7 | 1 | 5 | 2025-09-01 00:00:00 | `NULL` |

El empleado 1 tiene una asignación finalizada y otra activa. Esto comprueba que se conserva el historial de puestos. El valor `NULL` indica que la asignación continúa vigente.

### Tabla `tareas`

```sql
SELECT * FROM tareas;
```

| id | id_asignacion | id_zona | inicio | fin |
|---:|---:|---:|---|---|
| 1 | 1 | 1 | 2023-06-01 08:00:00 | 2023-06-10 16:00:00 |
| 2 | 2 | 2 | 2026-10-01 09:00:00 | `NULL` |
| 3 | 3 | 3 | 2025-11-01 08:00:00 | 2025-11-02 16:00:00 |
| 4 | 4 | 4 | 2023-05-01 08:00:00 | 2023-05-03 16:00:00 |
| 5 | 5 | 4 | 2025-05-01 08:00:00 | 2025-05-02 16:00:00 |
| 6 | 6 | 5 | 2025-06-01 08:00:00 | 2025-06-02 16:00:00 |
| 7 | 7 | 1 | 2025-11-01 08:00:00 | 2025-11-03 16:00:00 |

### Tabla `productos`

```sql
SELECT * FROM productos;
```

| codigo_de_barras | precio | nombre |
|---|---:|---|
| `8410000000001` | 12.50 | Maceta de barro |
| `8410000000002` | 5.99 | Sustrato universal |
| `8410000000003` | 21.00 | Rosal |
| `8410000000004` | 35.50 | Olivo joven |
| `8410000000005` | 8.90 | Abono orgánico |

### Tabla `stock`

```sql
SELECT * FROM stock;
```

| id_zona | codigo_producto | cantidad |
|---:|---|---:|
| 1 | `8410000000001` | 25 |
| 1 | `8410000000002` | 0 |
| 2 | `8410000000003` | 18 |
| 3 | `8410000000004` | 7 |
| 4 | `8410000000001` | 12 |
| 5 | `8410000000005` | 31 |
| 5 | `8410000000002` | 4 |

La cantidad cero es válida: indica que el producto está asignado a la zona, pero actualmente se encuentra agotado.

### Tabla `clientes`

```sql
SELECT * FROM clientes;
```

| id | nombre | apellido |
|---:|---|---|
| 1 | Beatriz | Suárez |
| 2 | Daniel | Méndez |
| 3 | Irene | Padrón |
| 4 | Pablo | León |
| 5 | Sara | Cabrera |
| 6 | Cliente ocasional | `NULL` |

### Tabla `clientes_correos`

```sql
SELECT * FROM clientes_correos;
```

| id_cliente | correo |
|---:|---|
| 1 | beatriz@example.test |
| 1 | b.suarez@example.test |
| 2 | daniel@example.test |
| 3 | irene@example.test |
| 4 | pablo@example.test |
| 5 | sara@example.test |

El cliente 1 tiene dos correos, lo que comprueba que el modelo permite almacenar varios correos para un mismo cliente.

### Tabla `tajinaste_plus`

```sql
SELECT * FROM tajinaste_plus;
```

| id_cliente | inicio | fin |
|---:|---|---|
| 1 | 2024-01-01 00:00:00 | `NULL` |
| 2 | 2024-05-01 00:00:00 | 2026-01-01 00:00:00 |
| 3 | 2025-01-01 00:00:00 | `NULL` |
| 4 | 2025-03-01 00:00:00 | `NULL` |
| 5 | 2024-01-01 00:00:00 | 2026-04-01 00:00:00 |

El cliente ocasional, con ID 6, no aparece en esta tabla porque no pertenece al programa Tajinaste Plus.

### Tabla `pedidos`

```sql
SELECT * FROM pedidos;
```

| id | fecha | id_empleado | id_cliente |
|---:|---|---:|---:|
| 1 | 2025-11-15 10:30:00 | 1 | 1 |
| 2 | 2025-11-20 12:15:00 | 2 | 2 |
| 3 | 2026-02-10 09:45:00 | 3 | 3 |
| 4 | 2026-03-01 14:20:00 | 5 | 6 |
| 5 | 2026-06-01 11:00:00 | 4 | 5 |
| 6 | 2025-04-05 16:05:00 | 3 | 4 |

Cada pedido tiene un único cliente y un único empleado responsable.

### Tabla `pedidos_contenido`

```sql
SELECT * FROM pedidos_contenido;
```

| id_pedido | id_producto | cantidad | precio_unitario |
|---:|---|---:|---:|
| 1 | `8410000000001` | 2 | 12.50 |
| 1 | `8410000000002` | 1 | 5.99 |
| 2 | `8410000000003` | 3 | 21.00 |
| 3 | `8410000000004` | 1 | 35.50 |
| 3 | `8410000000001` | 1 | 12.50 |
| 4 | `8410000000002` | 2 | 5.75 |
| 5 | `8410000000005` | 1 | 8.90 |
| 6 | `8410000000003` | 1 | 20.95 |
| 6 | `8410000000002` | 2 | 5.99 |

El pedido 1 contiene dos productos distintos. `precio_unitario` conserva el precio aplicado en el momento de la compra, aunque posteriormente cambie el precio actual del producto.

### Tabla `bonificaciones`

```sql
SELECT * FROM bonificaciones;
```

| id | id_cliente | id_producto | mes_referencia | descuento | valido_desde | valido_hasta |
|---:|---:|---|---|---:|---|---|
| 1 | 1 | `8410000000001` | 2025-10-01 | 10.00 | 2025-11-01 00:00:00 | 2025-12-01 00:00:00 |
| 2 | 2 | `8410000000003` | 2025-10-01 | 5.00 | 2025-11-01 00:00:00 | 2025-12-01 00:00:00 |
| 3 | 3 | `8410000000004` | 2026-01-01 | 12.50 | 2026-02-01 00:00:00 | 2026-03-01 00:00:00 |
| 4 | 4 | `8410000000002` | 2025-03-01 | 8.00 | 2025-04-01 00:00:00 | 2025-05-01 00:00:00 |
| 5 | 5 | `8410000000005` | 2026-03-01 | 15.00 | 2026-03-01 00:00:00 | 2026-04-01 00:00:00 |

## Vistas de atributos derivados

### Precio total de los pedidos

El precio de un pedido no se almacena directamente. La vista `vista_pedidos_totales` lo calcula sumando la cantidad por el precio unitario de todas sus líneas:

```sql
SELECT *
FROM vista_pedidos_totales
ORDER BY id_pedido;
```

Con los datos de prueba, los totales esperados son:

| Pedido | Cálculo | Total |
|---:|---|---:|
| 1 | `2 × 12.50 + 1 × 5.99` | 30.99 |
| 2 | `3 × 21.00` | 63.00 |
| 3 | `1 × 35.50 + 1 × 12.50` | 48.00 |
| 4 | `2 × 5.75` | 11.50 |
| 5 | `1 × 8.90` | 8.90 |
| 6 | `1 × 20.95 + 2 × 5.99` | 32.93 |

### Volumen mensual de compras de Tajinaste Plus

La vista `vista_compras_mensuales_plus` agrupa por cliente y mes los pedidos realizados mientras el cliente pertenecía al programa:

```sql
SELECT *
FROM vista_compras_mensuales_plus
ORDER BY id_cliente, mes;
```

Con los datos incluidos, deben aparecer estas compras:

| Cliente | Mes | Volumen de compras |
|---:|---|---:|
| 1 | 2025-11 | 30.99 |
| 2 | 2025-11 | 63.00 |
| 3 | 2026-02 | 48.00 |
| 4 | 2025-04 | 32.93 |

El pedido 4 no se incluye porque pertenece al cliente ocasional. El pedido 5 tampoco se incluye porque el cliente 5 terminó su pertenencia a Tajinaste Plus antes de la fecha del pedido.

## Comprobación de `ON DELETE CASCADE`

La siguiente prueba elimina temporalmente el pedido 1. Sus filas de `pedidos_contenido` deben desaparecer automáticamente. Se utiliza una transacción con `ROLLBACK` para recuperar los datos al terminar la comprobación:

```sql
BEGIN;

DELETE FROM pedidos
WHERE id = 1;

SELECT *
FROM pedidos_contenido
WHERE id_pedido = 1;

ROLLBACK;
```

El `SELECT` intermedio debe devolver cero filas. Después del `ROLLBACK`, el pedido y sus productos vuelven a estar disponibles.

## Observaciones sobre los datos

- Una asignación con `fin = NULL` representa el puesto actual del empleado.
- Una pertenencia a Tajinaste Plus con `fin = NULL` continúa vigente.
- Las fechas de las tareas están comprendidas dentro del periodo de sus asignaciones.
- Las bonificaciones guardan el mes cuyas compras se utilizaron para calcularlas y el periodo durante el que pueden aplicarse.
