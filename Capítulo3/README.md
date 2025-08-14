# Capítulo 3: Manejo de Transacciones

---

## Laboratorio 3.1 – Uso básico de transacciones

### Objetivo  
Comprender el funcionamiento básico de las transacciones en PostgreSQL mediante el uso de `BEGIN`, `COMMIT` y `ROLLBACK`.

### Requisitos  
- PostgreSQL instalado y funcionando.  
- Tabla de prueba creada con datos.

### Pasos

1. Acceder a `psql` y seleccionar la base de datos:
   ```bash
   psql -U usuario -d basededatos
2.	Crear una tabla simple para pruebas:
    ```sql
    CREATE TABLE cuentas (
    id SERIAL PRIMARY KEY,
    titular VARCHAR(100),
    saldo NUMERIC
    );

    INSERT INTO cuentas (titular, saldo) VALUES ('Juan', 1000), ('Ana', 1500);
3.	Iniciar una transacción y actualizar saldo:
    ```sql
    BEGIN;
    UPDATE cuentas SET saldo = saldo - 100 WHERE titular = 'Juan';
    UPDATE cuentas SET saldo = saldo + 100 WHERE titular = 'Ana';
    -- No hacer COMMIT todavía
4.	Consultar los saldos dentro de la misma sesión:
    ```sql
    SELECT * FROM cuentas;
5.	Abrir otra sesión psql y consultar la tabla:
    ```sql
    SELECT * FROM cuentas;
    Nota: Verás que los cambios aún no son visibles aquí.
6.	Volver a la primera sesión y realizar un ROLLBACK para deshacer cambios:
    ```sql
    ROLLBACK;
7.	Consultar nuevamente los saldos para confirmar reversión.
8.	Repetir la transacción, pero ahora realizar un COMMIT para confirmar cambios:
    ```sql
    BEGIN;
    UPDATE cuentas SET saldo = saldo - 100 WHERE titular = 'Juan';
    UPDATE cuentas SET saldo = saldo + 100 WHERE titular = 'Ana';
    COMMIT;
9.	Verificar que los cambios ahora son visibles en ambas sesiones.

### Explicación
Este laboratorio muestra cómo las transacciones agrupan operaciones para que se ejecuten todas o ninguna, garantizando atomicidad y consistencia.

## Laboratorio 3.2 – Bloqueos pesimistas
### Objetivo
Simular bloqueos pesimistas en PostgreSQL para entender cómo se gestionan conflictos entre sesiones concurrentes.
### Requisitos
•	PostgreSQL en ejecución.
•	Tabla con datos para prueba (por ejemplo, la tabla cuentas del laboratorio 3.1).
### Pasos
1.	Abrir dos sesiones psql conectadas a la misma base.
2.	En la sesión 1, iniciar transacción y bloquear una fila:
    ```sql
    BEGIN;
    SELECT * FROM cuentas WHERE titular = 'Juan' FOR UPDATE;
    -- No hacer COMMIT ni ROLLBACK todavía
3.	En la sesión 2, intentar actualizar la misma fila:
- Esta operación quedará bloqueada esperando a que la sesión 1 termine.
4.	Volver a sesión 1 y finalizar la transacción con:
    ```sql
    COMMIT;
5.	La sesión 2 continuará y podrá completar la actualización.
6.	Observar y analizar el comportamiento.
### Explicación
El bloqueo FOR UPDATE impide que otras transacciones modifiquen la fila hasta que se libere el bloqueo, previniendo inconsistencias.

## Laboratorio 3.3 – Observación de MVCC (Multi-Version Concurrency Control)
### Objetivo
Observar cómo PostgreSQL maneja versiones múltiples de filas para permitir concurrencia sin bloqueos pesados.
### Requisitos
•	Tabla cuentas con datos.
•	Dos sesiones psql abiertas.
### Pasos
1.	En la sesión 1, ejecutar una consulta para obtener xmin y xmax (identificadores de versión):
    ```sql
    SELECT id, titular, saldo, xmin, xmax FROM cuentas;
2.	En la sesión 2, iniciar transacción y actualizar una fila:
    ```sql
    BEGIN;
    UPDATE cuentas SET saldo = saldo + 200 WHERE titular = 'Juan';
3.	En la sesión 1, volver a consultar la tabla con xmin y xmax.
4.	Observar que la fila tiene una versión nueva (nuevo xmin) mientras la antigua persiste visible para la sesión 1.
5.	En la sesión 2, hacer COMMIT.
6.	En la sesión 1, consultar nuevamente y notar el cambio en versiones.
#### Explicación
MVCC permite que diferentes transacciones vean versiones distintas de una fila para evitar bloqueos, manteniendo consistencia.


## Laboratorio 3.4 – Análisis de VACUUM y autovacuum
### Objetivo
Entender la función de VACUUM y autovacuum para limpiar filas obsoletas y mantener el rendimiento.
### Requisitos
•	PostgreSQL en ejecución.
•	Tabla con actividad (como cuentas).
### Pasos
1.	Realizar múltiples actualizaciones y eliminaciones en la tabla para generar filas muertas:
    ```sql
    UPDATE cuentas SET saldo = saldo + 50 WHERE titular = 'Ana';
    DELETE FROM cuentas WHERE titular = 'Carlos'; -- Si existe
2.	Ejecutar manualmente VACUUM:
    ```sql
    VACUUM VERBOSE cuentas;
3.	Observar el reporte detallado del proceso.
4.	Consultar parámetros relacionados con autovacuum:
    ```sql
    SHOW autovacuum;
    SHOW autovacuum_vacuum_threshold;
    SHOW autovacuum_vacuum_scale_factor;
5.	Simular actividad intensa para forzar autovacuum (puede usar scripts o varias actualizaciones).
6.	Revisar logs para verificar ejecución de autovacuum o usar vistas del sistema:
    ```sql
    SELECT * FROM pg_stat_activity WHERE query LIKE '%autovacuum%';
#### Explicación
VACUUM limpia espacio ocupado por versiones antiguas de filas (tuplas muertas) para evitar crecimiento descontrolado de tablas y mantener performance. Autovacuum automatiza esta tarea.
