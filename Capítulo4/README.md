# Capítulo 4: Respaldos y Recuperación
---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo3/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo5/)**

---

## Laboratorio 4.1 – Respaldo lógico con `pg_dump` y restauración con `pg_restore`

### Objetivo  
Realizar un respaldo lógico de una base de datos usando `pg_dump` y restaurar ese respaldo en un nuevo esquema o instancia con `pg_restore`.

### Requisitos  
- PostgreSQL instalado y funcionando.  
- Permisos para crear bases nuevas.
- Acceso a la base de datos con datos para respaldar.  
- Para cumplir con los requisitos necesitas tener una base de datos ventas con un esquema y algunos datos.

### Pasos para crear la base de datos ventas
### 1. Conectarte a PostgreSQL como superusuario
```bash
% sudo -i -u postgres psql
```
### 2. Crear la base de datos ventas
```sql
CREATE DATABASE ventas;
```
Conéctate a la nueva base de datos:
```
\c ventas
```
### 3. Crear tablas
Vamos a usar dos tablas: clientes y pedidos.
- -- Tabla de clientes
```sql
CREATE TABLE clientes (
    id_cliente SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    fecha_registro DATE DEFAULT CURRENT_DATE
);
```
-- Tabla de pedidos
```sql
CREATE TABLE pedidos (
    id_pedido SERIAL PRIMARY KEY,
    id_cliente INT REFERENCES clientes(id_cliente),
    fecha DATE NOT NULL,
    monto NUMERIC(10,2) NOT NULL,
    estado VARCHAR(20) DEFAULT 'pendiente'
);
```
### 4. Insertar datos de prueba
-- Insertar clientes
```sql
INSERT INTO clientes (nombre, email) VALUES
('Ana López', 'ana@example.com'),
('Carlos Pérez', 'carlos@example.com'),
('María Gómez', 'maria@example.com');
```
-- Insertar pedidos
```sql
INSERT INTO pedidos (id_cliente, fecha, monto, estado) VALUES
(1, '2025-08-01', 250.50, 'pagado'),
(2, '2025-08-03', 125.00, 'pendiente'),
(3, '2025-08-05', 500.75, 'enviado'),
(1, '2025-08-10', 300.00, 'pendiente');
```
### 5. Verificar que los datos se insertaron
```sql
SELECT * FROM clientes;
SELECT * FROM pedidos;
\q
```

### Pasos para realizar el laboratorio 4.1

1. Realizar respaldo lógico de la base de datos `ventas` en formato comprimido personalizado:
  ```bash
   sudo -i -u postgres
   pg_dump -U postgres -F c -f respaldo.bak ventas
  ```
2.	Crear una nueva base de datos para restaurar:
```bash
createdb -U postgres basededatos_restaurada
```
3.	Restaurar el respaldo usando pg_restore:
```bash
pg_restore -U postgres -d basededatos_restaurada -v respaldo.bak
```
4.	Conectarse a la base restaurada y verificar tablas y datos:
```bash
psql -U postgres -d basededatos_restaurada
\dt
```
```sql
SELECT * FROM clientes;
SELECT * FROM pedidos;
```
### Explicación
El respaldo lógico crea un volcado que contiene comandos para reconstruir la base de datos. El formato personalizado (-F c) permite restaurar objetos selectivamente con pg_restore.


### Laboratorio: Respaldo y Recuperación Selectiva con pg_dump

1. Preparación de la Base de Datos

```bash
Primero, crea la base de datos y las tres tablas (Productos, Clientes, Ventas) con datos mínimos.
```
1.1. Crear la Base de Datos

```sql
createdb mi_tienda
```

1.2. Crear Tablas e Insertar Datos

Conéctate a la base de datos y ejecuta los siguientes comandos SQL:
```sql
psql -d mi_tienda
SQL
-- Tabla Productos
CREATE TABLE Productos (
    id_producto SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio NUMERIC(10, 2) NOT NULL
);

-- Insertar datos iniciales en Productos
INSERT INTO Productos (nombre, precio) VALUES
('Laptop', 1200.00),
('Mouse Inalámbrico', 25.50),
('Monitor 27"', 350.00);

-- Tabla Clientes
CREATE TABLE Clientes (
    id_cliente SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE
);

-- Insertar datos en Clientes
INSERT INTO Clientes (nombre, email) VALUES
('Ana Gómez', 'ana.gomez@mail.com'),
('Luis Pérez', 'luis.perez@mail.com');

-- Tabla Ventas
CREATE TABLE Ventas (
    id_venta SERIAL PRIMARY KEY,
    id_cliente INTEGER REFERENCES Clientes(id_cliente),
    id_producto INTEGER REFERENCES Productos(id_producto),
    cantidad INTEGER NOT NULL,
    fecha_venta DATE DEFAULT CURRENT_DATE
);

-- Insertar datos en Ventas
INSERT INTO Ventas (id_cliente, id_producto, cantidad) VALUES
(1, 1, 1), -- Ana compra una Laptop
(2, 2, 2); -- Luis compra dos Mouse Inalámbricos

\q
-- Salir de psql
________________________________________
```

2. Respaldo Lógico de la Base de Datos con pg_dump
Ahora, realiza el respaldo completo de la base de datos mi_tienda en un archivo llamado respaldo_completo.sql.
Bash
pg_dump -U postgres -F c mi_tienda > respaldo_completo.dump
Nota: Utilizamos el formato personalizado (-F c) para mayor flexibilidad, aunque para la recuperación de una sola tabla, el formato texto también funciona. Usamos un archivo .dump en lugar de .sql para denotar el formato personalizado.
________________________________________
3. Simular un Cambio (Punto de No Retorno)
Para demostrar la recuperación, modificaremos la tabla Productos de forma indeseada.
Conéctate a la base de datos y ejecuta lo siguiente:
Bash
psql -d mi_tienda
SQL
-- ⚠️ SIMULACIÓN DE ACCIDENTE/ERROR:
-- Se agrega un nuevo producto con un precio incorrecto, o se borra un producto
INSERT INTO Productos (nombre, precio) VALUES
('Teclado Mecánico', 1.00); -- ¡El precio es claramente erróneo!

-- Verificar los datos actuales (con el error)
SELECT * FROM Productos;

\q
________________________________________
4. Recuperación Selectiva de la Tabla Productos
Para recuperar la tabla Productos al estado exacto del respaldo, puedes usar el comando pg_restore junto con el flag -t (o --table).
4.1. Limpiar la Tabla Actual (Necesario para una Recuperación 'Tal Cual')
Primero, debes eliminar la tabla Productos en la base de datos actual para que pg_restore pueda recrearla y cargar los datos desde el respaldo sin conflictos de claves primarias o datos.
Bash
psql -d mi_tienda -c "DROP TABLE Productos CASCADE;"
Nota: El modificador CASCADE es crucial para eliminar las dependencias (como la referencia en la tabla Ventas). Después, el pg_restore recreará la tabla Productos y la rellenará, pero no recreará la dependencia en Ventas porque solo estamos restaurando una tabla. Si esto fuera una base de datos de producción, este paso sería muy delicado y se preferiría una restauración a una base de datos temporal.
4.2. Restaurar la Tabla Productos
Utiliza pg_restore para restaurar solo la definición y los datos de la tabla Productos desde el archivo de respaldo.
Bash
pg_restore -U postgres -d mi_tienda -t Productos respaldo_completo.dump
4.3. Corregir la Dependencia (Opcional, pero necesario si usaste CASCADE)
Como usamos CASCADE para eliminar la tabla, la referencia (Foreign Key) de Ventas a Productos se eliminó. Debemos recrearla.
Bash
psql -d mi_tienda
SQL
-- Recrear la restricción de clave foránea
ALTER TABLE Ventas
ADD CONSTRAINT fk_ventas_producto
FOREIGN KEY (id_producto)
REFERENCES Productos(id_producto);

-- Verificar que la tabla Productos está correcta (solo los 3 registros originales)
SELECT * FROM Productos;

\q
Resultado: La tabla Productos habrá regresado a su estado con los 3 registros originales (Laptop, Mouse Inalámbrico, Monitor 27"), ignorando el registro erróneo (Teclado Mecánico) que se insertó después del respaldo.
________________________________________
📝 Resumen de Comandos Clave
Acción	Comando Principal	Descripción
Respaldo Lógico Completo	pg_dump	Crea un archivo de respaldo con el esquema y datos de toda la base.
Recuperación Selectiva	pg_restore -t NombreTabla	Restaura el esquema y los datos solo para la tabla especificada.



## Laboratorio 4.2 – Respaldo físico con pg_basebackup
### Objetivo
Realizar un respaldo físico completo de PostgreSQL usando pg_basebackup y examinar los archivos generados.
### Requisitos
- Acceso al servidor PostgreSQL con permisos de replicación o superusuario.
- Espacio suficiente en disco para almacenar el respaldo.
- Crea el directorio de `respaldos` desde el usuario postgres:
  ```bash
      sudo -i -u postgres
  
      mkdir /var/lib/postgresql/respaldos
  ```
- Crear el directorio `archive` para almacenar los archivos de WALs:
  ```bash
      mkdir /var/lib/postgresql/archive
  ```
  - Editar el archivo /etc/postgresql/16/main/postgresql.conf:
  ```bash
    nano /etc/postgresql/16/main/postgresql.conf
  ```
  - Actualiza los siguientes parametros:
  ```
  wal_level = replica
  archive_mode = on
  archive_command = 'cp %p /var/lib/postgresql/archive/%f'
  ```
- Después de cambiar estos parámetros, reinicia PostgreSQL, desde la cuenta inicial de login.
```bash
sudo systemctl restart postgresql
```
### Pasos para realizar el backup

1.	Ejecutar respaldo físico con compresión en formato tar:
```bash
pg_basebackup -U postgres -D /var/lib/postgresql/respaldos -Ft -z -P
```
Nota: 
- El usuario replicador debe tener permisos para replicación.
- Verificar que se hayan generado archivos comprimidos en /var/lib/postgresql/respaldos.
- Para restaurar, detener el servidor y reemplazar el directorio de datos por el contenido del respaldo descomprimido.
- Despues de realizar la recuperación total, iniciar el servidor y verificar funcionamiento:


### Explicación
El respaldo físico copia todos los archivos de datos y WAL necesarios para restaurar la base de datos en un estado consistente exacto al momento del respaldo.

## Laboratorio 4.3 (Opcional)– Verificar la configuración de archivado WAL
### Objetivo
Verificar la configuración del archivado de Write-Ahead Logs (WAL) para permitir recuperaciones basadas en logs.
### Requisitos
- Acceso con permisos para modificar archivos de configuración.
- Directorio seguro para almacenar WAL archivados.
### Pasos
1.	Editar el archivo postgresql.conf (ubicación típica: /etc/postgresql/16/main/postgresql.conf):
Desde la cuenta postgres, activar archivado WAL:
```bash
    sudo -i -u postgres
    nano /etc/postgresql/16/main/postgresql.conf
```
Cambiar los siguiente parámetros:
```
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /var/lib/postgresql/archive/%f && cp %p /var/lib/postgresql/archive/%f'
```
2.	Reiniciar el servidor PostgreSQL para aplicar cambios, desde la cuenta inicial de login:
```bash
sudo systemctl restart postgresql
```
3.	Verificar que los WAL se estén copiando al directorio especificado.
### Explicación
- El archivado WAL es fundamental para realizar recuperaciones punto en el tiempo y para la replicación. El archive_command define cómo se guardan los logs.
- Si no configuras el archivado de WAL (Write-Ahead Log) en PostgreSQL con los parámetros (wal_level = replica, archive_mode = on, y archive_command), pierdes la capacidad de realizar dos funciones críticas para la gestión de bases de datos: Punto de Recuperación en el Tiempo (PITR) y Replicación de Streaming Asíncrona.

## Laboratorio 4.4 – Recuperación total desde respaldo físico completo

### Objetivo  
Restaurar completamente un servidor PostgreSQL usando un respaldo físico completo generado con `pg_basebackup`, para dejar la base de datos en un estado operativo idéntico al momento del respaldo.

### Requisitos  
- Respaldo físico completo realizado previamente con `pg_basebackup`.  
- Acceso administrativo al servidor donde se restaurará la base.  
- Permisos para detener y arrancar el servicio PostgreSQL.

### Pasos

1. **Detener el servidor PostgreSQL** para permitir restauración:
Desde la cuenta inicial de login ejecutar el comando:
  ```bash
   sudo systemctl stop postgresql
  ```
2.	Eliminar o mover el directorio de datos actual (normalmente /var/lib/postgresql/16/main o según configuración):
```bash
sudo mv /var/lib/postgresql/16/main /var/lib/postgresql/16/main_old
```
Crear el nuevo directorio donde sera restaurado el cluster.
```bash
sudo mkdir /var/lib/postgresql/16/main
```
3.	Descomprimir y copiar el respaldo físico al directorio de datos:
Supongamos que el respaldo está en /var/lib/postgresql/respaldos/base.tar.gz:
```bash
sudo tar -xzf /var/lib/postgresql/respaldos/base.tar.gz -C /var/lib/postgresql/16/main
```
Restaurar los archivos de WAL:
```bash
sudo tar -xzf /var/lib/postgresql/respaldos/pg_wal.tar.gz -C /var/lib/postgresql/16/main/pg_wal
```
4.	Ajustar los permisos y el propietario del directorio de datos para el usuario postgres:
```bash
sudo chmod -R 700 /var/lib/postgresql/16/main
sudo chown -R postgres:postgres /var/lib/postgresql/16/main
```
5.	Iniciar el servidor PostgreSQL:
```bash
sudo systemctl start postgresql
```
6.	Verificar que el servidor esté activo y la base restaurada correctamente:
```bash
sudo systemctl status postgresql
sudo -i -u postgres
psql -U postgres -d ventas
```
```sql
    SELECT * FROM clientes;
```
7.	(Opcional) Limpiar respaldo antiguo si todo está correcto:
```bash
sudo rm -rf /var/lib/postgresql/16/main_old
```
### Explicación
La restauración desde un respaldo físico completo consiste en reemplazar la carpeta de datos con una copia exacta de los archivos del servidor en un estado consistente. Este método es rápido y confiable para recuperaciones totales, pero requiere que el servidor esté apagado durante la operación.

## Laboratorio 4.5 – Recuperación Point-in-Time (PITR)
### Objetivo
Simular un fallo, y usar respaldos base junto con WAL archivados para recuperar la base de datos a un punto específico en el tiempo.
Los datos actuales se pierden porque se sobrescribe la base de datos, como precaución mantener el respaldo y WALs del respaldo total.
### Requisitos
- Respaldo base físico reciente (realizado con pg_basebackup).
- WAL archivados configurados y disponibles.
- Acceso para detener y arrancar el servidor.
### Pasos
1.	Detener el servidor PostgreSQL:
```bash
sudo systemctl stop postgresql
```
2.	Restaurar el respaldo base en el directorio de datos.
3.	Crear un archivo recovery.conf (o modificar postgresql.conf en versiones recientes) con parámetros para la recuperación:
```
restore_command = 'cp /var/lib/postgresql/archive/%f %p'
# recovery_target_time = 'YYYY-MM-DD HH:MM:SS'
recovery_target_time = '2025-08-19 14:30:00'
```
4.	Iniciar el servidor:
```bash
sudo systemctl start postgresql
```
5.	Observar los logs para confirmar que la recuperación se realizó hasta el punto deseado.
6.	Verificar que los datos son consistentes y corresponden a la fecha y hora objetivo.
### Explicación
PITR permite recuperar la base a un momento exacto usando el respaldo base y los WAL archivados. Es una técnica clave para minimizar pérdida de datos tras fallos.


