# Capítulo 8: Actualización, Estadísticas y Monitoreo

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo7/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo1/)**

---

## Laboratorio 8.1 – Actualización mayor con `pg_upgrade`

### Objetivo  
Realizar la actualización de una instalación PostgreSQL de versión mayor (por ejemplo, de v14 a v15) utilizando `pg_upgrade`, minimizando el tiempo de inactividad y asegurando la integridad de los datos.

### Requisitos  
- Respaldo completo previo (recomendado).  
- Acceso administrativo a los servidores y permisos para detener servicios.  
- Instalación de la nueva versión de PostgreSQL.

### Pasos

1. **Realizar respaldo completo de la base de datos (recomendado):**  
    ```bash
    pg_dumpall -U postgres -f respaldo_completo.sql
2.	Detener el servicio PostgreSQL versión antigua:
    ```bash
    sudo systemctl stop postgresql@14-main
3.	Ejecutar pg_upgrade:
Supongamos que los datos viejos están en /var/lib/postgresql/14/main y los nuevos en /var/lib/postgresql/15/main:
    ```bash
    pg_upgrade \
    --old-datadir=/var/lib/postgresql/14/main \
    --new-datadir=/var/lib/postgresql/15/main \
    --old-bindir=/usr/lib/postgresql/14/bin \
    --new-bindir=/usr/lib/postgresql/15/bin \
    --check
    (Primero correr con --check para validar).
4.	Ejecutar la actualización real:
    ```bash
    pg_upgrade \
    --old-datadir=/var/lib/postgresql/14/main \
    --new-datadir=/var/lib/postgresql/15/main \
    --old-bindir=/usr/lib/postgresql/14/bin \
    --new-bindir=/usr/lib/postgresql/15/bin
5.	Iniciar el nuevo servidor:
    ```bash
    sudo systemctl start postgresql@15-main
6.	Verificar que la base de datos funcione correctamente:
    ```bash
    psql -U postgres -c "SELECT version();"
    psql -U postgres -d basededatos -c "SELECT COUNT(*) FROM tabla_importante;"
#### Explicación
pg_upgrade permite migrar datos rápidamente entre versiones mayores sin necesidad de dump y restore completos, reduciendo tiempo de inactividad.

## Laboratorio 8.2 – Uso de vistas estadísticas
### Objetivo
Consultar las vistas estadísticas internas para monitorear la actividad y detectar posibles problemas de rendimiento.
### Requisitos
•	Acceso a consola psql.
•	Datos y actividad en la base para analizar.
### Pasos
1.	Consultar las conexiones y actividad actual:
    ```sql
    SELECT * FROM pg_stat_activity;
2.	Revisar estadísticas de tablas de usuario:
    ```sql
    SELECT * FROM pg_stat_user_tables ORDER BY seq_scan DESC LIMIT 10;
3.	Consultar estadísticas generales de la base de datos:
    ```sql
    SELECT * FROM pg_stat_database WHERE datname = 'basededatos';
4.	Identificar consultas lentas y tablas con alto número de accesos o escaneos secuenciales.
### Explicación
Estas vistas permiten tener visibilidad sobre el comportamiento de la base y ayudan a identificar cuellos de botella o problemas de rendimiento.

## Laboratorio 8.3 – Análisis con EXPLAIN ANALYZE BUFFERS
### Objetivo
Obtener y entender los planes detallados de ejecución de consultas para optimizar el rendimiento.
### Requisitos
•	Base con tablas y datos.
•	Acceso a consola SQL.
### Pasos
1.	Ejecutar una consulta con el comando:
    ```sql
    EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM tabla WHERE columna = 'valor';
2.	Revisar el plan generado, tiempos, y uso de buffers (lectura de disco vs cache).
3.	Comparar planes de consultas similares con y sin índices.
4.	Ajustar índices o consultas basándose en la información obtenida.
#### Explicación
EXPLAIN ANALYZE BUFFERS proporciona información detallada que ayuda a identificar cuellos de botella, operaciones costosas y optimizar el acceso a datos.
 
## Laboratorio 8.4 – Creación y uso de estadísticas extendidas
### Objetivo
Mejorar el planificador de consultas creando estadísticas extendidas para columnas relacionadas y verificar el impacto en la optimización.
### Requisitos
•	PostgreSQL 10 o superior.
•	Tablas con columnas relacionadas.
### Pasos
1.	Crear estadísticas extendidas para columnas que se usan juntas en consultas:
    ```sql
    CREATE STATISTICS estadisticas_relacionadas (ndistinct, dependencies) ON columna1, columna2 FROM tabla;
2.	Ejecutar ANALYZE para recolectar las estadísticas:
    ```sql
    ANALYZE tabla;
3.	Ejecutar consultas que involucren esas columnas y comparar planes con EXPLAIN.
4.	Observar mejoras en planes y tiempos de ejecución.
#### Explicación
Las estadísticas extendidas permiten al planificador tomar decisiones más precisas sobre combinaciones de columnas, mejorando la eficiencia de las consultas.

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo7/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo1/)**

---
