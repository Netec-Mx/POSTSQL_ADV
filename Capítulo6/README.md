# Capítulo 6: Seguridad

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo5/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo7/)**

---

## Laboratorio 6.1 – Creación de usuarios con privilegios distintos

### Objetivo  
Crear roles con diferentes niveles de privilegios para controlar el acceso a la base de datos y asignar contraseñas seguras.

### Requisitos  
- Acceso como superusuario a PostgreSQL.  
- Conocimiento básico de roles y permisos.

### Pasos

1. Crear un rol con permisos de solo lectura:  
   ```sql
   CREATE ROLE lector WITH LOGIN PASSWORD 'password_segura';
   GRANT CONNECT ON DATABASE basededatos TO lector;
   GRANT USAGE ON SCHEMA public TO lector;
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO lector;
2.	Crear un rol con permisos de escritura:
    ```sql
    CREATE ROLE escritor WITH LOGIN PASSWORD 'password_segura';
    GRANT CONNECT ON DATABASE basededatos TO escritor;
    GRANT USAGE ON SCHEMA public TO escritor;
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO escritor;
3.	Crear un rol administrador:
    ```sql
    CREATE ROLE admin WITH LOGIN PASSWORD 'password_segura' CREATEDB CREATEROLE;
4.	Verificar los roles creados:
    ```sql
    \du
#### Explicación
La creación de roles con diferentes privilegios permite un control granular del acceso, aumentando la seguridad y reduciendo riesgos.

## Laboratorio 6.2 – Configuración de acceso remoto
### Objetivo
Configurar el archivo pg_hba.conf para permitir conexiones remotas solo desde direcciones IP autorizadas, y probar la conexión.
### Requisitos
•	Acceso al servidor con permisos para modificar archivos de configuración.
•	Dirección IP autorizada para conexión remota.
### Pasos
1.	Editar el archivo pg_hba.conf (ubicación típica: /etc/postgresql/14/main/pg_hba.conf):
Añadir línea para permitir acceso remoto solo desde IP autorizada (ejemplo 192.168.1.100):
    ```nginx
    host    basededatos    all    192.168.1.100/32    md5
2.	Confirmar que el archivo postgresql.conf permite conexiones remotas (listen_addresses debe incluir la IP o '*'):
    ```ini
    listen_addresses = '*'
3.	Reiniciar el servidor PostgreSQL para aplicar cambios:
    ```bash
    sudo systemctl restart postgresql
4.	Desde la máquina remota autorizada, probar conexión:
    ```bash
    psql -h IP_del_servidor -U usuario -d basededatos
5.	Intentar conexión desde una IP no autorizada y confirmar que es rechazada.
#### Explicación
El control en pg_hba.conf limita las conexiones solo a clientes autorizados, evitando accesos no deseados y aumentando la seguridad del sistema.

## Laboratorio 6.3 – Autenticación segura
### Objetivo
Configurar y probar métodos de autenticación seguros como md5 y scram-sha-256 para mejorar la protección de credenciales.
### Requisitos
•	PostgreSQL 10 o superior (que soporte scram-sha-256).
•	Acceso para modificar configuración y reiniciar el servidor.
### Pasos
1.	Verificar el método de autenticación actual en pg_hba.conf. Cambiar a scram-sha-256 para un usuario o rango de IP:
    ```nginx
    host    basededatos    all    192.168.1.0/24    scram-sha-256
2.	Modificar postgresql.conf para que la autenticación SCRAM sea la predeterminada para nuevas contraseñas:
    ```ini
    password_encryption = scram-sha-256
3.	Reiniciar el servidor:
    ```bash
    sudo systemctl restart postgresql
4.	Cambiar la contraseña de un usuario para que use SCRAM:
    ```sql
    ALTER USER usuario WITH PASSWORD 'nueva_password';
5.	Probar conexión con usuario que use md5 y con usuario que use scram-sha-256.
6.	Revisar logs para detectar fallos o problemas de autenticación.
#### Explicación
scram-sha-256 es un método más seguro que md5 para almacenar y validar contraseñas, recomendándose su uso para proteger las credenciales.

## Laboratorio 6.4 – Gestión de permisos sobre objetos
### Objetivo
Gestionar permisos granulares otorgando y revocando privilegios sobre tablas, esquemas y funciones, validando que los usuarios solo acceden a lo autorizado.
### Requisitos
•	Base de datos con varias tablas y funciones.
•	Usuarios creados con distintos roles.
### Pasos
1.	Otorgar permisos de SELECT sobre una tabla específica a un usuario:
    ```sql
    GRANT SELECT ON TABLE clientes TO lector;
2.	Revocar permiso de INSERT a un usuario:
    ```sql
    REVOKE INSERT ON TABLE clientes FROM escritor;
3.	Otorgar ejecución de una función a un rol:
    ```sql
    GRANT EXECUTE ON FUNCTION actualizar_saldo(integer, numeric) TO admin;
4.	Verificar permisos con:
    ```sql
    \dp clientes
5.	Conectarse con diferentes usuarios y probar qué operaciones pueden realizar.
#### Explicación
Controlar permisos sobre objetos específicos permite adaptar el acceso a las necesidades reales, evitando riesgos por accesos indebidos o accidentales.

---

**[⬅️ Atrás](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo5/)** | **[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo7/)**

---
