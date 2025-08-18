#  Capítulo 2: Indexación
## Objetivos de los Laboratorios
-	Construir índices B-Tree avanzados (multicolumna, parciales y sobre expresiones) y evaluar su impacto real.
-	Diagnosticar cuándo un índice ayuda y cuándo no, usando EXPLAIN ANALYZE, estadísticas y parámetros de autovacuum.
-	Profundizar en métricas críticas (Heap Fetches, Rows Removed by Filter, Loops, Buffercache Hits).
-	Aplicar buenas prácticas de diseño de índices y demostrar problemas comunes (selectividad baja, funciones en columnas, inserciones masivas).
________________________________________
## Laboratorio 2.1 – Creación y Diagnóstico de Índices B-Tree
### Paso 1. Preparación de datos
```sql
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nombre TEXT,
    apellido TEXT,
    activo BOOLEAN,
    fecha_registro DATE DEFAULT now()
);
```
-- Insertamos 1 millón de registros con distribución controlada
```sql
INSERT INTO clientes (nombre, apellido, activo)
SELECT 
    md5(random()::text), 
    (ARRAY['García','López','Hernández','Martínez','Ramírez'])[floor(random()*5)+1],
    (random() < 0.7) -- 70% activos
FROM generate_series(1, 1000000);
```
👉 Con esto tendremos una tabla grande, con apellidos poco selectivos y un booleano (activo) de baja selectividad.
________________________________________
### Paso 2. Índice estándar
```sql
CREATE INDEX idx_clientes_apellido ON clientes (apellido);
Consulta de prueba:
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM clientes WHERE apellido = 'García';
```
👉 Observa:
-	Index Scan o Bitmap Index Scan.
-	Heap Fetches: ¿cuántas veces tuvo que ir a la tabla tras usar el índice?

### Paso 3. Índice multicolumna
```sql
CREATE INDEX idx_clientes_apellido_nombre ON clientes (apellido, nombre);
Consulta de prueba:
EXPLAIN (ANALYZE, BUFFERS)
SELECT * 
FROM clientes 
WHERE apellido = 'García' AND nombre LIKE 'a%';
```
👉 Compara con el índice simple.

Pregunta de análisis: ¿qué mejora y qué sigue igual?

### Paso 4. Índice parcial
```sql
CREATE INDEX idx_clientes_activos ON clientes (apellido)
WHERE activo = true;
Consulta de prueba:
EXPLAIN (ANALYZE, BUFFERS)
SELECT * 
FROM clientes 
WHERE apellido = 'García' AND activo = true;
```
👉 Analiza:
-	Diferencia en Rows Removed by Filter.
-	Menor número de páginas leídas vs índice normal.

### Paso 5. Índice sobre expresión
```sql
CREATE INDEX idx_clientes_apellido_lower ON clientes (LOWER(apellido));
```
Comparación:
-- Sin índice de expresión
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM clientes WHERE LOWER(apellido) = 'garcía';
```
-- Con índice de expresión
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM clientes WHERE LOWER(apellido) = 'garcía';
```
👉 Verifica cómo cambia de Seq Scan → Index Scan.

## Laboratorio 2.2 – Casos donde los B-Tree NO ayudan
### Paso 1. Baja selectividad
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM clientes WHERE activo = true;
```
👉 Aunque hay un índice, PostgreSQL hará un Seq Scan porque casi toda la tabla cumple la condición.

### Paso 2. Agregación masiva
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT COUNT(*) FROM clientes;
```
👉 Ningún índice ayuda: se necesita un Seq Scan.

### Paso 3. Inserciones masivas y page splits
-- Índice con fillfactor para optimizar inserciones
```sql
CREATE INDEX idx_clientes_fecha_registro
ON clientes (fecha_registro) WITH (fillfactor = 70);
```
-- Simulación de inserciones diarias
```sql
INSERT INTO clientes (nombre, apellido, activo, fecha_registro)
SELECT md5(random()::text), 'Nuevo', true, now()
FROM generate_series(1,100000);
```
👉 Usa pg_stat_all_indexes para observar crecimiento y validación de page splits.

## Laboratorio 2.3 – Interpretación avanzada con EXPLAIN
### Paso 1. Métricas críticas
Ejecuta una consulta con BUFFERS y analiza:
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM clientes WHERE apellido = 'Ramírez';
```
👉 Interpreta:
-	Heap Fetches: ¿el índice está accediendo demasiado a la tabla? → considerar índices covering.
-	Rows Removed by Filter: ¿el índice devuelve demasiados falsos positivos? → usar parcial.
-	Buffers: cache hits vs lecturas desde disco.

### Paso 2. Índices covering (INCLUDE)
```sql
CREATE INDEX idx_clientes_apellido_include ON clientes (apellido) INCLUDE (nombre);
Consulta:
EXPLAIN (ANALYZE, BUFFERS)
SELECT apellido, nombre FROM clientes WHERE apellido = 'López';
```
👉 Observa cómo ya no necesita Heap Fetches.

## Tarea Final del Capítulo
1.	Diseñar un set de consultas frecuentes (ej. búsquedas por apellido, búsquedas por clientes activos, búsquedas case-insensitive).
2.	Crear diferentes tipos de índices (estándar, multicolumna, parcial, expresión, covering).
3.	Medir el impacto con EXPLAIN (ANALYZE, BUFFERS) y documentar:
-	Costos estimados vs reales.
-	Accesos al heap.
-	Páginas leídas desde disco.
-	Diferencia en tiempo de ejecución.

📋 Guía de Interpretación de EXPLAIN (Checklist Experto)
Cuando ejecutes:

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...;
```
Tendrás salida con estimaciones y resultados reales. Interprétala así:

 ### 1. Tipo de operación
-	Seq Scan → PostgreSQL lee toda la tabla.
➝ Útil si no hay índices o la selectividad es muy baja.
-	Index Scan → usa el índice y va al heap por cada fila.
➝ Puede generar muchos Heap Fetches.
-	Bitmap Index Scan → el índice devuelve posiciones de páginas, se agrupan y luego se leen en bloque.
➝ Eficiente para condiciones que devuelven muchas filas.
✔️ Pregunta: ¿el plan usó índice cuando esperabas?

 ### 2. Costos estimados
Cada operación tiene:
(cost=0.29..8.30 rows=5 width=64)
-	0.29 → costo de inicio (primer fila).
-	8.30 → costo total estimado.
-	rows=5 → filas estimadas.
-	width=64 → tamaño medio de cada fila (bytes).
✔️ Pregunta: ¿la estimación de filas (rows) se acerca a la realidad? Si no, revisar estadísticas con ANALYZE.

 ### 3. Resultados reales

Ejemplo:
(actual time=0.020..0.025 rows=5 loops=1)
-	actual time=... → tiempo real (inicio..fin).
-	rows=5 → filas reales devueltas.
-	loops=1 → número de veces que se ejecutó este plan.
✔️ Pregunta: ¿las filas reales coinciden con las estimadas? Si no, el optimizador puede elegir mal los planes.

 ### 4. Métricas críticas
-	Index Cond → condición usada en el índice.
➝ Si está vacía o solo ves Filter, el índice no se aprovechó.
-	Filter → condición aplicada después de leer datos.
➝ Si aquí se eliminan muchas filas (Rows Removed by Filter), quizá necesites un índice parcial.
-	Heap Fetches → accesos a la tabla después de usar el índice.
➝ Demasiados fetches → evalúa índices covering con INCLUDE.
✔️ Pregunta: ¿estás filtrando demasiado tarde? ¿se puede optimizar con índices parciales o covering?

 ### 5. Estadísticas de memoria y cache (BUFFERS)

Con (BUFFERS) aparecen:
-	shared hit → páginas leídas desde cache (rápido).
-	shared read → páginas leídas desde disco (lento).
-	shared dirtied → páginas modificadas.
-	shared written → páginas escritas.
✔️ Pregunta: ¿hay demasiados read? → mejorar índices, aumentar cache (shared_buffers) o reescribir la consulta.

 ### 6. Escalabilidad y bucles
-	Si loops es alto → la operación se repite muchas veces.
➝ Común en Nested Loop. Puede explotar con millones de filas.
-	Si ves Hash Join o Merge Join, revisa si los índices permiten un Index Nested Loop más eficiente.
✔️ Pregunta: ¿el plan es escalable para millones de filas o solo funciona en pruebas pequeñas?

 ### 7. Diagnóstico final
-	¿El plan usó el índice correcto?
-	¿Las estimaciones de filas fueron realistas?
-	¿Se generaron demasiados Heap Fetches o Rows Removed by Filter?
-	¿El acceso a disco (read) es alto comparado con hit?
-	¿El plan elegido escala con más datos?

## Ejemplo Rápido Adicional
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM clientes WHERE LOWER(apellido) = 'garcía';
```
Salida (simplificada):
```
Seq Scan on clientes  (cost=0.00..35000.00 rows=500 width=64)
(actual time=0.05..120.00 rows=500 loops=1)
  Filter: (lower(apellido) = 'garcía')
  Rows Removed by Filter: 999500
  Buffers: shared hit=100, read=2000
```
✅ Diagnóstico con checklist:
```
1.	Seq Scan → No usó índice.
2.	Rows Removed by Filter = 999500 → pésima eficiencia.
3.	read=2000 → demasiadas lecturas desde disco.
```
👉 Solución: crear un índice de expresión LOWER(apellido).

Con esta guía, puedes leer un plan de ejecución, detectando cuellos de botella y justificando decisiones de indexación.

##  Tarea Final – Capítulo 2: Indexación (Nivel Avanzado)
 
## Objetivo
Diseñar, crear y evaluar estrategias de indexación avanzadas sobre una tabla de clientes simulada. El alumno deberá justificar con evidencia (EXPLAIN ANALYZE, BUFFERS) por qué un índice mejora (o no) el rendimiento.

### Preparación de datos
```sql
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nombre TEXT,
    apellido TEXT,
    activo BOOLEAN,
    fecha_registro DATE DEFAULT now()
);
```
-- Insertar 1 millón de registros de prueba
```sql
INSERT INTO clientes (nombre, apellido, activo, fecha_registro)
SELECT 
    md5(random()::text),
    (ARRAY['García','López','Hernández','Martínez','Ramírez'])[floor(random()*5)+1],
    (random() < 0.7), -- 70% activos
    now() - (random() * 365)::int * interval '1 day'
FROM generate_series(1,1000000);
```

### Parte 1. Consultas frecuentes
```
Los alumnos deben ejecutar estas consultas representativas:
1.	Búsqueda exacta por apellido
2.	SELECT * FROM clientes WHERE apellido = 'García';
3.	Búsqueda combinada (apellido + nombre inicial)
4.	SELECT * FROM clientes 
5.	WHERE apellido = 'Martínez' AND nombre LIKE 'a%';
6.	Búsqueda solo de clientes activos
7.	SELECT * FROM clientes WHERE activo = true;
8.	Búsqueda case-insensitive
9.	SELECT * FROM clientes WHERE LOWER(apellido) = 'lópez';
10.	Consulta de reporte parcial (solo columnas específicas)
11.	SELECT apellido, nombre FROM clientes WHERE apellido = 'Ramírez';
```

### Parte 2. Creación de índices avanzados
```
El alumno deberá crear y evaluar los siguientes índices:
1.	Índice estándar
2.	CREATE INDEX idx_apellido ON clientes (apellido);
3.	Índice multicolumna
4.	CREATE INDEX idx_apellido_nombre ON clientes (apellido, nombre);
5.	Índice parcial
6.	CREATE INDEX idx_activos ON clientes (apellido)
7.	WHERE activo = true;
8.	Índice sobre expresión
9.	CREATE INDEX idx_apellido_lower ON clientes (LOWER(apellido));
10.	Índice covering
11.	CREATE INDEX idx_apellido_include ON clientes (apellido) INCLUDE (nombre);
```

### Parte 3. Medición del impacto

Ejecutar cada consulta con y sin índice, usando:
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...
```
Y documentar en una tabla comparativa como la siguiente:

![Tabla de resultados](TablaIndices.png)
Consulta	Índice aplicado	Costo estimado	Filas estimadas vs reales	Heap Fetches	Buffers (hit vs read)	Tiempo ejecución (ms)	Observaciones
Buscar apellido exacto	idx_apellido	0.29..8.30	500 vs 498	500	hit=50, read=5	0.05	Buen match, planificador acertó
Case-insensitive	idx_apellido_lower	0.29..12.00	200 vs 195	200	hit=40, read=3	0.04	El índice evita el Seq Scan
Clientes activos	idx_activos	0.30..500	700000 vs 690000	Alto	read elevado	85.0	Muy baja selectividad, no conviene índice

### Parte 4. Informe final
El alumno deberá entregar un informe escrito que incluya:
-	Capturas de EXPLAIN (ANALYZE, BUFFERS).
-	Comparación de cada índice en términos de:
    - Diferencia entre costos estimados y reales.
    - Reducción (o no) de Heap Fetches.
    - Impacto en lecturas desde disco vs cache.
    - Variación en tiempo de ejecución.
-	Conclusiones sobre:
    - Qué índices son más útiles en este dataset?
    - ¿Qué índices son inútiles o incluso perjudiciales?
    - ¿Cómo cambia la estrategia de indexación si los datos crecen a 10M registros?

📌 Con esta tarea final, hemos practicado el diseño, diagnóstico y justificación del uso de índices, logrando pensar como un DBA de PostgreSQL en producción.
