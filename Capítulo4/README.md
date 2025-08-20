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
sudo -i -u postgres
psql -U postgres
```
### 2. Crear la base de datos ventas
```sql
CREATE DATABASE ventas;

Conéctate a la nueva base de datos:

\c ventas
```
### 3. Crear tablas
Vamos a usar dos tablas: clientes y pedidos.
-- Tabla de clientes
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
SELECT COUNT(*) FROM clientes;
```
### Explicación
El respaldo lógico crea un volcado que contiene comandos para reconstruir la base de datos. El formato personalizado (-F c) permite restaurar objetos selectivamente con pg_restore.

## Laboratorio 4.2 – Respaldo físico con pg_basebackup
### Objetivo
Realizar un respaldo físico completo de PostgreSQL usando pg_basebackup y examinar los archivos generados.
### Requisitos
- Acceso al servidor PostgreSQL con permisos de replicación o superusuario.
- Espacio suficiente en disco para almacenar el respaldo.
- Crea el directorio de respaldos desde el usuario postgres:
  ```bash
      mkdir /var/lib/postgresql/respaldos
  ```
- Crear el siguiente directorio desde el usuario postgres para almacenar los archivos de WALs:
  ```bash
      mkdir /var/lib/postgresql/archive
  ```
  - Editar el archivo /etc/postgresql/14/main/postgresql.conf con los siguientes parametros:
  ```
  wal_level = replica
  archive_mode = on
  archive_command = 'cp %p /var/lib/postgresql/archive/%f'
  ```
- Después de cambiar estos parámetros, reinicia PostgreSQL:
Desde la cuenta inicial de login, ejecutar el comando:
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
- Iniciar el servidor y verificar funcionamiento:
```bash
sudo systemctl restart postgresql
```
### Explicación
El respaldo físico copia todos los archivos de datos y WAL necesarios para restaurar la base de datos en un estado consistente exacto al momento del respaldo.

## Laboratorio 4.3 (Opcional)– Verificar la configuración de archivado WAL
### Objetivo
Verificar la configuración del archivado de Write-Ahead Logs (WAL) para permitir recuperaciones basadas en logs.
### Requisitos
- Acceso con permisos para modificar archivos de configuración.
- Directorio seguro para almacenar WAL archivados.
### Pasos
1.	Editar el archivo postgresql.conf (ubicación típica: /etc/postgresql/14/main/postgresql.conf):
Desde la cuenta postgres, activar archivado WAL:
```bash
    sudo -i -u postgres
    nano /etc/postgresql/14/main/postgresql.conf
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
El archivado WAL es fundamental para realizar recuperaciones punto en el tiempo y para la replicación. El archive_command define cómo se guardan los logs.

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
2.	Eliminar o mover el directorio de datos actual (normalmente /var/lib/postgresql/14/main o según configuración):
```bash
sudo mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main_old
sudo mkdir /var/lib/postgresql/14/main
```
3.	Descomprimir y copiar el respaldo físico al directorio de datos:
Supongamos que el respaldo está en /ruta/respaldo/base.tar.gz:
```bash
sudo tar -xzf /var/lib/postgresql/respaldos/base.tar.gz -C /var/lib/postgresql/14/main

Restaurar los archivos de WAL:
sudo tar -xzf /var/lib/postgresql/respaldos/pg_wal.tar.gz -C /var/lib/postgresql/14/main/pg_wal
```
4.	Ajustar permisos del directorio de datos para el usuario postgres:
```bash
sudo chown -R postgres:postgres /var/lib/postgresql/14/main
sudo chmod -R 700 /var/lib/postgresql/14/main
```
5.	Iniciar el servidor PostgreSQL:
```bash
sudo systemctl start postgresql
```
6.	Verificar que el servidor esté activo y la base restaurada correctamente:
```bash
sudo systemctl status postgresql
sudo -i -u postgres
```
```bash
psql -U postgres -d ventas
```
```sql
    SELECT * FROM clientes;
```
7.	(Opcional) Limpiar respaldo antiguo si todo está correcto:
```bash
sudo rm -rf /var/lib/postgresql/14/main_old
```
### Explicación
La restauración desde un respaldo físico completo consiste en reemplazar la carpeta de datos con una copia exacta de los archivos del servidor en un estado consistente. Este método es rápido y confiable para recuperaciones totales, pero requiere que el servidor esté apagado durante la operación.

## Laboratorio 4.5 – Recuperación Point-in-Time (PITR)
### Objetivo
Simular un fallo, y usar respaldos base junto con WAL archivados para recuperar la base de datos a un punto específico en el tiempo.
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
recovery_target_time = 'YYYY-MM-DD HH:MM:SS'
```
4.	Iniciar el servidor:
```bash
sudo systemctl start postgresql
```
5.	Observar los logs para confirmar que la recuperación se realizó hasta el punto deseado.
6.	Verificar que los datos son consistentes y corresponden a la fecha y hora objetivo.
### Explicación
PITR permite recuperar la base a un momento exacto usando el respaldo base y los WAL archivados. Es una técnica clave para minimizar pérdida de datos tras fallos.

## Laboratorio 4.6 – Recuperación total desde respaldo físico completo (incluyendo WAL)

### Objetivo  
Restaurar completamente un servidor PostgreSQL usando un respaldo físico completo con `pg_basebackup`, asegurando también la recuperación de los archivos WAL para mantener la consistencia y permitir recuperación hasta el último punto válido.

### Requisitos  
- Respaldo físico completo generado con `pg_basebackup`, que incluye archivos de datos y WAL.  
- Acceso administrativo al servidor.  
- Permisos para detener y arrancar PostgreSQL.

### Pasos
Los siguiet¿ntes pasos se deben realizar desde la cuenta de login inicial (netec).

1. **Detener el servidor PostgreSQL**:
```bash
   sudo systemctl stop postgresql
```
2.	Mover o eliminar el directorio de datos actual:
```bash
sudo mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main_old
```
3.	Restaurar el respaldo físico completo (incluyendo WAL):
-	Si el respaldo fue hecho en formato tar comprimido con WAL incluidos, extraer todo al directorio de datos:
```bash
sudo tar -xzf /var/lib/postgresql/respaldos/pg_wal.tar.gz -C /var/lib/postgresql/14/main/pg_wal
```
4.	Asegurar que los archivos WAL estén presentes dentro del directorio de datos o en la ubicación configurada para archivado WAL.
-	Si los WAL están en un directorio separado (archivo de archivado), asegurarse que restore_command esté configurado correctamente para recuperarlos durante el arranque.
5.	Configurar permisos adecuados para el directorio de datos:
```bash
sudo chown -R postgres:postgres /var/lib/postgresql/14/main
sudo chmod -R 700 /var/lib/postgresql/14/main
```
6.	Iniciar el servidor PostgreSQL:
```bash
sudo systemctl start postgresql
```
7.	Verificar el estado y la integridad de la base de datos:
```bash
sudo systemctl status postgresql
psql -U usuario -d basededatos -c "SELECT COUNT(*) FROM tabla_importante;"
```
### Explicación
Los archivos WAL contienen el historial de transacciones y cambios que no se reflejan inmediatamente en los archivos base. Restaurar los WAL junto con los archivos de datos es fundamental para asegurar que la base pueda recuperarse hasta el último punto consistente, evitando corrupción o pérdida de datos.

## Laboratorio 4.7 – Instalación y configuración de Barman
### Objetivo
Instalar Barman y preparar conexión segura con un servidor PostgreSQL.
### Requisitos
-	Servidor Barman en una máquina distinta (solo si va a ser remoto).
-	Acceso SSH y credenciales (solo si va a ser remoto).
### Pasos
1.	Instalar Barman:
```bash
apt install barman barman-cli
```
2.	Crear usuario de replicación en PostgreSQL:
```sql
CREATE ROLE barman WITH REPLICATION LOGIN PASSWORD 'segura';
```
3.	Configurar pg_hba.conf:
```bash
host replication barman 192.168.1.50/32 md5
```
4.	Copiar clave SSH a servidor PostgreSQL (solo si va a ser remoto):
```bash
ssh-copy-id barman@192.168.1.20
```
5.	Configurar /etc/barman.conf:
```bash
[produccion]
description = "Servidor Producción"
conninfo = host=192.168.1.20 user=barman dbname=postgres
backup_method = postgres
streaming_archiver = on
archiver = on
```
6.	Verificar conexión:
```bash
barman check produccion
```
## Laboratorio 4.8 – Respaldo local (o remoto) y restauración con Barman
Objetivo
Ejecutar un respaldo remoto centralizado y restaurarlo en un servidor alterno.
Requisitos
-	 4.7 completada.
Pasos
1.	Realizar respaldo:
```bash
barman backup producción
```
2.	Listar respaldos:
```bash
barman list-backup produccion
3.	Restaurar último respaldo en otra ruta:
```bash
barman recover produccion latest /var/lib/postgresql/restauración
```
Verificación
-	Iniciar PostgreSQL en la ruta de restauración y validar datos.


---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo3/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo5/)**

---
