---
lab:
  title: 'Laboratorio 7: detección y corrección de problemas de fragmentación'
  module: Monitor and optimize operational resources in Azure SQL
---

# Detección y corrección de problemas de fragmentación

**Tiempo estimado**: 15 minutos

Los alumnos tomarán la información adquirida en las lecciones para definir los resultados de un proyecto de transformación digital dentro de AdventureWorks. Al examinar Azure Portal, así como otras herramientas, los alumnos determinarán cómo usar herramientas nativas para identificar y resolver problemas relacionados con el rendimiento. Por último, el alumnado podrá identificar la fragmentación dentro de la base de datos, así como conocer los pasos para resolverla correctamente.

Le han contratado como administrador de bases de datos para identificar problemas relacionados con el rendimiento y proporcionar soluciones viables para resolver los problemas detectados. AdventureWorks ha vendido bicicletas y piezas de bicicleta directamente a los consumidores y distribuidores durante más de una década. Recientemente, la empresa ha observado una degradación del rendimiento en sus productos que se usan para atender las solicitudes de los clientes. Debes usar herramientas SQL para identificar los problemas de rendimiento y sugerir métodos para resolverlos.

**Nota:** estos ejercicios piden copiar y pegar código T-SQL. Comprueba que el código se ha copiado correctamente antes de ejecutar el código.

## Restauración de una base de datos

1. Descarga el archivo de copia de seguridad de la base de datos ubicado en **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** en **C:\LabFiles\Monitor y optimiza** la ruta de acceso en la máquina virtual del laboratorio (crea la estructura de carpetas si no existe).

    ![Imagen 03](../images/dp-300-module-07-lab-03.png)

1. Selecciona el botón Inicio de Windows y escribe SSMS. Selecciona **Microsoft SQL Server Management Studio 18** en la lista.  

    ![Imagen 01](../images/dp-300-module-01-lab-34.png)

1. Cuando se abra SSMS, observa que el cuadro de diálogo **Conectar con el servidor** se rellenará previamente con el nombre de la instancia predeterminado. Seleccione **Conectar**.

    ![Imagen 02](../images/dp-300-module-07-lab-01.png)

1. Selecciona la carpeta **Bases de datos** y después **Nueva consulta**.

    ![Imagen 03](../images/dp-300-module-07-lab-04.png)

1. En la ventana Nueva consulta, copia y pega la consulta de T-SQL siguiente. Ejecuta la consulta para restaurar la base de datos.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017_log.ldf';
    ```

    **Nota:** el nombre y la ruta de acceso del archivo de copia de seguridad de la base de datos deben coincidir con lo descargado en el paso 1; de lo contrario, se producirá un error en el comando.

1. Debería aparecer un mensaje de operación correcta una vez completada la restauración.

    ![Imagen 03](../images/dp-300-module-07-lab-05.png)

## Investigación de la fragmentación de índices

1. Seleccione **Nueva consulta**. Copie y pegue el código T-SQL siguiente en la ventana de consulta. Seleccione **Ejecutar** para ejecutar esta consulta.

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

1. Seleccione **Nueva consulta**. Copie y pegue el código T-SQL siguiente en la ventana de consulta. Seleccione **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE AdventureWorks2017
    GO
        
    INSERT INTO [Person].[Address]
        ([AddressLine1]
        ,[AddressLine2]
        ,[City]
        ,[StateProvinceID]
        ,[PostalCode]
        ,[SpatialLocation]
        ,[rowguid]
        ,[ModifiedDate])
        
    SELECT AddressLine1,
        AddressLine2, 
        'Amsterdam',
        StateProvinceID, 
        PostalCode, 
        SpatialLocation, 
        newid(), 
        getdate()
    FROM Person.Address;
    
    GO
    ```

    Esta consulta aumentará el nivel de fragmentación de la tabla Person.Address y sus índices, agregando un gran número de registros nuevos.

1. Vuelva a ejecutar la primera consulta. Ahora deberías poder ver cuatro índices altamente fragmentados.

    ![Imagen 03](../images/dp-300-module-07-lab-06.png)

1. Copie y pegue el código T-SQL siguiente en la ventana de consulta. Seleccione **Ejecutar** para ejecutar esta consulta.

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

    Haga clic en la pestaña **Mensajes** en el panel de resultados de SQL Server Management Studio. Anote el recuento de lecturas lógicas realizadas por la consulta.

    ![Imagen 03](../images/dp-300-module-07-lab-07.png)

## Reconstrucción de índices fragmentados

1. Copie y pegue el código T-SQL siguiente en la ventana de consulta. Seleccione **Ejecutar** para ejecutar esta consulta.

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

1. Ejecuta la siguiente consulta para confirmar que el índice de **IX_Address_StateProvinceID** ya no tiene una fragmentación superior al 50 %.

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

    Comparando los resultados, podemos ver que la fragmentación se ha reducido del 81 % al 0.

1. Vuelve a ejecutar la instrucción select de la sección anterior. Tome nota de las lecturas lógicas en la pestaña **Mensajes** del panel **Resultados** en Management Studio. ¿Hubo algún cambio en el número de lecturas lógicas detectadas antes de recompilar el índice?

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

En este ejercicio, has aprendido a recopilar el índice y a analizar lecturas lógicas para aumentar el rendimiento de las consultas.
