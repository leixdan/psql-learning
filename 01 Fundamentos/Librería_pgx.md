# Entendiendo PGX (Go + PostgreSQL) 📦

`pgx` es una librería nativa de `Go` para conectarse y trabajar con bases de datos PostgreSQL. Se considera una de las más rápidas y rica en funcionalidades a comparación del paquete estándar `database/sql`.

## Instalación ⬇️💻

Desde la terminal se deberá ingresar:

`go get github.com/jackc/pgx/v5
`

Antes de darle uso a la librería o importarla al código en `Go` se debe generar un módulo en la carpeta del proyecto, luego agregar el main.go y así podremos importar la librería.


Pensemos que tenemos una base de datos ya iniciada y queremos trabajar sobre una tabla llamada "empleados":

```sql

CREATE TABLE empleados (
    id SERIAL PRIMARY KEY,
    nombre TEXT NOT NULL,
    edad INT,
    salario NUMERIC
);
```

***

## Lo básico de PGX 👶

Conexión con Go y consultas simples:

Tomemos en cuenta que tiene que haberse creado la base de datos y tabla de forma previa, desde go y ya que hayamos importado la librería podremos hacer uso de elementos y funciones básicas de `pgx`:

| Función / Elemento     | Descripción                        | Ejemplo usando `empleados`                                          |
| ---------------------- | ---------------------------------- | ------------------------------------------------------------------- |
| `pgx.Connect`          | Conexión directa                   | `pgx.Connect(ctx, "postgres://user:pass@localhost/plantilla")`      |
| `QueryRow`             | Consulta que retorna una sola fila | `conn.QueryRow(ctx, "SELECT nombre FROM empleados WHERE id=$1", 1)` |
| `Scan`                 | Leer el resultado de una fila      | `.Scan(&nombre)`                                                    |
| `context.Background()` | Contexto base para operaciones     | `ctx := context.Background()`                                       |

## Intermedio de PGX 🎒

Pool, múltiples filas, inserciones y transacciones:

| Función / Elemento   | Descripción                              | Ejemplo usando `empleados`                                                            |
| -------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------- |
| `pgxpool.New`        | Crea un pool de conexiones               | `pgxpool.New(ctx, "postgres://user:pass@localhost/plantilla")`                        |
| `Query`              | Ejecutar SELECT con múltiples resultados | `dbpool.Query(ctx, "SELECT id, nombre FROM empleados")`                               |
| `rows.Next`          | Iterar sobre múltiples resultados        | `for rows.Next() { rows.Scan(&id, &nombre) }`                                         |
| `Exec`               | Ejecutar una consulta sin resultados     | `dbpool.Exec(ctx, "INSERT INTO empleados (nombre, edad) VALUES ($1, $2)", "Ana", 28)` |
| `Begin`              | Iniciar transacción                      | `tx, _ := dbpool.Begin(ctx)`                                                          |
| `tx.Exec`            | Ejecutar dentro de transacción           | `tx.Exec(ctx, "UPDATE empleados SET salario = salario + 500 WHERE id=$1", 1)`         |
| `Commit`, `Rollback` | Confirmar o revertir la transacción      | `tx.Commit(ctx)` / `tx.Rollback(ctx)`                                                 |

* ¿Qué hace un pool?
Un pool de conexiones te permite reutilizar conexiones a la base de datos en lugar de crear una nueva cada vez. Es esencial para aplicaciones web, APIs o cualquier sistema que maneje múltiples consultas.

a) Abre varias conexiones de antemano (ej. 10).

b) Las mantiene abiertas.

c) Cuando haces una consulta, te da una conexión del pool.

d) Cuando terminas, la conexión vuelve al pool para que otro la use.

## Avanzado PGX 🎓

Structs, prepared steatments y listen/notify:

| Función / Elemento            | Descripción                             | Ejemplo usando `empleados`                                              |
| ----------------------------- | --------------------------------------- | ----------------------------------------------------------------------- |
| `type Struct struct`          | Representación de filas como structs Go | `type Empleado struct { ID int; Nombre string }`                        |
| Mapear resultados a struct    | Manualmente mapear filas                | `rows.Scan(&e.ID, &e.Nombre)`                                           |
| `Prepare`                     | Consulta preparada (mejor rendimiento)  | `dbpool.Prepare(ctx, "getByID", "SELECT * FROM empleados WHERE id=$1")` |
| `QueryRow(... stmt.Name ...)` | Ejecutar consulta preparada             | `dbpool.QueryRow(ctx, "getByID", 2).Scan(&...)`                         |
| `LISTEN canal`                | Suscribirse a eventos PostgreSQL        | `conn.Exec(ctx, "LISTEN empleados_evento")`                             |
| `WaitForNotification`         | Esperar un mensaje `NOTIFY`             | `notif, _ := conn.WaitForNotification(ctx)`                             |
| `context.WithTimeout`         | Contexto con límite de tiempo           | `ctx, cancel := context.WithTimeout(ctx, 5*time.Second)`                |

## Experto PGX 😎

Herramientas y control avanzado:

| Herramienta / Función | Descripción                                 | Ejemplo usando `empleados`                                                           |
| --------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------ |
| `pgxscan`             | Mapea automáticamente filas a structs       | `pgxscan.Select(ctx, dbpool, &empleados, "SELECT * FROM empleados")`                 |
| `sqlc`                | Genera código Go desde SQL                  | En archivo `.sql`: `-- name GetEmpleado :one\nSELECT * FROM empleados WHERE id = $1` |
| `context.WithCancel`  | Cancelar operaciones programáticamente      | `ctx, cancel := context.WithCancel(context.Background())`                            |
| `zap.Logger`          | Logging estructurado para producción        | `logger.Info("Empleado creado", zap.String("nombre", nombre))`                       |
| Middleware            | Middleware para consultas (ej. logging SQL) | Interceptor personalizado en capa de servicio                                        |

## Un ejemplo rápido 👀

```go
type Empleado struct {
	ID     int
	Nombre string
	Edad   int
	Salario float64
}

// Consulta todos los empleados
rows, _ := dbpool.Query(ctx, "SELECT id, nombre, edad, salario FROM empleados")
defer rows.Close()

for rows.Next() {
	var e Empleado
	rows.Scan(&e.ID, &e.Nombre, &e.Edad, &e.Salario)
	fmt.Println(e)
}
```

## Notaciones importantes 📝

| 💡 Notación / Elemento       | ✅ Buenas prácticas y explicación                                                              |
| ---------------------------- | --------------------------------------------------------------------------------------------- |
| `$1`, `$2`, etc.             | Placeholder posicional de PostgreSQL. Úsalo siempre para evitar SQL Injection.                |
| `QueryRow(...)`              | Ejecuta una consulta que devuelve solo una fila. Debes llamar `.Scan(...)` obligatoriamente.  |
| `Scan(&var1, &var2, ...)`    | Asigna columnas de la fila a variables Go. El orden importa y debe coincidir con el `SELECT`. |
| `Query(...)`                 | Ejecuta una consulta que devuelve múltiples filas (`rows.Next()` + `Scan(...)`).              |
| `Exec(...)`                  | Ejecuta instrucciones sin retorno de filas (`INSERT`, `UPDATE`, `DELETE`).                    |
| `defer rows.Close()`         | Siempre cierra los `rows` después de usarlos para evitar fugas de recursos.                   |
| `defer conn.Close()`         | Cierra la conexión al finalizar su uso. Necesario con `pgx.Connect`.                          |
| `context.Background()`       | Contexto base sin cancelación ni timeout. Úsalo como punto de partida.                        |
| `context.WithTimeout(...)`   | Crea un contexto con tiempo límite (seguridad ante consultas largas o colgadas).              |
| `pgxpool.New(...)`           | Crea un pool de conexiones reutilizable (más eficiente que conexiones individuales).          |
| `Prepare(...)`               | Prepara una consulta para ejecución repetida (mejora el rendimiento).                         |
| `tx, _ := dbpool.Begin(...)` | Inicia una transacción. Necesario para operaciones ACID (atómicas).                           |
| `tx.Commit()` / `Rollback()` | Finaliza la transacción correctamente o revierte si hubo error.                               |
| `LISTEN canal`, `NOTIFY`     | Comunicación pub/sub con PostgreSQL. Ideal para recibir eventos del lado del servidor.        |
| `WaitForNotification(...)`   | Espera mensajes enviados por `NOTIFY`. Útil para reactividad.                                 |
| `type Struct struct {...}`   | Usa structs para representar filas de la base de datos en Go.                                 |
| `snake_case` ↔ `CamelCase`   | PostgreSQL usa `snake_case`, structs en Go usan `CamelCase`. Maneja mapeo manual o con tags.  |
| `pgxscan.Select(...)`        | Librería para mapear resultados directamente a structs Go.                                    |
| `sqlc`                       | Herramienta que genera código Go a partir de consultas SQL estáticas (`type-safe`).           |
| `logger.Info(...)`           | Usa logging estructurado (`zap`, `logrus`) en lugar de `fmt.Println`.                         |
| `defer tx.Rollback()`        | Asegura rollback si la transacción no se comitea (buena práctica defensiva).                  |


* Recomendaciones clave:
Evita interpolar variables en strings SQL. Usa $1, $2, etc.

Siempre maneja errores (err), incluso en .Scan, .Query, .Exec.

Cierra todo lo que abras (rows, conn, tx).

Usa contextos con timeout para evitar cuelgues o abusos.

Prefiere pgxpool para producción y aplicaciones concurrentes.

Mapea structs manualmente o usa herramientas como pgxscan o sqlc.

***

Venga, `pgx` no es tán complicado, pero tener las notas de los elementos más comúnes es una gran ayuda, lo más importante no es estudiarlos de forma independiente o saber que existen, es saber darles uso y aprovechamiento en proyectos reales, eso hará la diferencia.

![Study](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExNGQ2anBpcmo4NWttN3g1azk5OWE2cjVoOGtidjZtd3dldGNlZG5xbiZlcD12MV9naWZzX3NlYXJjaCZjdD1n/KCqO4k31TnkC2pT5LY/giphy.gif)
