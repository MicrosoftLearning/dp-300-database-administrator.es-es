---
lab:
  title: 'Laboratorio 10: Aislamiento de áreas problemáticas en consultas de rendimiento deficiente en SQL Database'
  module: Optimize query performance in Azure SQL
---

# Aislamiento de áreas problemáticas en consultas de rendimiento deficiente en SQL Database

**Tiempo estimado: 30 minutos**

Te han contratado como administrador de bases de datos senior para ayudar con problemas de rendimiento que ocurren actualmente cuando los usuarios consultan la base de datos *AdventureWorks2017*. Tu trabajo consiste en identificar problemas en el rendimiento de las consultas y corregirlos mediante las técnicas aprendidas en este módulo.

Ejecutarás consultas con un rendimiento por debajo del nivel óptimo, examinarás los planes de consulta e intentarás realizar mejoras en la base de datos.

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

    En el panel de resultados, verás una versión en texto del plan de ejecución, en lugar de los resultados de la consulta real de la instrucción **SELECT**.

1. Dedica un momento a examinar el texto de la segunda fila de la columna **StmtText**:

    ```console
    |--Index Seek(OBJECT:([AdventureWorks2017].[HumanResources].[Employee].[AK_Employee_NationalIDNumber]), SEEK:([AdventureWorks2017].[HumanResources].[Employee].[NationalIDNumber]=CONVERT_IMPLICIT(nvarchar(4000),[@1],0)) ORDERED FORWARD)
    ```

    En el texto anterior se explica que el plan de ejecución usa **Index Seek** en la clave **AK_Employee_NationalIDNumber**. También muestra que el plan de ejecución tuvo que realizar un paso **CONVERT_IMPLICIT**.

    El optimizador de consultas pudo localizar un índice adecuado para capturar los registros necesarios.

## Resolución de un plan de consulta poco óptimo

1. Copia el código siguiente y pégalo en una ventana de nueva consulta.

    Selecciona el icono **Incluir plan de ejecución real** a la derecha del botón Ejecutar o presiona <kbd>CTRL</kbd>+<kbd>M</kbd> Para ejecutar la consulta, selecciona **Ejecutar** o presiona <kbd>F5</kbd>. Anota el plan de ejecución y las lecturas lógicas de la pestaña Mensajes.

    ```sql
    SET STATISTICS IO, TIME ON;

    SELECT [SalesOrderID] ,[CarrierTrackingNumber] ,[OrderQty] ,[ProductID], [UnitPrice] ,[ModifiedDate]
    FROM [AdventureWorks2017].[Sales].[SalesOrderDetail]
    WHERE [ModifiedDate] > '2012/01/01' AND [ProductID] = 772;
    ```

    Al revisar el plan de ejecución, notarás que hay una **búsqueda de claves**. Si mantienes el mouse sobre el icono, verás que las propiedades indican que se realiza para cada fila que recupera la consulta. Puedes ver que el plan de ejecución está realizando una operación de **búsqueda de claves**.

    Anota las columnas de la sección **Lista de resultados**. ¿Cómo mejorarías esta consulta?

    Para identificar qué índice se debe modificar para quitar la búsqueda de claves, debes examinar la búsqueda de índice sobre ella. Mantén el mouse sobre el operador Index seek y aparecerán las propiedades del operador.

1. Las **búsquedas de claves** se pueden quitar agregando un índice de cobertura que incluya todos los campos devueltos o buscados en la consulta. En este ejemplo, el índice solo usa la columna **ProductID**. A continuación se muestra la definición actual del índice, ten en cuenta que la columna **ProductID** es la única columna de clave que fuerza una **Búsqueda de claves** para recuperar las otras columnas requeridas por la consulta.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID] ON [Sales].[SalesOrderDetail]
    ([ProductID] ASC)
    ```

    Si agregamos los campos de la **lista de salida** al índice como columnas incluidas, se quitará la **búsqueda de claves**. Puesto que el índice ya existe, debes ANULARLO y volver a crearlo, o bien establecer **DROP_EXISTING=ON** para agregar las columnas. Ten en cuenta que la columna **ProductID** ya forma parte del índice y no es necesario agregarlo como columna incluida. Hay otra mejora de rendimiento que se puede hacer en el índice si se agrega **ModifiedDate**. Abre una ventana **Nueva consulta** y ejecuta el siguiente script para quitarlo y volver a crear el índice.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID]
    ON [Sales].[SalesOrderDetail] ([ProductID],[ModifiedDate])
    INCLUDE ([CarrierTrackingNumber],[OrderQty],[UnitPrice])
    WITH (DROP_EXISTING = on);
    GO
    ```

1. Vuelve a ejecutar la consulta desde el paso 1. Toma nota de los cambios en las lecturas lógicas y los cambios en el plan de ejecución. El plan ahora solo necesita usar el índice no agrupado que creamos.

> &#128221; Al revisar el plan de ejecución, observarás que la **búsqueda de clave** ha desaparecido y que solo se usa el índice no agrupado.

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

1. Selecciona el menú **Archivo** > **Abrir** > **Archivo** en SQL Server Management Studio.

1. Ve al archivo **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\10\CreateRandomWorkloadGenerator.sql**.

1. Una vez abierto en SQL Server Management Studio, selecciona **Ejecutar** para ejecutar la consulta.

1. En un nuevo editor de consultas, abre el archivo **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\10\ExecuteRandomWorkload.sql** y selecciona **Ejecutar** para ejecutar la consulta.

1. Una vez finalizada la ejecución, ejecuta el script una segunda vez para crear una carga adicional en el servidor. Deja abierta la pestaña de consulta para esta consulta.

1. Copia el código siguiente y pégalo en una ventana de nueva consulta y ejecútalo seleccionando **Ejecutar**.

    Este script cambia el modo de compatibilidad de la base de datos a SQL Server 2022 (**160**). Todas las características y mejoras de la base de datos desde SQL Server 2008 estarán ahora disponibles para la base de datos.

    ```sql
    USE [master];

    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 160;

    GO
    ```

1. Vuelve a la pestaña de consulta desde el archivo **ExecuteRandomWorkload.sql** y vuelve a ejecutarla.

## Examen del informe de las consultas que más recursos consumen

1. Para ver el nodo del Almacén de consultas, deberás actualizar la base de datos AdventureWorks2017 en SQL Server Management Studio. Haz clic con el botón derecho en nombre de la base de datos y elige **Actualizar**. Verás el nodo del Almacén de consultas en la base de datos.

1. Expande el nodo **Almacén de consultas** para ver todos los informes disponibles. Selecciona el informe **Top Resource Consuming Queries**.

1. Cuando se abra el informe, selecciona la lista desplegable de menús y después selecciona **Configurar** en la esquina superior derecha del informe.

1. En la pantalla de configuración, cambia el filtro para el número mínimo de planes de consulta a 2. Después, selecciona **Aceptar**.

1. Elige la consulta de mayor duración; para ello, selecciona la barra situada más a la izquierda en el gráfico de barras en la parte superior izquierda del informe.

    Esto te mostrará la consulta y el resumen del plan para la consulta de mayor duración en el Almacén de consultas. Consulta el gráfico *Resumen del plan* en la esquina superior derecha del informe y el *plan de consulta* en la parte inferior del informe.

## Forzado de un mejor plan de ejecución

1. Ve a la parte del informe donde se encuentra el resumen del plan, como se muestra a continuación. Observarás que hay dos planes de ejecución con duraciones muy diferentes.

1. Selecciona el identificador del plan con la menor duración (esto se indica mediante una posición inferior en el eje Y del gráfico) en la ventana superior derecha del informe. Selecciona el id. del plan junto al gráfico Resumen del plan.

1. Selecciona **Forzar plan** en el gráfico del resumen. Aparecerá una ventana de confirmación, selecciona **Sí**.

    Una vez que lo hayas forzado, verás que el **plan forzado** ahora está atenuado, y el plan de la ventana Resumen del plan ahora tiene una marca de verificación que indica que se ha forzado.

    Puede haber ocasiones en las que el optimizador de consultas elija de forma deficiente el plan de ejecución que se va a usar. Cuando esto suceda, puedes forzar SQL Server para que use el plan que quieras cuando sepas que funcionará mejor.

## Uso de sugerencias de consulta para influir en el rendimiento

A continuación, ejecutarás una carga de trabajo, cambiarás la consulta para usar un parámetro, aplicarás la sugerencia de consulta a la consulta y volverás a ejecutarla.

Antes de continuar con el ejercicio, cierra todas las ventanas de consulta actuales seleccionando el menú **Ventana** y, a continuación, selecciona **Cerrar todos los documentos**. En la ventana emergente, selecciona **No**.

1. Selecciona **Nueva consulta** y luego el icono **Incluir plan de ejecución real** antes de ejecutar la consulta o usa <kbd>CTRL</kbd>+<kbd>M</kbd>.

1. Ejecuta la consulta siguiente. Ten en cuenta que el plan de ejecución muestra un operador de búsqueda de índice.

    ```sql
    USE AdventureWorks2017;

    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=288;
    ```

1. En una nueva ventana de consulta, ejecuta la siguiente consulta. Compara los planes de ejecución.

    ```sql
    USE AdventureWorks2017;
    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=277;
    ```

    El único cambio esta vez es que el valor de SalesPersonID se ha establecido en 277. Ten en cuenta la operación Clustered Index Scan en el plan de ejecución.

Como podemos ver, en función de las estadísticas del índice, el optimizador de consultas ha elegido otro plan de ejecución debido a los distintos valores de la cláusula **WHERE**.

¿Por qué tenemos planes diferentes si solo cambiamos el valor *SalesPersonID*?

Dado que esta consulta usa una constante en su cláusula **WHERE**, el optimizador ve cada una de estas consultas como únicas y genera un plan de ejecución diferente cada vez.

## Cambio de consulta para usar una variable y usar una sugerencia de consulta

1. Cambia la consulta para usar un valor variable para SalesPersonID.

1. Usa la instrucción **DECLARE** de T-SQL para declarar <strong>@SalesPersonID</strong>, de modo que puedas pasar un valor en lugar de codificar de forma rígida el valor en la cláusula **WHERE**. Debes asegurarte de que el tipo de datos de la variable coincide con el tipo de datos de la columna en la tabla de destino para evitar una conversión implícita. Ejecuta la consulta con el plan de consulta real habilitado.

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

1. Para ayudar a que el optimizador de consultas tome mejores decisiones proporciona una sugerencia de consulta. Vuelve a ejecutar la consulta anterior con **OPTION (RECOMPILE)**:

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

    Ten en cuenta que el optimizador de consultas ha podido elegir un plan de ejecución más eficaz. La opción **RECOMPILE** hace que el compilador de consultas reemplace la variable por su valor.

    Al comparar las estadísticas puedes ver en la pestaña Mensajes que la diferencia entre lecturas lógicas es de un **68 %** (689 frente a 409) más para la consulta sin la sugerencia de consulta.

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

En este ejercicio, has aprendido a identificar problemas de consulta y a corregirlos para mejorar el plan de consulta.
