# Capítulo 1: Introducción a PostgreSQL
---
## Laboratorio 1.1 – Instalación de PostgreSQL v14 en Ubuntu

### Objetivo  
Instalar y configurar PostgreSQL versión 14 en Ubuntu, verificar el correcto funcionamiento del servidor y preparar el entorno para su uso básico.

### Requisitos  
- Sistema operativo Ubuntu 20.04 o superior.  
- Usuario con privilegios sudo.  
- Acceso a internet.

### Pasos
1. Actualizar repositorios y sistema:
    ```bash
   sudo apt update
   sudo apt upgrade -y
    ```
2.	Agregar repositorio oficial PostgreSQL:
    ```bash
    wget -qO - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    sudo apt update
    ```
3.	Instalar PostgreSQL 14:
    ```bash
    sudo apt install postgresql-14 -y
    ```
4.	Verificar estado del servicio:
    ```bash
    sudo systemctl status postgresql
    ```
5.	Acceder al prompt de PostgreSQL como usuario postgres:
    ```bash
    sudo -i -u postgres
    psql
    ```
6.	Verificar versión en psql:
    ```sql
    SELECT version();
    ```
7.	Salir de psql y cerrar sesión postgres:
    ```sql
    \q
    exit
    ```
#### Explicación
Este laboratorio cubre la instalación desde repositorios oficiales para garantizar estabilidad y actualizaciones. Se verifica el servicio y se accede con el usuario administrador predeterminado.
 
## Laboratorio 1.2 – Uso básico del cliente psql
### Objetivo
Familiarizarse con el cliente de línea de comandos psql para ejecutar comandos y explorar la base de datos.
### Requisitos
- PostgreSQL instalado y servicio activo.
- Acceso a usuario postgres o con permisos equivalentes.
### Pasos
1.	Acceder a psql como usuario postgres:
    ```bash
    sudo -i -u postgres
    psql
    ```
2.	Listar todas las bases de datos:
    ```sql
    \l
    ```
3.	Conectarse a una base de datos específica (ejemplo: postgres):
    ```sql
    \c postgres
    ```
4.	Listar todas las tablas en la base de datos actual:
    ```sql
    \dt
    ```
5.	Listar todos los roles de usuarios:
    ```sql
    \du
    ```
6.	Mostrar ayuda para comandos meta:
    ```sql
    \?
    ```
7.	Salir de psql:
    ```sql
    \q
    ```
#### Explicación
El cliente psql permite administrar y consultar PostgreSQL de forma interactiva. Los comandos con barra invertida (\) son meta comandos internos que facilitan la navegación y gestión.

## Laboratorio 1.3 – Consulta de la arquitectura del servidor
### Objetivo
Explorar la arquitectura de PostgreSQL y localizar archivos de configuración y directorios de datos.
### Requisitos
- PostgreSQL instalado y servicio activo.
- Acceso a usuario con permisos para consultar vistas del sistema.
### Pasos
1.	Acceder a psql como usuario postgres:
    ```bash
    sudo -i -u postgres
    psql
    ```
2.	Consultar la ubicación del directorio de datos:
    ```sql
    SHOW data_directory;
    ```
3.	Consultar el archivo principal de configuración:
    ```sql
    SHOW config_file;
    ```
4.	Consultar archivo de control de accesos (pg_hba.conf):
    ```sql
    SHOW hba_file;
    ```
5.	Listar los procesos del servidor:
    ```sql
    SELECT * FROM pg_stat_activity;
    ```
6.	Salir de psql:
    ```sql
    \q
    ```
7.	En el sistema operativo, navegar al directorio de datos para confirmar existencia:
    ```bash
    cd $(psql -U postgres -t -c "SHOW data_directory")
    ls -l
    ```
#### Explicación
Estos pasos permiten identificar la estructura de PostgreSQL, archivos críticos de configuración y visualizar la actividad interna del servidor para entender su funcionamiento.

## Laboratorio 1.4 – Conexión remota usando psql
### Objetivo
Configurar PostgreSQL para aceptar conexiones remotas y establecer una conexión desde un equipo cliente.
### Requisitos
- Servidor PostgreSQL instalado y accesible en red.
- Acceso administrativo para modificar archivos de configuración.
- Equipo cliente con psql instalado.
### Pasos
1.	Editar el archivo postgresql.conf para permitir conexiones remotas:
Ubicar y modificar (usualmente en /etc/postgresql/14/main/postgresql.conf o similar):
    ```ini
    listen_addresses = '*'
    ```
2.	Editar pg_hba.conf para permitir conexiones desde la IP del cliente (ejemplo para red 192.168.1.0/24):
    ```css
    host    all             all             192.168.1.0/24           md5
    ```
3.	Reiniciar el servicio PostgreSQL para aplicar cambios:
    ```bash
    sudo systemctl restart postgresql
    ```
4.	Crear un usuario con contraseña para conexión remota:
    ```bash
    sudo -i -u postgres
    psql
    CREATE ROLE remoto LOGIN PASSWORD 'tu_contraseña';
    GRANT ALL PRIVILEGES ON DATABASE postgres TO remoto;
    \q
    ```
5.	Desde el equipo cliente, conectar con:
    ```bash
    psql -h <IP_SERVIDOR> -U remoto -d postgres
    ```
6.	Verificar conexión y ejecutar consultas básicas.
#### Explicación
PostgreSQL por defecto escucha sólo conexiones locales. Cambiar listen_addresses y ajustar pg_hba.conf habilitan el acceso remoto seguro con autenticación. Se debe crear un usuario con permiso para acceder desde red externa.

Copywrite 2025
