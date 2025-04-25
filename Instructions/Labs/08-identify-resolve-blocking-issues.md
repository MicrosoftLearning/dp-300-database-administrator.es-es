---
lab:
  title: 'Laboratorio 8: Identificación y resolución de problemas de bloqueo'
  module: Optimize query performance in Azure SQL
---

# Identificación y resolución de problemas de bloqueo

**Tiempo estimado**: 15 minutos

Los alumnos tomarán la información adquirida en las lecciones para definir los resultados de un proyecto de transformación digital dentro de AdventureWorks. Al examinar Azure Portal, así como otras herramientas, los alumnos determinarán cómo usar herramientas nativas para identificar y resolver problemas relacionados con el rendimiento. Por último, los alumnos podrán identificar y resolver los problemas de bloqueo adecuadamente.

Te han contratado como administrador de bases de datos para identificar problemas relacionados con el rendimiento y proporcionar soluciones viables para resolver los problemas detectados. Debes investigar los problemas de rendimiento y sugerir métodos para resolverlos.

> &#128221; Estos ejercicios te piden copiar y pegar código T-SQL. Comprueba que el código se ha copiado correctamente antes de ejecutar el código.

## Configura el entorno

Si la máquina virtual de laboratorio se proporcionó y está preconfigurada, deberías encontrar los archivos de laboratorio listos en la carpeta **C:\LabFiles**. *Dedica un momento a comprobar si los archivos ya están allí, y omite esta sección*. Sin embargo, si usas tu propia máquina o faltan los archivos para laboratorio, deberás clonarlos desde *GitHub* para continuar.

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, inicia una sesión de Visual Studio Code.

1. Abre la paleta de comandos (CTRL+Mayús+P) y escribe **Git: Clone**. Selecciona la opción **Git: Clone**.

1. Pega la siguiente dirección URL en el campo **Repository URL** y selecciona **Enter**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Guarda el repositorio en la carpeta **C:\LabFiles** en la máquina virtual del laboratorio o en tu máquina local si no se te proporcionó ninguna (crea la carpeta si no existe).

---

## Restaurar una base de datos

Si ya tienes restaurada la base de datos **AdventureWorks2017**, puedes omitir esta sección.

1. Desde la máquina virtual del laboratorio o la máquina local si no se proporcionó una, inicia una sesión de SQL Server Management Studio (SSMS).

1. Cuando se abra SSMS, aparecerá de forma predeterminada el cuadro de diálogo **Conectar al servidor**. Elige la instancia predeterminada y selecciona **Conectar**. Es posible que tengas que activar la casilla **Certificado de servidor de confianza**.

    > &#128221; Ten en cuenta que si usas tu propia instancia de SQL Server, deberás conectarte a ella mediante el nombre y las credenciales de la instancia de servidor adecuadas.

1. Seleccione la carpeta **Bases de datos** y, a continuación, **Nueva consulta**.

1. En la ventana Nueva consulta, copie y pegue T-SQL. Ejecute la consulta para restaurar la base de datos.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; Debes tener una carpeta denominada **C:\LabFiles**. Si no tienes esta carpeta, créala o especifica otra ubicación para la base de datos y los archivos de copia de seguridad.

1. En la pestaña **Mensajes**, deberías ver un mensaje que indica que la base de datos se restauró correctamente.

## Ejecución de un informe de consultas bloqueadas

1. Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE MASTER

    GO

    CREATE EVENT SESSION [Blocking] ON SERVER 
    ADD EVENT sqlserver.blocked_process_report(
    ACTION(sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.database_id,sqlserver.database_name,sqlserver.nt_username,sqlserver.session_id,sqlserver.sql_text,sqlserver.username))
    ADD TARGET package0.ring_buffer
    WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS, MAX_DISPATCH_LATENCY=30 SECONDS, MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE, TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)

    GO

    -- Start the event session 
    ALTER EVENT SESSION [Blocking] ON SERVER 
    STATE = start;

    GO
    ```

    El código T-SQL anterior creará una sesión de evento extendido que capturará los eventos de bloqueo. Los datos contendrán los siguientes elementos:

    - Nombre de la aplicación cliente
    - Nombre de host del cliente.
    - Identificador de base de datos
    - Nombre de la base de datos
    - Nombre de usuario de NT
    - Id. sesión
    - Texto T-SQL
    - Nombre de usuario

1. Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    EXEC sys.sp_configure N'show advanced options', 1

    RECONFIGURE WITH OVERRIDE;

    GO
    EXEC sp_configure 'blocked process threshold (s)', 60

    RECONFIGURE WITH OVERRIDE;

    GO
    ```

    > &#128221; Ten en cuenta que el comando anterior especifica el umbral, en segundos, en el que se generan los informes de procesos bloqueados. Como resultado, no es necesario esperar hasta que *blocked_process_report* se genere en esta lección.

1. Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO

    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;

    GO
    ```

1. Para abrir otra ventana de consulta, selecciona el botón **Nueva consulta**. Copia y pega el siguiente código T-SQL siguiente en la nueva ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO

    SELECT TOP (1000) [LastName]
      ,[FirstName]
      ,[Title]
    FROM Person.Person
    WHERE FirstName = 'David'
    ```

    > &#128221; Ten en cuenta que esta consulta no devuelve ningún resultado y parece ejecutarse indefinidamente.

1. En el **Explorador de objetos**, expande **Administración** -> **Eventos extendidos** -> **Sesiones**.

    Observa el evento extendido denominado *Bloqueos* que acabamos de crear está en la lista.

1. Expande el evento extendido *Bloqueo* y haz clic con el botón derecho en **package0.ring_buffer**. Selecciona **Ver datos de destino**.

1. Selecciona el hipervínculo que aparece.

1. El XML te mostrará los procesos que se están bloqueando y el proceso que causa el bloqueo. Puedes ver las consultas que se ejecutaron en este proceso, así como la información del sistema. Toma nota de los id. de sesión (SPID).

1. Como alternativa, puedes ejecutar la siguiente consulta para identificar las sesiones que bloquean otras sesiones, incluida una lista de id. de sesión bloqueados por *session_id*. Abre una ventana de **Nueva consulta**, copia y pega el siguiente código T-SQL en ella y selecciona **Ejecutar**.

    ```sql
    WITH cteBL (session_id, blocking_these) AS 
    (SELECT s.session_id, blocking_these = x.blocking_these FROM sys.dm_exec_sessions s 
    CROSS APPLY    (SELECT isnull(convert(varchar(6), er.session_id),'') + ', '  
                    FROM sys.dm_exec_requests as er
                    WHERE er.blocking_session_id = isnull(s.session_id ,0)
                    AND er.blocking_session_id <> 0
                    FOR XML PATH('') ) AS x (blocking_these)
    )
    SELECT s.session_id, blocked_by = r.blocking_session_id, bl.blocking_these
    , batch_text = t.text, input_buffer = ib.event_info, * 
    FROM sys.dm_exec_sessions s 
    LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
    INNER JOIN cteBL as bl on s.session_id = bl.session_id
    OUTER APPLY sys.dm_exec_sql_text (r.sql_handle) t
    OUTER APPLY sys.dm_exec_input_buffer(s.session_id, NULL) AS ib
    WHERE blocking_these is not null or r.blocking_session_id > 0
    ORDER BY len(bl.blocking_these) desc, r.blocking_session_id desc, r.session_id;
    ```

    > &#128221; Ten en cuenta que la consulta anterior devolverá los mismos SPID que el XML.

1. Haz clic con el botón derecho en el evento extendido denominado **Bloqueos** y selecciona **Detener sesión**.

1. Vuelve a la sesión de consulta que está causando el bloqueo y escribe `ROLLBACK TRANSACTION` en la línea debajo de la consulta. Resalta `ROLLBACK TRANSACTION`y selecciona **Ejecutar**.

1. Vuelva a la sesión de consulta que se bloqueó. Observarás que ahora se ha completado la consulta.

## Habilitación del nivel de aislamiento de instantánea de confirmación de lectura

1. En SQL Server Management Studio, selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona en el botón **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE master

    GO
    
    ALTER DATABASE AdventureWorks2017 SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;

    GO
    ```

1. Vuelve a ejecutar la consulta que provocó el bloqueo en un nuevo editor de consultas. *No ejecutes el comando ROLLBACK TRANSACTION*.

    ```sql
    USE AdventureWorks2017
    GO
    
    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;
    GO
    ```

1. Vuelve a ejecutar la consulta que se bloqueó en un nuevo editor de consultas.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT TOP (1000) [LastName]
     ,[FirstName]
     ,[Title]
    FROM Person.Person
    WHERE firstname = 'David'
    ```

    ¿Por qué se completa la misma consulta, mientras que en la tarea anterior estaba bloqueada por la instrucción de actualización?

    El aislamiento de instantánea de lectura confirmada es una forma optimista de aislamiento de transacciones y la última consulta muestra la última versión confirmada de los datos, en lugar de bloquearse.

---

## Limpieza

Si no usas la base de datos o los archivos de laboratorio para ningún otro propósito, puedes limpiar los objetos que creaste en este laboratorio.

### Eliminar la carpeta C:\LabFiles

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, abre **Explorador de archivos**.
1. Ve a **C:\\**.
1. Elimina la carpeta **C:\LabFiles**.

### Eliminar la base de datos AdventureWorks2017

1. Desde la máquina virtual del laboratorio o la máquina local si no se proporcionó una, inicia una sesión de SQL Server Management Studio (SSMS).
1. Cuando se abra SSMS, aparecerá de forma predeterminada el cuadro de diálogo **Conectar al servidor**. Elige la instancia predeterminada y selecciona **Conectar**. Es posible que tengas que activar la casilla **Certificado de servidor de confianza**.
1. En el **Explorador de objetos**, expande la carpeta **Bases de datos**.
1. Haz clic con el botón derecho en la base de datos **AdventureWorks2017** y selecciona **Eliminar**.
1. En el cuadro de diálogo **Eliminar objeto**, activa la casilla **Cerrar conexiones existentes**.
1. Seleccione **Aceptar**.

---

Completaste correctamente este laboratorio.

En este ejercicio, has aprendido a identificar las sesiones bloqueadas y a mitigar esos escenarios.
