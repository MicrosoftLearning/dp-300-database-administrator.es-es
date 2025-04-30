---
lab:
  title: 'Laboratorio 7: detección y corrección de problemas de fragmentación'
  module: Monitor and optimize operational resources in Azure SQL
---

# Detección y corrección de problemas de fragmentación

**Tiempo estimado**: 20 minutos

Los alumnos tomarán la información adquirida en las lecciones para definir los resultados de un proyecto de transformación digital dentro de AdventureWorks. Al examinar Azure Portal, así como otras herramientas, los alumnos determinarán cómo usar herramientas nativas para identificar y resolver problemas relacionados con el rendimiento. Por último, el alumnado podrá identificar la fragmentación dentro de la base de datos, así como conocer los pasos para resolverla correctamente.

Te han contratado como administrador de bases de datos para identificar problemas relacionados con el rendimiento y proporcionar soluciones viables para resolver los problemas detectados. AdventureWorks ha vendido bicicletas y piezas de bicicleta directamente a los consumidores y distribuidores durante más de una década. Recientemente, la empresa ha observado una degradación del rendimiento en sus productos que se usan para atender las solicitudes de los clientes. Debes usar herramientas SQL para identificar los problemas de rendimiento y sugerir métodos para resolverlos.

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

## Investigación de la fragmentación de índices

1. Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT i.name Index_Name
     , avg_fragmentation_in_percent
     , db_name(database_id)
     , i.object_id
     , i.index_id
     , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
     INNER JOIN sys.indexes i ON ps.object_id = i.object_id 
     AND ps.index_id = i.index_id
    WHERE avg_fragmentation_in_percent > 50 -- find indexes where fragmentation is greater than 50%
    ```

    Esta consulta informará sobre los índices que tengan una fragmentación superior al **50 %**. La consulta no debe devolver ningún resultado.

1. La fragmentación del índice puede deberse a una serie de factores, incluidos los siguientes:

    - Actualizaciones frecuentes de la tabla o índice.
    - Inserciones o eliminaciones frecuentes en la tabla o índice.
    - Divisiones de páginas.

    Para aumentar el nivel de fragmentación de la tabla Person.Address y sus índices, deberás insertar y eliminar un gran número de registros nuevos. Para hacerlo, ejecuta la siguiente consulta.

    Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    -- Insert 60000 records into the Address table    

    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Split Avenue ' + CAST(v1.number AS VARCHAR(10)), 
        'Apt ' + CAST(v2.number AS VARCHAR(10)), 
        'PageSplitTown', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '88' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure unique rowguid
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 300 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;
    GO
    
    -- DELETE 25000 records from the Address table
    DELETE FROM [Person].[Address] WHERE AddressID BETWEEN 35001 AND 60000;

    GO

    -- Insert 40000 records into the Address table
    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Fragmented Street ' + CAST(v1.number AS VARCHAR(10)), 
        'Suite ' + CAST(v2.number AS VARCHAR(10)), 
        'FragmentCity', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '99' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure a unique rowguid per row
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 200 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;

    GO
    ```

    Esta consulta aumentará el nivel de fragmentación de la tabla Person.Address y sus índices, agregando y eliminando un gran número de registros.

1. Vuelva a ejecutar la primera consulta. Ahora deberías poder ver cuatro índices altamente fragmentados.

1. Selecciona **Nueva consulta** y copia y pega el siguiente código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

    Selecciona la pestaña **Mensajes** en el panel de resultados de SQL Server Management Studio. Anota el recuento de lecturas lógicas realizadas por la consulta en la tabla **Drección**.

## Reconstrucción de índices fragmentados

1. Selecciona **Nueva consulta** y copia y pega el siguiente código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    ALTER INDEX [IX_Address_StateProvinceID] ON [Person].[Address] REBUILD PARTITION = ALL 
    WITH (PAD_INDEX = OFF, 
        STATISTICS_NORECOMPUTE = OFF, 
        SORT_IN_TEMPDB = OFF, 
        IGNORE_DUP_KEY = OFF, 
        ONLINE = OFF, 
        ALLOW_ROW_LOCKS = ON, 
        ALLOW_PAGE_LOCKS = ON)
    ```

1. Selecciona **New Query** y ejecuta la siguiente consulta para confirmar que el índice de **IX_Address_StateProvinceID** ya no tiene una fragmentación superior al 50 %.

    ```sql
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT i.name Index_Name
        , avg_fragmentation_in_percent
        , db_name(database_id)
        , i.object_id
        , i.index_id
        , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
        INNER JOIN sys.indexes i ON (ps.object_id = i.object_id AND ps.index_id = i.index_id)
    WHERE i.name = 'IX_Address_StateProvinceID'
    ```

    Comparando los resultados, podemos ver que la fragmentación **IX_Address_StateProvinceI** se ha reducido del 88 % al 0.

1. Vuelve a ejecutar la instrucción select de la sección anterior. Tome nota de las lecturas lógicas en la pestaña **Mensajes** del panel **Resultados** en Management Studio. *¿Hubo algún cambio en el número de lecturas lógicas detectadas antes de recompilar el índice para la tabla de direcciones?*

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

Dado que el índice se ha recompilado, ahora será lo más eficaz posible y deben reducirse las lecturas lógicas. Ahora ha descubierto que el mantenimiento de índices puede tener un efecto en el rendimiento de las consultas.

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

En este ejercicio, has aprendido a recopilar el índice y a analizar lecturas lógicas para aumentar el rendimiento de las consultas.
