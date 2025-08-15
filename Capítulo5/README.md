# Capítulo 5: Replicación y Alta Disponibilidad

---

## Laboratorio 5.1 – Configuración de replicación asíncrona

### Objetivo  
Configurar un entorno básico de replicación asíncrona entre un servidor primario y un secundario para mantener sincronizados los datos.

### Requisitos  
- Dos servidores con PostgreSQL instalado (pueden ser máquinas físicas o virtuales).  
- Usuario con permisos de replicación creado en el primario.  
- Conexión de red entre ambos servidores.

### Pasos

1. **En el servidor primario:**

   - Editar `postgresql.conf` para habilitar la replicación:
     ```
     wal_level = replica
     max_wal_senders = 5
     wal_keep_size = 16MB
     ```
   
   - Configurar el archivo `pg_hba.conf` para permitir conexión de replicación desde el secundario:
     ```
     host replication replicador IP_secundario/32 md5
     ```

   - Reiniciar el servidor primario:
     ```bash
     sudo systemctl restart postgresql
     ```

2. **Crear usuario replicador:**
   ```sql
   CREATE ROLE replicador WITH REPLICATION LOGIN ENCRYPTED PASSWORD 'tu_password';
3.	En el servidor secundario:
o	Hacer una copia base usando pg_basebackup:
    ```bash
    pg_basebackup -h IP_primario -D /var/lib/postgresql/14/main -U replicador -v -P --wal-method=stream
- Crear archivo standby.signal en el directorio de datos para activar modo standby:
    ```bash
    touch /var/lib/postgresql/14/main/standby.signal
- Configurar postgresql.conf en el secundario para conexión al primario:
    ```ini
    primary_conninfo = 'host=IP_primario port=5432 user=replicador password=tu_password'
4.	Iniciar servidor secundario:
    ```bash
    sudo systemctl start postgresql
5.	Verificar estado de la replicación en el primario:
    ```sql
    SELECT * FROM pg_stat_replication;
#### Explicación
La replicación asíncrona permite que el servidor secundario reciba los cambios del primario con cierto retraso, ofreciendo alta disponibilidad con mínima latencia en la operación del primario.

## Laboratorio 5.2 – Promoción de servidor esclavo
### Objetivo
Promover un servidor secundario en modo standby a servidor primario para recuperación rápida en caso de fallo.
### Requisitos
•	Laboratorio 5.1 completado y replicación funcionando.
•	Acceso al servidor secundario.
### Pasos
1.	En el servidor secundario, detener el servicio:
    ```bash
    sudo systemctl stop postgresql
2.	Ejecutar el comando de promoción:
    ```bash
    pg_ctl -D /var/lib/postgresql/14/main promote
3.	Verificar que el servidor ahora acepta conexiones y es primario:
    ```bash
    psql -c "SELECT pg_is_in_recovery();"
- Resultado debe ser false.
4.	Verificar creación de nuevos archivos de control y que la base está operativa.
#### Explicación
Promover el secundario permite mantener la disponibilidad del servicio ante fallos del primario, facilitando un failover manual.

## Laboratorio 5.3 – Configuración de replicación síncrona
### Objetivo
Configurar la replicación para que sea síncrona, garantizando que las transacciones se confirmen solo cuando el secundario haya recibido los cambios.
### Requisitos
•	Laboratorio 5.1 completado.
### Pasos
1.	En el servidor primario, modificar postgresql.conf:
    ```ini
    synchronous_standby_names = '*'
    synchronous_commit = on
2.	Reiniciar el servidor primario:
    ```bash
    sudo systemctl restart postgresql
3.	Verificar en el primario la configuración:
    ```sql
    SHOW synchronous_standby_names;
4.	Insertar datos en primario y observar latencia:
    ```sql
    INSERT INTO tabla VALUES (...);
5.	Monitorear replicación y confirmar que la transacción espera a que el secundario confirme antes de finalizar.
#### Explicación
La replicación síncrona aumenta la seguridad de los datos al garantizar que no se pierde información en caso de fallo, aunque puede aumentar la latencia en transacciones.

## Laboratorio 5.4 – Simulación de failover manual
### Objetivo
Simular la caída del servidor primario y realizar el failover manual promoviendo el secundario para mantener el servicio activo.
### Requisitos
•	Entorno replicación configurado (laboratorios previos).
•	Acceso a ambos servidores.
### Pasos
1.	Detener el servidor primario para simular fallo:
    ```bash
    sudo systemctl stop postgresql
2.	Promover el servidor secundario:
    ```bash
    pg_ctl -D /var/lib/postgresql/14/main promote
3.	Verificar que el secundario ahora es primario y acepta conexiones.
4.	(Opcional) Reconfigurar el antiguo primario para que sea secundario una vez vuelva a estar en línea.
5.	Documentar tiempo, pasos y resultados del proceso.
#### Explicación
Este procedimiento asegura la continuidad del servicio ante fallos, aunque es manual y requiere supervisión, siendo una base para sistemas de alta disponibilidad.
