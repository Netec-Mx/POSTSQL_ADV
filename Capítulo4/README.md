# Capítulo 4: Respaldos y Recuperación

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo3/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo5/)**

---

## Laboratorio 4.1 – Respaldo lógico con `pg_dump` y restauración con `pg_restore`

### Objetivo  
Realizar un respaldo lógico de una base de datos usando `pg_dump` y restaurar ese respaldo en un nuevo esquema o instancia con `pg_restore`.

### Requisitos  
- PostgreSQL instalado y funcionando.  
- Acceso a la base de datos con datos para respaldar.  
- Permisos para crear bases nuevas.

### Pasos

1. Realizar respaldo lógico de la base de datos `basededatos` en formato comprimido personalizado:
   ```bash
   pg_dump -U usuario -F c -f respaldo.bak basededatos
2.	Crear una nueva base de datos para restaurar:
    ```bash
    createdb -U usuario basededatos_restaurada
3.	Restaurar el respaldo usando pg_restore:
    ```bash
    pg_restore -U usuario -d basededatos_restaurada -v respaldo.bak
4.	Conectarse a la base restaurada y verificar tablas y datos:
    ```bash
    psql -U usuario -d basededatos_restaurada
    \dt
    SELECT COUNT(*) FROM tabla_importante;
#### Explicación
El respaldo lógico crea un volcado que contiene comandos para reconstruir la base de datos. El formato personalizado (-F c) permite restaurar objetos selectivamente con pg_restore.

## Laboratorio 4.2 – Respaldo físico con pg_basebackup
### Objetivo
Realizar un respaldo físico completo de PostgreSQL usando pg_basebackup y examinar los archivos generados.
### Requisitos
•	Acceso al servidor PostgreSQL con permisos de replicación o superusuario.
•	Espacio suficiente en disco para almacenar el respaldo.
### Pasos
1.	Ejecutar respaldo físico con compresión en formato tar:
    ```bash
    pg_basebackup -U replicador -D /ruta/respaldo -Ft -z -P
Nota: El usuario replicador debe tener permisos para replicación.
2.	Verificar que se hayan generado archivos comprimidos en /ruta/respaldo.
3.	Para restaurar, detener el servidor y reemplazar el directorio de datos por el contenido del respaldo descomprimido.
4.	Iniciar el servidor y verificar funcionamiento:
    ```bash
    sudo systemctl restart postgresql
#### Explicación
El respaldo físico copia todos los archivos de datos y WAL necesarios para restaurar la base de datos en un estado consistente exacto al momento del respaldo.

## Laboratorio 4.3 – Configuración de archivado WAL
### Objetivo
Configurar el archivado de Write-Ahead Logs (WAL) para permitir recuperaciones basadas en logs.
### Requisitos
•	Acceso con permisos para modificar archivos de configuración.
•	Directorio seguro para almacenar WAL archivados.
### Pasos
1.	Editar el archivo postgresql.conf (ubicación típica: /etc/postgresql/14/main/postgresql.conf):
o	Activar archivado WAL:

    ```bash
    wal_level = replica
    archive_mode = on
    archive_command = 'test ! -f /ruta_archivo/%f && cp %p /ruta_archivo/%f'
2.	Crear el directorio para almacenar WAL y asignar permisos adecuados.
3.	Reiniciar el servidor PostgreSQL para aplicar cambios:
    ```bash
    sudo systemctl restart postgresql
4.	Verificar que los WAL se estén copiando al directorio especificado.
### Explicación
El archivado WAL es fundamental para realizar recuperaciones punto en el tiempo y para la replicación. El archive_command define cómo se guardan los logs.

## Laboratorio 4.4 – Recuperación Point-in-Time (PITR)
### Objetivo
Simular un fallo, y usar respaldos base junto con WAL archivados para recuperar la base de datos a un punto específico en el tiempo.
### Requisitos
•	Respaldo base físico reciente (realizado con pg_basebackup).
•	WAL archivados configurados y disponibles.
•	Acceso para detener y arrancar el servidor.
### Pasos
1.	Detener el servidor PostgreSQL:
    ```bash
    sudo systemctl stop postgresql
2.	Restaurar el respaldo base en el directorio de datos.
3.	Crear un archivo recovery.conf (o modificar postgresql.conf en versiones recientes) con parámetros para la recuperación:
    ```bash
    restore_command = 'cp /ruta_archivo/%f %p'
    recovery_target_time = 'YYYY-MM-DD HH:MM:SS'
4.	Iniciar el servidor:
    ```bash
    sudo systemctl start postgresql
5.	Observar los logs para confirmar que la recuperación se realizó hasta el punto deseado.
6.	Verificar que los datos son consistentes y corresponden a la fecha y hora objetivo.
#### Explicación
PITR permite recuperar la base a un momento exacto usando el respaldo base y los WAL archivados. Es una técnica clave para minimizar pérdida de datos tras fallos.
 
## Laboratorio 4.5 – Recuperación total desde respaldo físico completo

### Objetivo  
Restaurar completamente un servidor PostgreSQL usando un respaldo físico completo generado con `pg_basebackup`, para dejar la base de datos en un estado operativo idéntico al momento del respaldo.

### Requisitos  
- Respaldo físico completo realizado previamente con `pg_basebackup`.  
- Acceso administrativo al servidor donde se restaurará la base.  
- Permisos para detener y arrancar el servicio PostgreSQL.

### Pasos

1. **Detener el servidor PostgreSQL** para permitir restauración:  
   ```bash
   sudo systemctl stop postgresql
2.	Eliminar o mover el directorio de datos actual (normalmente /var/lib/postgresql/14/main o según configuración):
    ```bash
    sudo mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main_old
3.	Descomprimir y copiar el respaldo físico al directorio de datos:
Supongamos que el respaldo está en /ruta/respaldo/base.tar.gz:
    ```bash
    sudo tar -xzf /ruta/respaldo/base.tar.gz -C /var/lib/postgresql/14/main
4.	Ajustar permisos del directorio de datos para el usuario postgres:
    ```bash
    sudo chown -R postgres:postgres /var/lib/postgresql/14/main
    sudo chmod -R 700 /var/lib/postgresql/14/main
5.	Iniciar el servidor PostgreSQL:
    ```bash
    sudo systemctl start postgresql
6.	Verificar que el servidor esté activo y la base restaurada correctamente:
    ```bash
    sudo systemctl status postgresql
    psql -U usuario -d basededatos -c "SELECT COUNT(*) FROM tabla_importante;"
7.	(Opcional) Limpiar respaldo antiguo si todo está correcto:
    ```bash
    sudo rm -rf /var/lib/postgresql/14/main_old
#### Explicación
La restauración desde un respaldo físico completo consiste en reemplazar la carpeta de datos con una copia exacta de los archivos del servidor en un estado consistente. Este método es rápido y confiable para recuperaciones totales, pero requiere que el servidor esté apagado durante la operación.
 
## Laboratorio 4.5 – Recuperación total desde respaldo físico completo (incluyendo WAL)

### Objetivo  
Restaurar completamente un servidor PostgreSQL usando un respaldo físico completo con `pg_basebackup`, asegurando también la recuperación de los archivos WAL para mantener la consistencia y permitir recuperación hasta el último punto válido.

### Requisitos  
- Respaldo físico completo generado con `pg_basebackup`, que incluye archivos de datos y WAL.  
- Acceso administrativo al servidor.  
- Permisos para detener y arrancar PostgreSQL.

### Pasos

1. **Detener el servidor PostgreSQL**:
   ```bash
   sudo systemctl stop postgresql
2.	Mover o eliminar el directorio de datos actual:
    ```bash
    sudo mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main_old
3.	Restaurar el respaldo físico completo (incluyendo WAL):
o	Si el respaldo fue hecho en formato tar comprimido con WAL incluidos, extraer todo al directorio de datos:
    ```bash
    sudo tar -xzf /ruta/respaldo/base_con_wal.tar.gz -C /var/lib/postgresql/14/main
4.	Asegurar que los archivos WAL estén presentes dentro del directorio de datos o en la ubicación configurada para archivado WAL.
o	Si los WAL están en un directorio separado (archivo de archivado), asegurarse que restore_command esté configurado correctamente para recuperarlos durante el arranque.
5.	Configurar permisos adecuados para el directorio de datos:
    ```bash
    sudo chown -R postgres:postgres /var/lib/postgresql/14/main
    sudo chmod -R 700 /var/lib/postgresql/14/main
6.	Iniciar el servidor PostgreSQL:
    ```bash
    sudo systemctl start postgresql
7.	Verificar el estado y la integridad de la base de datos:
    ```bash
    sudo systemctl status postgresql
    psql -U usuario -d basededatos -c "SELECT COUNT(*) FROM tabla_importante;"
#### Explicación
Los archivos WAL contienen el historial de transacciones y cambios que no se reflejan inmediatamente en los archivos base. Restaurar los WAL junto con los archivos de datos es fundamental para asegurar que la base pueda recuperarse hasta el último punto consistente, evitando corrupción o pérdida de datos.

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo3/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo5/)**

---
