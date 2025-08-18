# Capítulo 7: Configuración

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo6/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo8/)**

---

## Laboratorio 7.1 – Modificación de parámetros en `postgresql.conf`

### Objetivo  
Modificar parámetros clave de memoria y concurrencia para optimizar el rendimiento de PostgreSQL.

### Requisitos  
- Acceso para editar archivos de configuración.  
- Permisos para reiniciar el servicio PostgreSQL.

### Pasos

1. Editar el archivo `postgresql.conf` (ubicación típica: `/etc/postgresql/14/main/postgresql.conf`).

2. Cambiar los parámetros siguientes para ajustar memoria y concurrencia:

    ```
    shared_buffers = 1GB
    work_mem = 64MB
    max_connections = 200
    ```
3. Guardar los cambios y reiniciar PostgreSQL para que tomen efecto:
    ```bash
    sudo systemctl restart postgresql
4.	Verificar que los parámetros se aplicaron correctamente desde la consola SQL:
    ```sql
    SHOW shared_buffers;
    SHOW work_mem;
    SHOW max_connections;
#### Explicación
Ajustar estos parámetros es fundamental para que PostgreSQL utilice adecuadamente los recursos del servidor y pueda manejar más conexiones y consultas simultáneas.

## Laboratorio 7.2 – Configuración y análisis de logs
### Objetivo
Configurar logging detallado para monitorear consultas y analizar el comportamiento de la base de datos.
### Requisitos
•	Acceso para modificar postgresql.conf.
•	Permisos para reiniciar el servidor.
### Pasos
1.	Modificar postgresql.conf para activar logging detallado:
    ```ini
    log_statement = 'all'
    log_duration = on
    log_directory = 'pg_log'
    log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
2.	Reiniciar PostgreSQL:
    ```bash
    sudo systemctl restart postgresql
3.	Realizar algunas consultas desde psql o aplicaciones conectadas.
4.	Revisar los archivos de log generados en el directorio configurado:
    ```bash
    tail -f /var/lib/postgresql/14/main/pg_log/postgresql-*.log
5.	Analizar los logs para identificar consultas y tiempos de ejecución.
#### Explicación
El logging detallado permite auditar la actividad, diagnosticar problemas y optimizar consultas, facilitando la administración.

## Laboratorio 7.3 – Administración de recursos
### Objetivo
Monitorear conexiones activas y ajustar parámetros para limitar recursos y evitar sobrecarga.
### Requisitos
•	Acceso a consola SQL.
•	Permisos para modificar parámetros y reiniciar.
### Pasos
1.	Consultar conexiones activas con:
    ```sql
    SELECT pid, usename, application_name, client_addr, state FROM pg_stat_activity;
2.	Identificar conexiones inactivas o bloqueadas.
3.	Ajustar en postgresql.conf parámetros para limitar conexiones y recursos:
    ```ini
    max_connections = 150
    superuser_reserved_connections = 3
4.	Reiniciar el servidor y verificar el efecto.
5.	Opcionalmente, usar herramientas externas o comandos OS para monitorear uso de CPU y memoria.
#### Explicación
La administración proactiva de recursos evita cuellos de botella y mantiene la base de datos estable bajo cargas variables.

## Laboratorio 7.4 – Ajuste de parámetros de seguridad
### Objetivo
Configurar parámetros relacionados con seguridad como SSL y encriptación de contraseñas para proteger las conexiones y datos.
### Requisitos
•	Acceso para modificar configuración y reiniciar PostgreSQL.
•	Certificados SSL (auto-firmados o emitidos por CA).
### Pasos
1.	Configurar SSL en postgresql.conf:
    ```ini
    ssl = on
    ssl_cert_file = '/ruta/certificado/server.crt'
    ssl_key_file = '/ruta/llave/server.key'
2.	Configurar encriptación de contraseñas:
    ```ini
    password_encryption = scram-sha-256
3.	Reiniciar PostgreSQL:
    ```bash
    sudo systemctl restart postgresql
4.	Probar conexiones SSL con psql:
    ```bash
    psql "host=localhost port=5432 sslmode=require dbname=basededatos user=usuario"
5.	Verificar que la conexión esté cifrada y que la autenticación use el método configurado.
#### Explicación
Implementar SSL y métodos seguros de autenticación protege la confidencialidad e integridad de la información en tránsito y las credenciales de acceso.

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo6/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo8/)**

---
