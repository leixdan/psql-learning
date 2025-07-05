# PostgreSQL y bases de datos 🐘️
## *¿Qué son las bases de datos?*
Las bases de datos son sistemas organizados que almacenarán, gestionarán editando o actualizando datos, recuperarlos (backup) y más de forma muy eficiente. Son la herramienta más poderosa en entornos digitales para poder organizar de forma estructurada grandes cantidades de información.

- Permiten acceso rápido y eficiente.
- Aseguran la integridad y seguridad de la información.
- Soportan múltiples usuarios y servir a distintas aplicaciones (multilenguaje).
- Son la base de toda aplicación moderna (multiplataforma).

Sus componentes fundamentales: 🍲️

| Componente          | Función                                                     |
| ------------------- | ----------------------------------------------------------- |
| **Tabla**           | Estructura básica de almacenamiento. Como una hoja de Excel |
| **Fila (registro)** | Una entrada o instancia concreta de datos                   |
| **Columna (campo)** | Un atributo específico de los datos (nombre, edad, etc.)    |
| **Clave primaria**  | Identifica de forma única cada fila                         |
| **Clave foránea**   | Relaciona una fila con otra tabla (relaciones entre datos)  |
| **Índice**          | Acelera la búsqueda y orden de datos                        |

Principios clave de las bases de datos:

1. Integridad de datos: evita errores, duplicaciones o incoherencias.

2. Atomicidad (ACID): garantiza que las transacciones se completen totalmente o no se hagan.

3. Seguridad: control de accesos, cifrado, autenticación.

4. Escalabilidad: puede crecer en tamaño y usuarios sin fallar.


## ¿Por qué PostgreSQL? 🤷‍♂️️

PostgreSQL es un sistema de gestión de bases de datos relacional (RDBMS) open source, altamente confiable, extensible y conforme con el estándar SQL. Se le considera uno de los motores más avanzados del mundo open source.

De hecho, mi elección de Go como lenguaje para backend fue en función de la robustez y capacidades, popularidad en crecimiento y versatilidad, para mí pSQL resolvía muy bien mi necesidad de una base de datos a diferencia del uso en Excel que estaba explorando con Python.

## Primeros pasos en Postgre 🦶️

Ya instalado, a mi me resultó mucho más cómodo iniciar mis tablas y bases de datos desde la terminal, yo uso fedora / linux. Dejaré aquí los comandos y pasos base a modo de repaso:

`sudo -i -u postgres` - Acceso a Postgre como superusuario.

`psql` - Abre la consola `psql`.

`CREATE DATABASE mibasedatos;` - Crea una base de datos de nombre "mibasededatos".

`\c mibasedatos` - Conectamos con la base de datos, como abrir la carpeta.

`DROP DATABASE nombre_basedatos;` - Borrar base de datos (si fuera necesario).

Aquí es importante resaltar que una base de datos no es directamente lo que guarda los datos, para eso hay que crear las tablas:

```psql

CREATE TABLE empleados (

    id SERIAL PRIMARY KEY,

    nombre VARCHAR(100),

    salario NUMERIC(10, 2),

    fecha_ingreso DATE DEFAULT CURRENT_DATE

);
```

Como vemos, tenemos una base de datos `mibasededatos` que contiene una tabla `empleados`, ahora sólo quedará salir de la consola para poder conectarnos desde `Go`, lo haremos en otra ocasión.

`DROP TABLE IF EXISTS nombre_tabla;` - Borrar tabla si fuera necesario.

`DELETE FROM empleados WHERE nombre = 'Ana';` - Eliminar filas con condición de un elemento "nombre".

`TRUNCATE TABLE nombre_tabla;` - Borrar elementos en la tabla sin eliminar la tabla en sí.

`\q` - Salir de la terminal `psql`

Aquí te dejo un listado de comandos básicos en psql:

| Comando          | Descripción                                   |
| ---------------- | --------------------------------------------- |
| `\l`             | Lista todas las bases de datos                |
| `\c mibasedatos` | Conectar a la base de datos "mibasedatos"     |
| `\dt`            | Lista las tablas de la base de datos actual   |
| `\d empleados`   | Muestra la estructura de la tabla `empleados` |
| `\q`             | Salir de la consola `psql`                    |
| `\h`             | Ayuda para comandos SQL (ej: `\h CREATE`)     |
| `\?`             | Ayuda general de comandos `psql`              |



## Operaciones básicas en Postgre 🧮️

| Operación | Acción                  | SQL (psql)                                                         | Go con `pgx` (ejemplo básico)                                                                                                                                                                                                                                                                                                                        |
| --------- | ----------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **C**     | **Crear (INSERT)**      | `INSERT INTO empleados (nombre, salario) VALUES ('Ana', 3000.00);` | `go<br>_, err := conn.Exec(ctx, "INSERT INTO empleados (nombre, salario) VALUES ($1, $2)", "Ana", 3000.00)<br>if err != nil { /* manejar error */ }`                                                                                                                                                                                                 |
| **R**     | **Leer (SELECT)**       | `SELECT * FROM empleados WHERE salario > 2500;`                    | `go<br>rows, err := conn.Query(ctx, "SELECT id, nombre, salario FROM empleados WHERE salario > $1", 2500)<br>defer rows.Close()<br>for rows.Next() {<br>  var id int<br>  var nombre string<br>  var salario float64<br>  rows.Scan(&id, &nombre, &salario)<br>  // procesar datos<br>}<br>if err := rows.Err(); err != nil { /* manejar error */ }` |
| **U**     | **Actualizar (UPDATE)** | `UPDATE empleados SET salario = 3200.00 WHERE nombre = 'Ana';`     | `go<br>_, err := conn.Exec(ctx, "UPDATE empleados SET salario = $1 WHERE nombre = $2", 3200.00, "Ana")<br>if err != nil { /* manejar error */ }`                                                                                                                                                                                                     |
| **D**     | **Eliminar (DELETE)**   | `DELETE FROM empleados WHERE nombre = 'Ana';`                      | `go<br>_, err := conn.Exec(ctx, "DELETE FROM empleados WHERE nombre = $1", "Ana")<br>if err != nil { /* manejar error */ }`                                                                                                                                                                                                                          |

## Tipos de datos en Postgre 🗃️

| Categoría        | Ejemplos                                                                          | Descripción                                             |
| ---------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------- |
| **Números**      | `INTEGER`, `SMALLINT`, `BIGINT`, `DECIMAL`, `NUMERIC`, `REAL`, `DOUBLE PRECISION` | Valores numéricos exactos o aproximados                 |
| **Texto**        | `CHAR`, `VARCHAR`, `TEXT`                                                         | Cadenas de texto                                        |
| **Booleanos**    | `BOOLEAN`                                                                         | `TRUE`, `FALSE`, `NULL`                                 |
| **Fecha y hora** | `DATE`, `TIME`, `TIMESTAMP`, `INTERVAL`                                           | Manejo de tiempo y duración                             |
| **UUID**         | `UUID`                                                                            | Identificadores únicos universales                      |
| **JSON/JSONB**   | `JSON`, `JSONB`                                                                   | Almacenamiento de datos semiestructurados               |
| **Geográficos**  | `POINT`, `LINE`, `CIRCLE`, etc.                                                   | Datos espaciales básicos (más potentes con PostGIS)     |
| **Arreglos**     | `INTEGER[]`, `TEXT[]`                                                             | Permite almacenar múltiples valores en una sola columna |
| **Enumerados**   | `ENUM`                                                                            | Conjuntos definidos por el usuario                      |

## Brindar permisos a bases de datos y tablas a usuarios 🔐️
Sí, se pueden tener usuarios con acceso a las bases de datos y las tablas que puedan sólo ver, actualizar o un poco de todo, esto siempre pensando en la seguridad de los datos, con mucho cuidado.

```psql
-- 1. Crear usuario
CREATE USER juan WITH PASSWORD 's3guraP@ss';

-- 2. Crear base de datos
CREATE DATABASE empresa;

-- 3. Dar permisos completos sobre la base de datos a juan
GRANT ALL PRIVILEGES ON DATABASE empresa TO juan;

-- 4. Conectarse a la base empresa
\c empresa

-- 5. Crear tabla empleados
CREATE TABLE empleados (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    salario NUMERIC(10,2)
);

-- 6. Dar permisos sobre tabla empleados al usuario juan
GRANT SELECT, INSERT, UPDATE, DELETE ON empleados TO juan;


```

Si ya no quieres algún usuario, lo puedes eliminar fácilmente:

`DROP USER nombre_usuario;`

***

## Repaso 🧠️
* PostgreSQL combina muy bien con mi lenguaje de base `Go`, tiene grandes capacidades de manejar miles/millones de datos sin pestañear al igual que `Go`.
* La mayoría de solicitudes a la base de datos se harán desde funciones en mi código, pero conocer las bases en `psql` es importante igualmente.
* Tener bien guardados usuarios y contraseñas, también terminar conexiones si queremos eliminar los usuarios.
