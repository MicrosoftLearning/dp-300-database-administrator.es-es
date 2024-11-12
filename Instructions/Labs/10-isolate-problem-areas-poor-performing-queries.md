---
lab:
  title: 'Laboratorio 10: Aislamiento de áreas problemáticas en consultas de rendimiento deficiente en SQL Database'
  module: Optimize query performance in Azure SQL
---

# Aislamiento de áreas problemáticas en consultas de rendimiento deficiente en SQL Database

**Tiempo estimado: 30 minutos**

Te han contratado como administrador de bases de datos senior para ayudar con problemas de rendimiento que ocurren actualmente cuando los usuarios consultan la base de datos *AdventureWorks2017*. Tu trabajo consiste en identificar problemas en el rendimiento de las consultas y corregirlos mediante las técnicas aprendidas en este módulo.

Ejecutarás consultas con un rendimiento por debajo del nivel óptimo, examinarás los planes de consulta e intentarás realizar mejoras en la base de datos.

**Nota:** estos ejercicios piden copiar y pegar código T-SQL. Comprueba que el código se ha copiado correctamente antes de ejecutar el código.

## Restaurar una base de datos

1. Descarga el archivo de copia de seguridad de la base de datos ubicado en **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** en **C:\LabFiles\Monitor y optimiza** la ruta de acceso en la máquina virtual del laboratorio (crea la estructura de carpetas si no existe).

    ![Imagen 03](../images/dp-300-module-07-lab-03.png)

1. Selecciona el botón Inicio de Windows y escribe SSMS. Selecciona **Microsoft SQL Server Management Studio 18** en la lista.  

    ![Imagen 01](../images/dp-300-module-01-lab-34.png)

1. Cuando se abra SSMS, observa que el cuadro de diálogo **Conectar con el servidor** se rellenará previamente con el nombre de la instancia predeterminado. Selecciona **Conectar**.

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

## Generación del plan de ejecución real

Hay varias maneras de generar un plan de ejecución en SQL Server Management Studio.

1. Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    **Nota:** Usa **SHOWPLAN_ALL** para ver una versión en texto del plan de ejecución de una consulta en el panel de resultados, en lugar de hacerlo de forma gráfica en una pestaña independiente.

    ```sql
    USE AdventureWorks2017;
    GO

    SET SHOWPLAN_ALL ON;
    GO

    SELECT BusinessEntityID
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';
    GO

    SET SHOWPLAN_ALL OFF;
    GO
    ```

    Verás una versión en texto del plan de ejecución, en lugar de los resultados de la consulta real de la instrucción **SELECT**.

    ![Captura de pantalla que muestra la versión en texto de un plan de consulta.](../images/dp-300-module-10-lab-01.png)

1. Dedica un momento a examinar el texto de la segunda fila de la columna **StmtText**:

    ```console
    |--Index Seek(OBJECT:([AdventureWorks2017].[HumanResources].[Employee].[AK_Employee_NationalIDNumber]), SEEK:([AdventureWorks2017].[HumanResources].[Employee].[NationalIDNumber]=CONVERT_IMPLICIT(nvarchar(4000),[@1],0)) ORDERED FORWARD)
    ```

    En el texto anterior se explica que el plan de ejecución usa **Index Seek** en la clave **AK_Employee_NationalIDNumber**. También muestra que el plan de ejecución tuvo que realizar un paso **CONVERT_IMPLICIT**.

    El optimizador de consultas pudo localizar un índice adecuado para capturar los registros necesarios.

## Resolución de un plan de consulta poco óptimo

1. Copia el código siguiente y pégalo en una ventana de nueva consulta.

    Selecciona el icono **Incluir plan de ejecución real** como se muestra a continuación antes de ejecutar la consulta o presiona <kbd>CTRL</kbd>+<kbd>M</kbd>. Para ejecutar la consulta, selecciona **Ejecutar** o presiona <kbd>F5</kbd>. Anota el plan de ejecución y las lecturas lógicas de la pestaña Mensajes.

    ```sql
    SET STATISTICS IO, TIME ON;

    SELECT [SalesOrderID] ,[CarrierTrackingNumber] ,[OrderQty] ,[ProductID], [UnitPrice] ,[ModifiedDate]
    FROM [AdventureWorks2017].[Sales].[SalesOrderDetail]
    WHERE [ModifiedDate] > '2012/01/01' AND [ProductID] = 772;
    ```

    ![Captura de pantalla donde se muestra el plan de ejecución para la consulta.](../images/dp-300-module-10-lab-02.png)

    Al revisar el plan de ejecución, notarás que hay una **búsqueda de claves**. Si mantienes el mouse sobre el icono, verás que las propiedades indican que se realiza para cada fila que recupera la consulta. Puedes ver que el plan de ejecución está realizando una operación de **búsqueda de claves**.

    ![Captura de pantalla que muestra la lista de salida de columnas.](../images/dp-300-module-10-lab-03.png)

    Anota las columnas de la sección **Lista de resultados**. ¿Cómo mejorarías esta consulta?

    Para identificar qué índice se debe modificar para quitar la búsqueda de claves, debes examinar la búsqueda de índice sobre ella. Mantén el mouse sobre el operador Index seek y aparecerán las propiedades del operador.

    ![Captura de pantalla que muestra el índice no agrupado](../images/dp-300-module-10-lab-04.png)

1. Las **búsquedas de claves** se pueden quitar agregando un índice de cobertura que incluya todos los campos devueltos o buscados en la consulta. En este ejemplo, el índice solo usa la columna **ProductID**. Corrige la **búsqueda de claves** y vuelve a ejecutar la consulta para ver el nuevo plan.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID] ON [Sales].[SalesOrderDetail]
    ([ProductID] ASC)
    ```

    Si agregamos los campos de la **lista de salida** al índice como columnas incluidas, se quitará la **búsqueda de claves**. Puesto que el índice ya existe, debes ANULARLO y volver a crearlo, o bien establecer **DROP_EXISTING=ON** para agregar las columnas. Ten en cuenta que la columna **ProductID** ya forma parte del índice y no es necesario agregarlo como columna incluida. Hay otra mejora de rendimiento que se puede hacer en el índice si se agrega **ModifiedDate**.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID]
    ON [Sales].[SalesOrderDetail] ([ProductID],[ModifiedDate])
    INCLUDE ([CarrierTrackingNumber],[OrderQty],[UnitPrice])
    WITH (DROP_EXISTING = on);
    GO
    ```

1. Vuelve a ejecutar la consulta desde el paso 1. Toma nota de los cambios en las lecturas lógicas y los cambios en el plan de ejecución. El plan ahora solo necesita usar el índice no agrupado que creamos.

    ![Captura de pantalla que muestra el plan de ejecución mejorado.](../images/dp-300-module-10-lab-05.png)

## Uso del Almacén de consultas para detectar y controlar la regresión

A continuación, ejecutarás una carga de trabajo para generar estadísticas de consultas para el almacén de consultas, examinarás el informe **Top Resource Consuming Queries** para identificar rendimientos deficientes, y verás cómo forzar un mejor plan de ejecución.

1. Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    Este script habilitará la característica Almacén de consultas para la base de datos de AdventureWorks2017 y establecerá la base de datos en el nivel de compatibilidad 100.

    ```sql
    USE [master];
    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE = ON;
    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE (OPERATION_MODE = READ_WRITE);
    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 100;
    GO
    ```

    Cambiar el nivel de compatibilidad es como hacer retroceder la base de datos en el tiempo. Restringe las características que SQL Server puede usar a aquellas disponibles en SQL Server 2008.

1. Descarga el script de T-SQL ubicado en **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/CreateRandomWorkloadGenerator.sql** en **C:\LabFiles\Monitor y optimiza** la ruta de acceso en la máquina virtual del laboratorio.

1. Descarga el script de T-SQL ubicado en **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/ExecuteRandomWorkload.sql** en **C:\LabFiles\Monitor y optimiza** la ruta de acceso en la máquina virtual del laboratorio.

1. Selecciona el menú **Archivo** > **Abrir** > **Archivo** en SQL Server Management Studio.

1. Ve al archivo **C:\LabFiles\Monitor and archivo optimize\CreateRandomWorkloadGenerator.sql**.

1. Una vez abierto en SQL Server Management Studio, selecciona **Ejecutar** o presiona <kbd>F5</kbd> para ejecutar la consulta.

1. En un nuevo editor de consultas, abre el archivo **C:\LabFiles\Monitor and optimize\ExecuteRandomWorkload.sql** y selecciona **Ejecutar** o presiona <kbd>F5</kbd> para ejecutar la consulta.

1. Una vez finalizada la ejecución, ejecuta el script una segunda vez para crear una carga adicional en el servidor. Deja abierta la pestaña de consulta para esta consulta.

1. Copia el código siguiente y pégalo en una ventana de nueva consulta y, para ejecutarlo, selecciona **Ejecutar** o presiona <kbd>F5</kbd>. 

    Este script cambia el modo de compatibilidad de la base de datos a SQL Server 2019 (**150**). Todas las características y mejoras de la base de datos desde SQL Server 2008 estarán ahora disponibles para la base de datos.

    ```sql
    USE [master];
    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 150;
    GO
    ```

1. Vuelve a la pestaña de consulta desde el archivo **ExecuteRandomWorkload.sql** y vuelve a ejecutarla.

## Examen del informe de las consultas que más recursos consumen

1. Para ver el nodo del Almacén de consultas, deberás actualizar la base de datos AdventureWorks2017 en SQL Server Management Studio. Haz clic con el botón derecho en nombre de la base de datos y elige **Actualizar**. Verás el nodo del Almacén de consultas en la base de datos.

    ![Expansión del Almacén de consultas](../images/dp-300-module-10-lab-06.png)

1. Expande el nodo **Almacén de consultas** para ver todos los informes disponibles. Selecciona el informe **Top Resource Consuming Queries**.

    ![Top Resource Consuming Queries del Almacén de consultas](../images/dp-300-module-10-lab-07.png)

1. El informe se abrirá tal y como se muestra a continuación. A la derecha, selecciona el menú desplegable y, a continuación, selecciona **Configurar**.

    ![Selección de la opción configurar para el informe Top Resource Consuming Queries](../images/dp-300-module-10-lab-08.png)

1. En la pantalla de configuración, cambia el filtro para el número mínimo de planes de consulta a 2. Después, selecciona **Aceptar**.

    ![Establecer Número mínimo de planes de consulta](../images/dp-300-module-10-lab-09.png)

1. Elige la consulta de mayor duración; para ello, selecciona la barra situada más a la izquierda en el gráfico de barras en la parte superior izquierda del informe.

    ![Consulta con la mayor duración](../images/dp-300-module-10-lab-10.png)

    Esto te mostrará la consulta y el resumen del plan para la consulta de mayor duración en el Almacén de consultas.

## Forzado de un mejor plan de ejecución

1. Ve a la parte del informe donde se encuentra el resumen del plan, como se muestra a continuación. Observarás que hay dos planes de ejecución con duraciones muy diferentes.

    ![Resumen del plan](../images/dp-300-module-10-lab-11.png)

1. Selecciona el identificador del plan con la menor duración (esto se indica mediante una posición inferior en el eje Y del gráfico) en la ventana superior derecha del informe. En el gráfico anterior, es *PlanID 43*. Selecciona el identificador del plan situado junto al gráfico de resumen del plan (tiene que estar resaltado como en la captura de pantalla anterior).

1. Selecciona **Forzar plan** en el gráfico del resumen. Aparecerá una ventana de confirmación, selecciona **Sí**.

    ![Captura de pantalla que muestra la confirmación](../images/dp-300-module-10-lab-12.png)

    Una vez que lo hayas forzado, verás que el **plan forzado** ahora está atenuado, y el plan de la ventana Resumen del plan ahora tiene una marca de verificación que indica que se ha forzado.

    Puede haber ocasiones en las que el optimizador de consultas elija de forma deficiente el plan de ejecución que se va a usar. Cuando esto suceda, puedes forzar SQL Server para que use el plan que quieras cuando sepas que funcionará mejor.

## Uso de sugerencias de consulta para influir en el rendimiento

A continuación, ejecutarás una carga de trabajo, cambiarás la consulta para usar un parámetro, aplicarás la sugerencia de consulta a la consulta y volverás a ejecutarla.

Antes de continuar con el ejercicio, cierra todas las ventanas de consulta actuales seleccionando el menú **Ventana** y, a continuación, selecciona **Cerrar todos los documentos**. En la ventana emergente, selecciona **No**.

1. Selecciona **Nueva consulta** y luego el icono **Incluir plan de ejecución real** antes de ejecutar la consulta o usa <kbd>CTRL</kbd>+<kbd>M</kbd>.

    ![Incluir plan de ejecución real](../images/dp-300-module-10-lab-13.png)

1. Ejecuta la consulta siguiente. Ten en cuenta que el plan de ejecución muestra un operador de búsqueda de índice.

    ```sql
    USE AdventureWorks2017;
    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=288;
    ```

    ![Captura de pantalla que muestra el plan de ejecución actualizado.](../images/dp-300-module-10-lab-14.png)

1. En una nueva ventana de consulta, ejecuta la siguiente consulta. Compara los planes de ejecución.

    ```sql
    USE AdventureWorks2017;
    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=277;
    ```

    El único cambio esta vez es que el valor de SalesPersonID se ha establecido en 277. Ten en cuenta la operación Clustered Index Scan en el plan de ejecución.

    ![Captura de pantalla que muestra la instrucción SQL](../images/dp-300-module-10-lab-15.png)

Como podemos ver, en función de las estadísticas del índice, el optimizador de consultas ha elegido otro plan de ejecución debido a los distintos valores de la cláusula `WHERE`.

¿Por qué tenemos planes diferentes si solo cambiamos el valor *SalesPersonID*?

Dado que esta consulta usa una constante en su cláusula `WHERE`, el optimizador ve cada una de estas consultas como únicas y genera un plan de ejecución diferente cada vez.

## Cambio de consulta para usar una variable y usar una sugerencia de consulta

1. Cambia la consulta para usar un valor variable para SalesPersonID.

1. Usa la instrucción **DECLARE** de T-SQL para declarar <strong>@SalesPersonID</strong>, de modo que puedas pasar un valor en lugar de codificar de forma rígida el valor en la cláusula **WHERE**. Debes asegurarte de que el tipo de datos de la variable coincide con el tipo de datos de la columna en la tabla de destino para evitar una conversión implícita.

    ```sql
    USE AdventureWorks2017;
    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID;
    ```

    Si examinas el plan de ejecución, notarás que usa una operación Index Scan para obtener los resultados. El optimizador de consultas no pudo realizar buenas optimizaciones porque no puede conocer el valor de la variable local hasta runtime.

1. Para ayudar a que el optimizador de consultas tome mejores decisiones proporciona una sugerencia de consulta. Vuelve a ejecutar la consulta anterior con `OPTION (RECOMPILE)`:

    ```sql
    USE AdventureWorks2017
    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID
    OPTION (RECOMPILE);
    ```

    Ten en cuenta que el optimizador de consultas ha podido elegir un plan de ejecución más eficaz. La opción `RECOMPILE` hace que el compilador de consultas reemplace la variable por su valor.

    Al comparar las estadísticas puedes ver en la pestaña Mensajes que la diferencia entre lecturas lógicas es de un **68 %** (689 frente a 409) más para la consulta sin la sugerencia de consulta.

En este ejercicio, has aprendido a identificar problemas de consulta y a corregirlos para mejorar el plan de consulta.
