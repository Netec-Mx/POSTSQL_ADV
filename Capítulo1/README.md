# Práctica 1. Instalación de PostgreSQL en Linux

---

**[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo2/)**

---

## Tarea 1. Instalación de PostgreSQL en Linux Ubuntu.

### 🎯 Objetivos:
Al finalizar la práctica, serás capaz de:
- Instalar PostgreSQL 16 y PostgreSQL 14 en la misma máquina, cada uno en un clúster y un puerto independientes (por ejemplo, 5433), para pruebas, desarrollo o migración controlada.

### 📝 Descripción:
Usaremos el repositorio oficial de PostgreSQL (PGDG) para instalar PostgreSQL 16.
Este proceso crea un clúster nuevo 16/main con su propio directorio de datos y archivos de configuración, independiente de 14/main.

### 🛠️ Prerrequisitos:
-	Ubuntu 22.04 (Jammy) con privilegios de **sudo**.
-	Conexión a Internet.
-	PostgreSQL 14 instalado y en funcionamiento (normalmente en el puerto 5432).
-	Un puerto alternativo libre (se recomienda el 5433).
-	**Opcional:** ufw habilitado si se expondrá el puerto hacia la red.

### 📋 Instrucciones:

**Paso 1.** Verifica la instalación actual (versión 14).

```
sudo pg_lsclusters

# También:
sudo psql -V
sudo -i -u postgres psql -p 5432 -c "select version();"

Confirma que 14 esté “online” y en el puerto 5432.
```

**Paso 2.** Agrega el repositorio oficial de PostgreSQL (PGDG)

```
sudo apt update
sudo apt install -y curl ca-certificates gnupg lsb-release
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
echo "deb [signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
  | sudo tee /etc/apt/sources.list.d/pgdg.list
sudo apt update
```

**Paso 3.** Instala PostgreSQL 16.

```
sudo apt install -y postgresql-16 postgresql-client-16
```

Este comando crea el clúster 16/main, con su propio directorio de datos en:

- `/var/lib/postgresql/16/main`
- `/etc/postgresql/16/main/`

> *💡 **Nota:** si el puerto 5432 ya está en uso por la versión 14, en algunos casos la versión 16 se crea automáticamente en el puerto 5433. Esto se confirmará en el siguiente paso.*

**Paso 4 (opcional).** Confirmar o ajustar el puerto del clúster 16.

```
sudo pg_lsclusters
```

- Si aparece 16 main online port 5433, no es necesario hacer cambios.
- Si el clúster 16 quedó en el puerto 5432 (o prefieres usar el 5433), edita el archivo de configuración:

```
sudo nano /etc/postgresql/16/main/postgresql.conf
```

Busca la línea correspondiente a port (descoméntala si es necesario) y déjala así:

```
port = 5433
```

Guarda los cambios y reinicia únicamente el clúster 16:

```
sudo pg_ctlcluster 16 main restart
```

**Paso 5 (opcional)** Abre el puerto en el firewall.

Si utilizas ufw y necesitas habilitar el acceso externo:

```
sudo ufw allow 5433/tcp
sudo ufw status
```

**Paso 6.** Verificar que ambos clústeres estén en línea.

```
sudo pg_lsclusters
```

Para probar las conexiones con cada versión:

```
sudo -u postgres psql -p 5432 -c "select version();"
sudo -u postgres psql -p 5433 -c "select version();"
```

Deberías obtener como resultado:

- PostgreSQL 14 respondiendo en el puerto 5432.
- PostgreSQL 16 respondiendo en el puerto 5433.

**Paso 7** (opcional – verificación). Configurar autenticación con contraseña.

En Ubuntu, el acceso local suele estar configurado con el método peer.
Si deseas conectarte con **pgAdmin** u otras herramientas que requieren contraseña, realiza lo siguiente:

a) Edita el archivo `pg_hba.conf` del clúster 16:

```
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

b) Modifica las líneas de autenticación para usar md5 o scram-sha-256 (este último es el recomendado si password_encryption está en scram-sha-256):

```
# ejemplo:
local   all             all                                     scram-sha-256
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

c)	Guarda los cambios y reinicia el clúster:

```
sudo pg_ctlcluster 16 main restart
```

d).	Asigna contraseña al rol postgres (del clúster 16):

```
sudo -u postgres psql -p 5433 -c "\password postgres"
```

**Paso 8 (opcional).** Agrega el servidor 16 en pgAdmin.

En pgAdmin, selecciona **Add New Server…*.

-	**General → Name:** `PostgreSQL 16 (5433)`
-	**Connection → Host:** `localhost`
-	**Port:** `5433`
-	**Maintenance DB:** postgres`
-	**Username:** `postgres`
-	**Password:** (la contraseña que definiste)

Haz clic en **Save** y prueba la conexión.

Si lo deseas, repite el procedimiento para agregar también el servidor 14 (5432).```

**Paso 9.** Comandos útiles de operación.

-	Ver estado de clusters:

```
sudo pg_lsclusters
```

-	Operar por clúster:
  
```
sudo pg_ctlcluster 14 main stop
sudo pg_ctlcluster 14 main start
sudo pg_ctlcluster 16 main restart
```

-	Servicio global (todos los clústers):

```
sudo systemctl status postgresql
sudo systemctl restart postgresql
```

**Paso 10.** Estructura de archivos (para que ubiques todo)

-	Configuración 16:

```
/etc/postgresql/16/main/
```

-	Datos 16:

```
/var/lib/postgresql/16/main/
```

-	Logs (según su confguración): Normalmente en:

```
/var/log/postgresql/
```

**Paso 11 (opcional).** Migrar datos de 14 a 16 más adelante.

- Para bases pequeñas (rápido): usa `pg_dump` o `pg_dumpall` en el servidor 14 e importa en el 16 (puerto 5433).
-	Para bases grandes (con mínima indisponibilidad): utiliza `pg_upgrade`. Ten en cuenta que este método requiere detener ambos clusters durante la migración.

**Paso 12 (opcional).** Desinstalar o limpiar.

-	Detén y borra el cluster 16 (se conservan los paquetes):

```
sudo pg_dropcluster 16 main –stop
```

-	Quita los paquetes de PostgreSQL 16:

```
sudo apt remove --purge -y postgresql-16 postgresql-client-16
sudo apt autoremove -y
 ```

### Resultado esperado
- Tienes PostgreSQL 14 en el puerto 5432 y PostgreSQL 16 en el puerto 5433, ambos en línea.
- Puedes acceder a cada versión de forma independiente usando psql y pgAdmin.

---

### Interpretación de la actualización de llaves del repositorio de PostgreSQL

Interpreta la siguiente informacion: 

```
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg

echo "deb [signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
  | sudo tee /etc/apt/sources.list.d/pgdg.list
```

**¿De dónde proviene toda esta información?**

### Interpretación línea por línea

#### Línea 1

```
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
```

-	`curl -fsSL URL` → Descarga silenciosamente (`-s`), muestra errores (`-f`), y sigue redirecciones (`-L`) desde la URL.
-	`https://www.postgresql.org/media/keys/ACCC4CF8.asc` → Es la clave pública oficial GPG de PostgreSQL, usada para verificar la autenticidad de los paquetes.
-	`gpg --dearmor` → Convierte el formato ASCII de la clave (`.asc`) a un formato binario (`.gpg`) que APT puede usar.
-	Salida a `/etc/apt/trusted.gpg.d/postgresql.gpg` → Guarda la clave en el directorio de claves de confianza para APT.

**En resumen:**
Esta línea descarga y guarda la clave de firma oficial de PostgreSQL, para que tu sistema pueda verificar que los paquetes del repositorio no han sido modificados y sean legítimos.

#### Línea 2

```
echo "deb [signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg] \
http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
  | sudo tee /etc/apt/sources.list.d/pgdg.list
```

-	`echo "deb ..."` → Genera la línea que describe el repositorio.
-	`[signed-by=/etc/apt/trusted.gpg.d/postgresql.gpg]` → Le dice a APT que los paquetes de este repositorio deben validarse con la clave GPG que descargaste antes.
-	`http://apt.postgresql.org/pub/repos/apt` → Dirección oficial del repositorio PGDG (PostgreSQL Global Development Group).
-	`$(lsb_release -cs)` → Comando que devuelve el nombre en clave de tu distribución Ubuntu (por ejemplo, en Ubuntu 22.04 es jammy).
-	`-pgdg` → Indica que es el repositorio oficial de PostgreSQL, no el de Ubuntu.
-	`main` → Rama principal del repositorio.
-	`sudo tee /etc/apt/sources.list.d/pgdg.list` → Guarda esta línea en un archivo de configuración de APT para habilitar el repositorio.

#### En resumen:
Esta línea crea un archivo en /etc/apt/sources.list.d/ con la definición del repositorio oficial de PostgreSQL, usando el nombre de tu versión de Ubuntu para apuntar a la carpeta correcta.

#### Fuente oficial:

- [https://www.postgresql.org/download/linux/ubuntu/](https://www.postgresql.org/download/linux/ubuntu/)

En esa página, el equipo de PostgreSQL mantiene actualizadas estas instrucciones con la clave GPG más reciente y la URL correcta del repositorio.

---

## Tarea 1.2. Uso básico del cliente psql

### 🎯 Objetivos:
Al finalizar la práctica, serás capaz de:
- Familiarizarte con el cliente de línea de comandos psql para ejecutar comandos y explorar la base de datos.

### 🛠️ Prerrequisitos:
- PostgreSQL instalado y servicio activo.
- Acceso a usuario postgres o con permisos equivalentes.

### Instrucciones:

**Paso 1.** Accede a **psql** como usuario **postgres**:
```bash
sudo -i -u postgres
psql
```

**Paso 2.**	Lista todas las bases de datos:

```sql
\l
```

**Paso 3.**	Conéctate a una base de datos específica (ejemplo: postgres):

```sql
\c postgres
```
    
**Paso 4.**	Lista todas las tablas en la base de datos actual:

```sql
\dt
```

**Paso 5.**	Lista todos los roles de usuarios:

```sql
\du
```

**Paso 6.**	Muestra ayuda para los metacomandos:

```sql
\?
```

**Paso 7.**	Sal de psql:

```sql
\q
```

### Explicación:
El cliente psql permite administrar y consultar PostgreSQL de forma interactiva. Los comandos con barra invertida (`\`) son _metacomandos_ internos que facilitan la navegación y la gestión.

---

## Tarea 3. Consulta de la arquitectura del servidor

### 🎯 Objetivos:
Al finalizar la práctica, serás capaz de:
- Explorar la arquitectura de PostgreSQL y localizar los archivos de configuración y los directorios de datos.

### 🛠️ Prerrequisitos:
- PostgreSQL instalado y servicio activo.
- Acceso a usuario con permisos para consultar vistas del sistema.

### Instrucciones:

**Paso 1.**	Accede a psql como usuario _postgres_:

```bash
sudo -i -u postgres
psql
```

**Paso 2.**	Consulta la ubicación del directorio de datos:

```sql
SHOW data_directory;
```

**Paso 3.**	Consulta el archivo principal de configuración:
```sql
SHOW config_file;
```
**Paso 4.**	Consulta el archivo de control de accesos (pg_hba.conf):

```sql
SHOW hba_file;
```

**Paso 5.**	Lista los procesos del servidor:

```sql
SELECT * FROM pg_stat_activity;
```

**Paso 6.**	Sal de psql:

```sql
\q
```

**Paso 7.**	En el sistema operativo, navega al directorio de datos para confirmar existencia:

```bash
cd $(psql -U postgres -t -c "SHOW data_directory")
ls -l
```

### Explicación:
Estos pasos permiten identificar la estructura de PostgreSQL, archivos críticos de configuración y visualizar la actividad interna del servidor para entender su funcionamiento.

---

## Tarea 4. Conexión remota usando psql

### 🎯 Objetivos:
Al finalizar la práctica, serás capaz de:
- Configurar PostgreSQL para aceptar conexiones remotas y establecer una conexión desde un equipo cliente.

### 🛠️ Prerrequisitos:
- Servidor PostgreSQL instalado y accesible en red.
- Acceso administrativo para modificar archivos de configuración.
- Equipo cliente con psql instalado.

### Instrucciones:
**Paso 1.**	Edita el archivo postgresql.conf para permitir conexiones remotas.

Ubícalo y modifícalo (usualmente en /etc/postgresql/14/main/postgresql.conf o similar):

```ini
listen_addresses = '*'
```
**Paso 2.**	Edita `pg_hba.conf` para permitir conexiones desde la IP del cliente (ejemplo para la red `192.168.1.0/24`):
    
```css
host    all             all             192.168.1.0/24           md5
```

**Paso 3.**	Reinicia el servicio PostgreSQL para aplicar los cambios:

```bash
sudo systemctl restart postgresql
```

**Paso 4.**	Crea un usuario con contraseña para conexión remota:

```bash
sudo -i -u postgres
psql
CREATE ROLE remoto LOGIN PASSWORD 'tu_contraseña';
GRANT ALL PRIVILEGES ON DATABASE postgres TO remoto;
\q
```

**Paso 5.**	Desde el equipo cliente, conéctate con:

```bash
psql -h <IP_SERVIDOR> -U remoto -d postgres
```

**Paso 6.**	Verifica la conexión y ejecuta consultas básicas.

### Explicación
PostgreSQL por defecto escucha sólo conexiones locales. Cambiar listen_addresses y ajustar pg_hba.conf habilitan el acceso remoto seguro con autenticación. Se debe crear un usuario con permiso para acceder desde red externa.

---

**[Lista general 🗂️](https://netec-mx.github.io/POSTSQL_ADV/)** | **[Siguiente ➡️](https://netec-mx.github.io/POSTSQL_ADV/Cap%C3%ADtulo2/)**

---
