---
lab:
  title: 'Laboratorio 9: identificación de problemas de diseño de bases de datos'
  module: Optimize query performance in Azure SQL
---

# Identificación de problemas de diseño de bases de datos

**Tiempo estimado**: 15 minutos

Los alumnos tomarán la información adquirida en las lecciones para definir los resultados de un proyecto de transformación digital dentro de AdventureWorks. Al examinar Azure Portal, así como otras herramientas, los alumnos determinarán cómo usar herramientas nativas para identificar y resolver problemas relacionados con el rendimiento. Finalmente, los alumnos podrán evaluar un diseño de base de datos para los problemas con la normalización, la selección de tipos de datos y el diseño de índices.

Te han contratado como administrador de bases de datos para identificar problemas relacionados con el rendimiento y proporcionar soluciones viables para resolver los problemas detectados. AdventureWorks ha vendido bicicletas y piezas de bicicleta directamente a los consumidores y distribuidores durante más de una década. Tu trabajo consiste en identificar problemas en el rendimiento de las consultas y corregirlos mediante las técnicas aprendidas en este módulo.

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

## Examen de la consulta e identificación del problema

1. Selecciona **Nueva consulta**. Copia y pega el código T-SQL siguiente en la ventana de consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

1. Selecciona el icono **Incluir plan de ejecución real** a la derecha del botón **Ejecutar** antes de ejecutar la consulta o presiona **CTRL+M**. Esto hará que se muestre el plan de ejecución al ejecutar la consulta. Selecciona **Ejecutar** para ejecutar esta consulta.

1. Para ir al plan de ejecución, selecciona la pestaña **Plan de ejecución** en el panel de resultados. Observarás que el operador **SELECT** tiene un triángulo amarillo con un signo de exclamación en él. Esto indica que hay un mensaje de advertencia asociado al operador. Mantén el puntero sobre el icono de advertencia para ver el mensaje y leer el mensaje de advertencia.

    > &#128221; El mensaje de advertencia indica que hay una conversión implícita en la consulta. Esto significa que el optimizador de consultas de SQL Server tenía que convertir el tipo de datos de una de las columnas de la consulta a otro tipo de datos para ejecutar la consulta.

## Identificación de maneras de corregir el mensaje de advertencia

La estructura de la tabla *[HumanResources].[Employee]* se muestra en la instrucción del lenguaje de definición de datos (DDL) siguiente. Revise los campos que se usan en la consulta SQL anterior en este DDL y preste atención a los tipos.

```sql
CREATE TABLE [HumanResources].[Employee](
     [BusinessEntityID] [int] NOT NULL,
     [NationalIDNumber] [nvarchar](15) NOT NULL,
     [LoginID] [nvarchar](256) NOT NULL,
     [OrganizationNode] [hierarchyid] NULL,
     [OrganizationLevel] AS ([OrganizationNode].[GetLevel]()),
     [JobTitle] [nvarchar](50) NOT NULL,
     [BirthDate] [date] NOT NULL,
     [MaritalStatus] [nchar](1) NOT NULL,
     [Gender] [nchar](1) NOT NULL,
     [HireDate] [date] NOT NULL,
     [SalariedFlag] [dbo].[Flag] NOT NULL,
     [VacationHours] [smallint] NOT NULL,
     [SickLeaveHours] [smallint] NOT NULL,
     [CurrentFlag] [dbo].[Flag] NOT NULL,
     [rowguid] [uniqueidentifier] ROWGUIDCOL NOT NULL,
     [ModifiedDate] [datetime] NOT NULL
) ON [PRIMARY]
```

1. Según el mensaje de advertencia presentado en el plan de ejecución, ¿qué cambio recomendarías?

    1. Identifique qué campo causa la conversión implícita y por qué.
    1. Si revisa la consulta:

        ```sql
        SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
        FROM HumanResources.Employee
        WHERE NationalIDNumber = 14417807;
        ```

        Notarás que el valor comparado con la columna *NationalIDNumber* de la cláusula **WHERE** se compara como un número, ya que **14417807** no está en una cadena entre comillas.

        Después de examinar la estructura de la tabla, verás que la columna *NationalIDNumber* usa el tipo de datos **NVARCHAR** y no un tipo de datos entero **INT**. Esta incoherencia hace que el optimizador de la base de datos convierta de manera implícita el número a un valor *NVARCHAR*, lo que genera una sobrecarga adicional en el rendimiento de la consulta mediante la creación de un plan poco óptimo.

Hay dos enfoques que se pueden implementar para corregir la advertencia de conversión implícita. Investigaremos cada uno de ellos en los pasos siguientes.

### Cambio del código

1. ¿Cómo cambiarías el código para resolver la conversión implícita? Cambia el código y vuelve a ejecutar la consulta.

    No olvides activar la opción **Incluir plan de ejecución real** (**CTRL+M**) si todavía no está activada. 

    En este escenario, si solo se agrega una comilla simple a cada lado del valor, lo transforma de número a formato de caracteres. Mantenga abierta la ventana de consulta de esta consulta.

    Ejecute la consulta SQL actualizada:

    ```sql
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';
    ```

    > &#128221; Ten en cuenta que el mensaje de advertencia ha desaparecido y el plan de consulta se ha mejorado. Al cambiar la cláusula *WHERE* para que el valor comparado con la columna *NationalIDNumber* coincida con el tipo de datos de la columna de la tabla, el optimizador puede deshacerse de la conversión implícita y generar un plan más óptimo.

### Cambio del tipo de datos

1. También podemos corregir la advertencia de conversión implícita cambiando la estructura de la tabla.

    Para intentar corregir el índice, copie y pegue la consulta siguiente en una ventana de consulta nueva para cambiar el tipo de datos de la columna. Intente ejecutar la consulta. Para ello, seleccione **Ejecutar** o presione <kbd>F5</kbd>.

    ```sql
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    ```

    Cambiar el tipo de datos de columna *NationalIDNumber* por INT resolvería el problema de conversión. Sin embargo, este cambio genera otro problema que debe resolver como administrador de bases de datos. Al ejecutar la consulta anterior se generará el siguiente mensaje de error:

    <span style="color:red">Msg 5074, Level 16, Sate 1, Line1  The index 'AK_Employee_NationalIDNumber' is dependent on column 'NationalIDNumber  Msg 4922, Level 16, State 9, Line 1  ALTER TABLE ALTER COLUMN NationalIDNumber failed because one or more objects access this column</span>

    Como la columna *NationalIDNumber* forma parte de un índice no agrupado existente, es necesario volver a crear el índice para cambiar el tipo de datos. **Esto podría provocar un tiempo de inactividad prolongado en la producción, lo que resalta la importancia de elegir los tipos de datos correctos en el diseño.**

1. Para resolver este problema, copie y pegue el código siguiente en la ventana de consulta y seleccione **Ejecutar** para ejecutarlo.

    ```sql
    USE AdventureWorks2017

    GO
    
    --Dropping the index first
    DROP INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]

    GO

    --Changing the column data type to resolve the implicit conversion warning
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;

    GO

    --Recreating the index
    CREATE UNIQUE NONCLUSTERED INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]( [NationalIDNumber] ASC );

    GO
    ```

1. Ejecuta la consulta siguiente para confirmar que el tipo de datos se cambió correctamente.

    ```sql
    SELECT c.name, t.name
    FROM sys.all_columns c INNER JOIN sys.types t
        ON (c.system_type_id = t.user_type_id)
    WHERE OBJECT_ID('[HumanResources].[Employee]') = c.object_id
        AND c.name = 'NationalIDNumber'
    ```

1. Ahora vamos a comprobar el plan de ejecución. Vuelva a ejecutar la consulta original sin las comillas.

    ```sql
    USE AdventureWorks2017
    GO

    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

     Examina el plan de consulta y ten en cuenta que ahora puedes usar un entero para filtrar por *NationalIDNumber* sin la advertencia de conversión implícita. Ahora, el optimizador de consultas de SQL puede generar y ejecutar el plan más óptimo.

>&#128221; Aunque cambiar el tipo de datos de una columna puede resolver problemas de conversión implícitos, no siempre es la mejor solución. En este caso, cambiar el tipo de datos de la columna *NationalIDNumber* a un tipo de datos **INT** habría provocado un tiempo de inactividad en producción, ya que el índice de esa columna tendría que quitarse y volver a crearse. Es importante tener en cuenta el impacto de cambiar el tipo de datos de una columna en las consultas e índices existentes antes de realizar cambios. Además, puede haber otras consultas que dependan de que la columna *NationalIDNumber* sea un tipo de datos **NVARCHAR**, por lo que cambiar el tipo de datos podría interrumpir esas consultas.

---

## Limpieza

Si no usas la base de datos o los archivos de laboratorio para ningún otro propósito, puedes limpiar los objetos que creaste en este laboratorio.

### Eliminar la carpeta C:\LabFiles

1. Desde la máquina virtual del laboratorio, o tu máquina local si no se te proporcionó ninguna, abre **Explorador de archivos**.
1. Ve a **C:\\**.
1. Elimina la carpeta **C:\LabFiles**.

## Eliminar la base de datos AdventureWorks2017

1. Desde la máquina virtual del laboratorio o la máquina local si no se proporcionó una, inicia una sesión de SQL Server Management Studio (SSMS).
1. Cuando se abra SSMS, aparecerá de forma predeterminada el cuadro de diálogo **Conectar al servidor**. Elige la instancia predeterminada y selecciona **Conectar**. Es posible que tengas que activar la casilla **Certificado de servidor de confianza**.
1. En el **Explorador de objetos**, expande la carpeta **Bases de datos**.
1. Haz clic con el botón derecho en la base de datos **AdventureWorks2017** y selecciona **Eliminar**.
1. En el cuadro de diálogo **Eliminar objeto**, activa la casilla **Cerrar conexiones existentes**.
1. Seleccione **Aceptar**.

---

Completaste correctamente este laboratorio.

En este ejercicio, has aprendido a identificar problemas de consulta causados por conversiones implícitas de tipos de datos y a corregirlos para mejorar el plan de consulta.
