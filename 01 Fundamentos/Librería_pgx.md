# Entendiendo PGX (Go + PostgreSQL) 📦

`pgx` es una librería nativa de `Go` para conectarse y trabajar con bases de datos PostgreSQL. Se considera una de las más rápidas y rica en funcionalidades a comparación del paquete estándar `database/sql`.

## Instalación ⬇️💻

Desde la terminal se deberá ingresar:

`go get github.com/jackc/pgx/v5
`
Antes de darle uso a la librería o importarla al código en `Go` se debe generar un módulo en la carpeta del proyecto, luego agregar el main.go y así podremos importar la librería.


***

## Lo básico de PGX

Tomemos en cuenta que tiene que haberse creado la base de datos y tabla de forma previa, desde go y ya instalada la librería
