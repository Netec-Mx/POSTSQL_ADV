# Capítulo 2: Indexación
---
## Laboratorio 2.1 – Creación de índices B-tree

### Objetivo  
Aprender a crear índices B-tree simples y compuestos para mejorar el rendimiento de las consultas en PostgreSQL.

### Requisitos  
- PostgreSQL instalado y funcionando.  
- Base de datos con tablas y datos para pruebas.

### Pasos

1. Acceder a `psql` y conectar a la base de datos de prueba:
   ```bash
   psql -U usuario -d basededatos
2.	Crear una tabla de ejemplo con datos:
    ```sql
    CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    ciudad VARCHAR(50),
    edad INT
    );

    INSERT INTO clientes (nombre, ciudad, edad) VALUES
    ('Ana', 'Madrid', 30),
    ('Luis', 'Barcelona', 25),
    ('Carlos', 'Madrid', 40),
    ('María', 'Valencia', 35),
    ('Sofía', 'Barcelona', 28);
3.	Crear un índice B-tree simple sobre la columna ciudad:
    ```sql
    CREATE INDEX idx_ciudad ON clientes(ciudad);
4.	Crear un índice B-tree compuesto sobre ciudad y edad:
    ```sql
    CREATE INDEX idx_ciudad_edad ON clientes(ciudad, edad);
5.	Verificar que los índices se crearon:
    ```sql
    \d clientes
#### Explicación
Los índices B-tree son el tipo de índice por defecto en PostgreSQL, ideales para búsqueda rápida y ordenamiento en columnas con datos discretos o rangos. Los índices compuestos permiten acelerar consultas que filtran o ordenan por varias columnas.

## Laboratorio 2.2 – Comparación de rendimiento con EXPLAIN
### Objetivo
Comparar el plan de ejecución de consultas con y sin índices para entender el impacto de la indexación.
### Requisitos
•	Laboratorio 2.1 completado.
•	Tabla clientes con índices creados.
### Pasos
1.	Ejecutar consulta sin usar índice (desactivar temporalmente los índices):
    ```sql
    SET enable_indexscan = off;

    EXPLAIN ANALYZE SELECT * FROM clientes WHERE ciudad = 'Madrid';
2.	Volver a activar el uso de índices:
    ```sql
    SET enable_indexscan = on;
3.	Ejecutar la misma consulta usando índices:
    ```sql
    EXPLAIN ANALYZE SELECT * FROM clientes WHERE ciudad = 'Madrid';
4.	Comparar tiempos de ejecución y método de acceso en los planes mostrados.

#### Explicación
EXPLAIN ANALYZE muestra cómo PostgreSQL ejecuta una consulta, incluyendo el uso de índices y tiempos reales. Desactivar índices permite observar cómo la consulta se ejecuta con escaneo secuencial, útil para medir la mejora que ofrece el índice.

## Laboratorio 2.3 – Eliminación y recreación de índices
### Objetivo
Observar el impacto en el rendimiento tras eliminar índices y recuperarlo al recrearlos.
### Requisitos
•	Laboratorio 2.1 completado.
### Pasos
1.	Eliminar el índice idx_ciudad:
    ```sql
    DROP INDEX idx_ciudad;
2.	Ejecutar consulta y medir rendimiento:
    ```sql
    EXPLAIN ANALYZE SELECT * FROM clientes WHERE ciudad = 'Madrid';
3.	Volver a crear el índice:
    ```sql
    CREATE INDEX idx_ciudad ON clientes(ciudad);
4.	Ejecutar nuevamente la consulta y comparar resultados:
    ```sql
    EXPLAIN ANALYZE SELECT * FROM clientes WHERE ciudad = 'Madrid';
#### Explicación
Eliminar un índice obliga a PostgreSQL a hacer escaneo secuencial, lo que puede impactar negativamente el rendimiento. Al recrear el índice, se recupera la velocidad de acceso eficiente.

## Laboratorio 2.4 – Optimización de índices
### Objetivo
Detectar índices redundantes y optimizar la estructura de índices para mejorar la eficiencia.
### Requisitos
•	Laboratorio 2.1 completado.
### Pasos
1.	Crear un índice redundante para observar duplicidad:
    ```sql
    CREATE INDEX idx_nombre_ciudad ON clientes(nombre, ciudad);
    CREATE INDEX idx_nombre ON clientes(nombre);
2.	Consultar los índices existentes:
    ```sql
    SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'clientes';
3.	Analizar consultas que pueden beneficiarse de índices compuestos vs simples.
4.	Eliminar índices redundantes (por ejemplo, idx_nombre si se prefiere el compuesto):
    ```sql
    DROP INDEX idx_nombre;
5.	Volver a verificar los índices restantes.

#### Explicación
Índices redundantes consumen espacio y afectan rendimiento en inserciones/actualizaciones. Un índice compuesto bien diseñado puede reemplazar múltiples índices simples, optimizando uso de recursos.
