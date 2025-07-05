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

## Lo básico de PGX

Tomemos en cuenta que tiene que haberse creado la base de datos y tabla de forma previa, desde go y ya que hayamos importado la librería podremos hacer uso de elementos y funciones básicas de `pgx`:

| Función / Elemento     | Descripción                        | Ejemplo usando `empleados`                                          |
| ---------------------- | ---------------------------------- | ------------------------------------------------------------------- |
| `pgx.Connect`          | Conexión directa                   | `pgx.Connect(ctx, "postgres://user:pass@localhost/plantilla")`      |
| `QueryRow`             | Consulta que retorna una sola fila | `conn.QueryRow(ctx, "SELECT nombre FROM empleados WHERE id=$1", 1)` |
| `Scan`                 | Leer el resultado de una fila      | `.Scan(&nombre)`                                                    |
| `context.Background()` | Contexto base para operaciones     | `ctx := context.Background()`                                       |

