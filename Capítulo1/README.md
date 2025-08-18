# Capítulo 1: Introducción a PostgreSQL
---
## Laboratorio 1.1 – Instalación de PostgreSQL en Linux Ubuntu

### Objetivo
Instalar PostgreSQL 16 junto a PostgreSQL 14 en la misma máquina, en un cluster y puerto independientes (p. ej. 5433), para pruebas, desarrollo o migración controlada.

## Descripción
Usaremos el repositorio oficial de PostgreSQL (PGDG) para instalar postgresql-16. Esto crea un cluster nuevo 16/main con su propio directorio de datos y archivos de configuración (separado de 14/main). Ajustaremos el puerto, verificaremos que ambos clusters estén en línea y los agregaremos a pgAdmin.

## Prerrequisitos
-	Ubuntu 22.04 (Jammy) con sudo.
-	Conexión a Internet.
-	PostgreSQL 14 ya instalado y funcionando (normalmente en el puerto 5432).
-	Puerto alterno libre (recomendado 5433).
-	(Opcional) ufw habilitado si expondrás el puerto hacia tu red.
---

## Procedimiento paso a paso

### Paso 1. Verifica tu instalación actual (versión 14)
```
sudo pg_lsclusters
# También:
psql -V
sudo -u postgres psql -p 5432 -c "select version();"
Confirma que 14 esté “online” y en el puerto 5432.
```

### Paso 2. Agrega el repositorio oficial de PostgreSQL (PGDG)
```
sudo apt update
sudo apt install -y curl ca-certificates gnupg lsb-release
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
echo "deb [signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
  | sudo tee /etc/apt/sources.list.d/pgdg.list
sudo apt update
```

### Paso 3. Instala PostgreSQL 16
```
sudo apt install -y postgresql-16 postgresql-client-16

Esto crea el cluster 16/main con su propio directorio de datos en /var/lib/postgresql/16/main y configuración en /etc/postgresql/16/main/.
Nota: Si el puerto 5432 ya está ocupado por 14, a veces 16 se crea automáticamente en 5433. Lo confirmas en el siguiente paso.
```

### Paso 4 (Opcional). Confirma/ajusta el puerto del cluster 16
```
sudo pg_lsclusters

-	Si ves 16 main online port 5433, perfecto.
-	Si 16 quedó en 5432 (o quieres 5433), edita:
sudo nano /etc/postgresql/16/main/postgresql.conf

Busca la línea port = (descoméntala si es necesario) y deja:
port = 5433

Guarda y reinicia solo el cluster 16:
sudo pg_ctlcluster 16 main restart
```

### Paso 5. (Opcional) Abre el puerto en el firewall
```
Si usas ufw y necesitas acceso externo:
sudo ufw allow 5433/tcp
sudo ufw status
```

### Paso 6. Verifica que ambos clusters estén en línea
```
sudo pg_lsclusters
# Pruebas con psql:
sudo -u postgres psql -p 5432 -c "select version();"
sudo -u postgres psql -p 5433 -c "select version();"
Deberías ver 14 respondiendo en 5432 y 16 en 5433.
```

### Paso 7 (Opcional-Verificar). Configura autenticación (si usarás contraseña)
Por defecto en Ubuntu el acceso local suele ser por “peer”. Para usar pgAdmin con password:
1.	Edita pg_hba.conf del 16:
```
sudo nano /etc/postgresql/16/main/pg_hba.conf
Cambia líneas locales a md5 o scram-sha-256 (recomendado si tu password_encryption está en scram-sha-256):
# ejemplo:
local   all             all                                     scram-sha-256
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

2.	Reinicia el cluster 16:
```
sudo pg_ctlcluster 16 main restart
3.	Asigna password al rol postgres (del cluster 16):
sudo -u postgres psql -p 5433 -c "\password postgres"
```

### Paso 8 (Opcional). Agrega el servidor 16 en pgAdmin
```
En pgAdmin: Add New Server…

-	General → Name: PostgreSQL 16 (5433)
-	Connection → Host: localhost
-	Port: 5433
-	Maintenance DB: postgres
-	Username: postgres
-	Password: (la que definiste)

Guarda y prueba la conexión. Repite si quieres dejar también el 14 (5432) en pgAdmin.
```

### Paso 9. Comandos útiles de operación
-	Ver estado de clusters:
-	sudo pg_lsclusters
-	Operar por cluster:
-	sudo pg_ctlcluster 14 main stop
-	sudo pg_ctlcluster 14 main start
-	sudo pg_ctlcluster 16 main restart
-	Servicio global (todos los clusters):
-	sudo systemctl status postgresql
-	sudo systemctl restart postgresql

### Paso 10. Estructura de archivos (para ubicar todo)
-	Configuración 16: /etc/postgresql/16/main/
-	Datos 16: /var/lib/postgresql/16/main/
-	Logs (según config): normalmente /var/log/postgresql/

### Paso 11. (Opcional) Migrar datos de 14 → 16 más adelante
-	Rápido para bases pequeñas: pg_dump/pg_dumpall desde 14 e importas en 16 (5433).
-	Para tiempos cortos de indisponibilidad en bases grandes: pg_upgrade (requiere detener ambos durante el cambio).

### Paso 12. (Opcional) Desinstalar o limpiar
-	Detener y borrar cluster 16 (conserva paquetes):
-	sudo pg_dropcluster 16 main –stop
-	Quitar paquetes 16:
-	sudo apt remove --purge -y postgresql-16 postgresql-client-16
-	sudo apt autoremove -y
 

### Resultado esperado
-	PostgreSQL 14 en 5432 y PostgreSQL 16 en 5433, ambos “online”.
-	Acceso por psql y pgAdmin a cada versión de forma independiente.


## Interpretación de la actualización de llaves del repositorio de PostgreSQL.
Interpretar la siguiente informacion: curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \ | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg echo "deb [signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \ | sudo tee /etc/apt/sources.list.d/pgdg.list y de donde sale toda esta infornacion?

## Interpretación línea por línea
### Línea 1
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
-	curl -fsSL URL → Descarga silenciosamente (-s), muestra errores (-f), y sigue redirecciones (-L) desde la URL.
-	https://www.postgresql.org/media/keys/ACCC4CF8.asc → Es la clave pública oficial GPG de PostgreSQL, usada para verificar la autenticidad de los paquetes.
-	gpg --dearmor → Convierte el formato ASCII de la clave (.asc) a un formato binario (.gpg) que APT puede usar.
-	Salida a /etc/apt/trusted.gpg.d/postgresql.gpg → Guarda la clave en el directorio de claves de confianza para APT.

#### En resumen:
Esta línea descarga y guarda la clave de firma oficial de PostgreSQL para que tu sistema pueda verificar que los paquetes del repositorio no han sido modificados y son legítimos.

### Línea 2
echo "deb [signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg] \
http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
  | sudo tee /etc/apt/sources.list.d/pgdg.list
-	echo "deb ..." → Genera la línea que describe el repositorio.
-	[signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg] → Le dice a APT que los paquetes de este repositorio deben validarse con la clave GPG que descargaste antes.
-	http://apt.postgresql.org/pub/repos/apt → Dirección oficial del repositorio PGDG (PostgreSQL Global Development Group).
-	$(lsb_release -cs) → Comando que devuelve el nombre en clave de tu distribución Ubuntu (por ejemplo, en Ubuntu 22.04 es jammy).
-	-pgdg → Indica que es el repositorio oficial de PostgreSQL, no el de Ubuntu.
-	main → Rama principal del repositorio.
-	sudo tee /etc/apt/sources.list.d/pgdg.list → Guarda esta línea en un archivo de configuración de APT para habilitar el repositorio.

#### En resumen:
Esta línea crea un archivo en /etc/apt/sources.list.d/ con la definición del repositorio oficial de PostgreSQL, usando el nombre de tu versión de Ubuntu para apuntar a la carpeta correcta.

#### Fuente oficial:
https://www.postgresql.org/download/linux/ubuntu/
En esa página, el equipo de PostgreSQL mantiene actualizadas estas instrucciones con la clave GPG más reciente y la URL correcta del repositorio.


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
